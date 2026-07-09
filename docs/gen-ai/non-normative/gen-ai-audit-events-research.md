# GenAI audit events research

This note summarizes the research used to decide how GenAI audit events should be
modeled before changing the normative YAML. It focuses on current
OpenTelemetry event/log semantics, existing GenAI examples, OpenLLMetry's agent
observability RFC, and the OpenClaw trajectory sample in
[`../openclaw-trajectory.md`](../openclaw-trajectory.md).

## Sources reviewed

- OpenTelemetry Logs Data Model:
  <https://opentelemetry.io/docs/specs/otel/logs/data-model/>
- OpenTelemetry semantic conventions for events:
  <https://opentelemetry.io/docs/specs/semconv/general/events/>
- OpenTelemetry semantic conventions for session:
  <https://opentelemetry.io/docs/specs/semconv/general/session/>
- OpenTelemetry semantic conventions for GenAI events:
  <https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-events/>
- Local upstream-style GenAI event model:
  [`../../../model/gen-ai/events.yaml`](../../../model/gen-ai/events.yaml)
- Deprecated GenAI message events:
  [`../../../model/gen-ai/deprecated/events-deprecated.yaml`](../../../model/gen-ai/deprecated/events-deprecated.yaml)
- OpenLLMetry RFC for AI agent observability:
  <https://github.com/traceloop/openllmetry/issues/3460>

## OTel event and log findings

OpenTelemetry models standalone Events as a specialized `LogRecord`, not as a
separate signal. The Logs Data Model says a log record with a non-empty event
name is an Event, and the event name should identify the event structure. The
semantic conventions for events add the stricter rule that semantic convention
event definitions must document the event name and attributes.

The practical implication is that `event.name` is not just a category label. It
should identify the kind of occurrence and the expected attribute/body shape.
Attributes then provide details and context about that occurrence. OTel also
discourages using log body for structured event semantics; body is best kept for
a display message.

OTel's current examples support this split:

- General session conventions define concrete lifecycle events:
  `session.start` and `session.end`.
- Current GenAI events include concrete names such as
  `gen_ai.client.inference.operation.details` and
  `gen_ai.evaluation.result`.
- This repository's GenAI event model also includes point-in-time token and
  exception events such as `gen_ai.token.scheduled`,
  `gen_ai.token.generated`, and `gen_ai.client.operation.exception`.
- Deprecated GenAI message events such as `gen_ai.user.message`,
  `gen_ai.assistant.message`, and `gen_ai.tool.message` were replaced by
  structured `gen_ai.input.messages` / `gen_ai.output.messages` capture on
  spans or events. This is strong evidence against creating a separate event for
  every chat role when the content can be represented as structured message
  attributes.

## Naming conclusion

The recommended style is a hybrid:

- Use fully qualified names under `gen_ai.*`; avoid unqualified names such as
  `model.request`.
- Split `event.name` when the event has a different lifecycle point, required
  attributes, severity expectations, or consumer handling. Examples:
  `gen_ai.model.request`, `gen_ai.model.response`, `gen_ai.tool.call`,
  `gen_ai.tool.result`, `session.start`, and `session.end`.
- Use an operation attribute when variants share the same event structure and
  differ only by operation value. `gen_ai.human.review` plus
  `gen_ai.human.operation = approve | deny | stop` is better than three
  separate event names unless those variants later require distinct schemas.
- Prefer `gen_ai.<component>.<occurrence>` over deeply nested names when the
  third segment already identifies the occurrence. For example,
  `gen_ai.human.review` is clearer than
  `gen_ai.human.review.operation` because `operation` is an attribute-like word,
  not a concrete event occurrence.

In other words, `gen_ai.model.request` and `gen_ai.model.response` should remain
separate events because request and response have materially different schemas.
`gen_ai.human.review` should probably remain one event with an operation
attribute because approve/deny/stop are variants of the same review occurrence.

## Decisions from review

These decisions were made after reviewing the initial research:

1. Do not reuse generic OTel `session.start` and `session.end` for GenAI
   application audit events. The generic session attributes are too thin for GenAI
   application sessions, autonomous runs, and agent sessions.
2. Model context compilation as a standalone event. It represents the context
   assembled before model submission, which is important for audit and prompt
   injection analysis. It should not be hidden inside `gen_ai.model.request`.
3. Use one model-change event with a reason attribute. A single
   `gen_ai.model.change` event with reason values such as `user`, `policy`, and
   `fallback` should cover explicit user changes, policy-directed changes, and
   fallback-driven changes.
4. Use `model` rather than `llm` in log event names. The intended boundary is a
   model invocation, not an LLM-specific implementation detail, and `model`
   aligns with sibling event names such as `gen_ai.tool.call` and
   `gen_ai.agent.invoke`.
5. Rename `gen_ai.agent.spawn` to `gen_ai.agent.invoke`, while keeping
   `gen_ai.agent.result`. `invoke/result` names the call boundary and returned
   result without implying that the runtime created a new process or worker.
   Use `gen_ai.agent.peer.*` for the other agent in the immediate interaction
   so current-agent identity and target-agent identity can coexist on the same
  audit event. Keep `gen_ai.agent.invocation.id` scoped to the invocation
   boundary itself, and use `gen_ai.agent.parent.invocation.id` on the callee
   side when correlating back to the parent invocation.
6. Do not use handoff terminology in this draft. It is not accepted as a core
   event name for these conventions.
7. Do not include `gen_ai.skill.use` in the first core event set. Reading a
   skill file has no clear telemetry boundary, and whether the skill is injected
   into context as instructions is runtime-dependent. If a future runtime has a
   clear skill boundary, use `gen_ai.skill.invoke` and `gen_ai.skill.result`.

## OpenLLMetry RFC relevance

The OpenLLMetry issue is relevant because it proposes a broader agent
observability taxonomy across frameworks such as LangGraph, CrewAI, Autogen,
Google ADK, LlamaIndex, OpenAI Agents SDK, and AWS Bedrock AgentCore. Its span
taxonomy uses a `gen_ai.<component>.<operation>` pattern and explicitly calls
out agent lifecycle, task execution, tool execution, workflow orchestration,
context/state, guardrails, evaluations, human-in-the-loop, session management,
artifacts, and memory systems.

Takeaways for this repository:

- It supports treating agent behavior as first-class GenAI semantics instead of
  modeling everything as LLM calls.
- It supports the component/operation naming shape already used in this draft.
- It is an RFC in an external issue, not an accepted OTel semantic convention,
  so it should inform design but not be copied wholesale.
- Its memory taxonomy is intentionally not adopted for now. The current repo has
  no broader memory conventions, and memory support was explicitly removed from
  this draft's immediate scope.
- Its artifact attributes describe output artifacts by id, MIME type, size, URI,
  and description. The RFC treats artifacts as outputs of the agent/task, not
  necessarily as immediate model or tool return values.

## Candidate event additions from OpenLLMetry

OpenLLMetry is most useful as a taxonomy check. The following items are worth
borrowing or explicitly rejecting for this GenAI audit events draft:

| OpenLLMetry concept | Recommendation | Reason |
| --- | --- | --- |
| Session lifecycle | Add GenAI-specific `gen_ai.session.start` and `gen_ai.session.end` | The generic OTel session events are too thin, and OpenLLMetry's session attributes better match agent sessions, autonomous runs, and resumable conversations. |
| Context/state | Add `gen_ai.context.compile` | OpenClaw exposes `context.compiled`, and audit use cases need to see the assembled context before model submission. |
| Model/provider change | Add `gen_ai.model.change` with reason values such as `user`, `policy`, `fallback` | This covers OpenClaw `model.fallback_step` and ordinary runtime model switches without defining a separate fallback event. |
| Agent invocation | Replace `gen_ai.agent.spawn` with `gen_ai.agent.invoke`; keep `gen_ai.agent.result`; use `gen_ai.agent.peer.*` for the other agent in the interaction | Invocation names the boundary without implying process creation. Peer attributes cover local agents, remote agents, delegation, agent-as-tool, and message-send relationships without forcing hierarchy. Invocation ids stay on the invocation boundary; callee logs can correlate with `gen_ai.agent.parent.invocation.id`. |
| Task lifecycle | Defer | Useful for CrewAI-style task systems, but overlaps with our current turn/step model and is not required to cover OpenClaw. |
| Workflow transition/branch | Defer | Useful for LangGraph/Mastra graph execution, but not needed for the current OpenClaw trajectory sample. Could be a later workflow-specific extension. |
| Guardrail check | Defer, but keep in mind for audit/security | Valuable for prompt-injection and safety pipelines, but no current OpenClaw example forces it. |
| Evaluation result | Reuse existing `gen_ai.evaluation.result` | OTel GenAI already defines this event; this audit events draft should reference it instead of creating another eval event. |
| MCP connect/execute | Do not add separate core events yet | MCP can be represented as tool execution with `gen_ai.tool.type` or operation details unless connection lifecycle becomes important. |
| Skill execution | Do not add `gen_ai.skill.use` now; reserve `gen_ai.skill.invoke` / `gen_ai.skill.result` for a future explicit boundary | In current coding-agent usage, a skill may just be a file read and optional context injection, so it is not yet a stable event boundary. |
| Memory events | Do not add | Memory is out of scope for this repository iteration. |
| Artifact event | Defer | Useful only for durable run/task outputs with their own identity or URI; ordinary model/tool outputs are already covered. |

## OpenClaw coverage matrix

The OpenClaw sample is a good completeness test because it has both a transcript
log and a runtime trajectory log.

| OpenClaw item | Candidate semantic mapping | Coverage status |
| --- | --- | --- |
| `session.started` | GenAI-specific session lifecycle event with richer session attributes | Gap |
| `session.ended` | GenAI-specific session lifecycle event with richer session attributes | Gap |
| `trace.metadata` | Common session, agent, user, workspace, and runtime attributes | Partially covered; arbitrary metadata needs a policy decision |
| `context.compiled` | Standalone context compile event with `gen_ai.input.messages`, `gen_ai.system_instructions`, `gen_ai.tool.definitions`, context window attributes | Gap; decided as standalone event |
| `prompt.submitted` | `gen_ai.model.request` | Covered after rename |
| `model.completed` | `gen_ai.model.response` | Covered after rename |
| `model.fallback_step` | `gen_ai.model.change` with reason `fallback`; optional fallback detail attributes if needed later | Gap |
| `trace.artifacts` | Artifact event or artifact attributes linked to session/turn/step | Open question |
| user message | `gen_ai.input.messages_delta` or `gen_ai.input.messages` on request/context event | Covered if content capture is enabled |
| assistant message text | `gen_ai.output.messages` on response event | Covered if content capture is enabled |
| assistant tool-call intent | `gen_ai.output.messages` with tool-call parts; actual execution uses `gen_ai.tool.call` | Covered |
| tool result | `gen_ai.tool.result` plus tool-call identifiers and result body | Covered |
| runtime model change | `gen_ai.model.change` with previous/new model and reason | Gap |
| labels/custom session entries | Annotation/custom event or an `other` escape hatch | Open question |
| top-level `id` / `parentId` | Source event id attribute; defer parent event id until a generic event-parent convention exists | Gap |

The current draft is therefore enough for basic LLM/tool/HITL agent audit
events, but it does not fully cover the OpenClaw trajectory. The missing pieces
are mostly runtime lifecycle and audit artifacts rather than ordinary model
request/response data.

## Artifact notes

An artifact is a durable or separately addressable output of the agent run, such
as a generated report, code patch, file, image, exported prompt bundle, or
summary JSON. OpenLLMetry's RFC examples include markdown, JSON, and PNG
artifacts. OpenClaw trajectory bundles also have an `artifacts.json` file that
summarizes final status, errors, usage, prompt cache, assistant text, and tool
metadata.

Artifacts overlap with `gen_ai.model.response` and `gen_ai.tool.result`, but they
are not identical:

- `gen_ai.model.response` records what the model returned at a specific model
  call boundary.
- `gen_ai.tool.result` records what a tool returned at a specific tool execution
  boundary.
- An artifact records a run-level or task-level output that may be assembled
  from many model/tool steps, stored outside telemetry, too large to inline, or
  useful to retrieve independently.

Because of that, artifact should probably not be a required core event in the
first audit events spec. A separate artifact event is useful only when the runtime
creates or exports a durable object with its own identity, URI, size, MIME type,
or lifecycle. If the "artifact" is just the immediate LLM text or tool result,
the existing response/result event is enough.

## Labels and custom entries notes

OpenClaw mentions labels and custom session entries as transcript events
reconstructed into the trajectory, but the current sample does not include
concrete examples. Plausible examples are user- or system-attached labels such
as task type, risk level, source channel, benchmark id, evaluation category, or
custom application markers inserted by plugins.

Without concrete examples, labels/custom entries should not drive first-class
event definitions. The conservative options are:

- represent known labels as normal attributes on the nearest session, turn, or
  event;
- add a generic annotation event only if multiple runtimes expose comparable
  label/custom-entry concepts;
- otherwise keep an `other`/raw-event escape hatch for runtime-specific entries.

## Design implications before YAML changes

Before expanding the YAML, decide these remaining points explicitly:

1. Whether artifacts are represented inline, by URI only, or both. This has
   privacy and payload-size implications.
2. Whether labels/custom entries deserve a generic annotation event. This avoids
   creating one event per local runtime extension.
3. Whether source runtime event ids and parent ids are common enough to add as
   common attributes, or should stay in runtime-specific extensions.

The next YAML iteration should be driven by those decisions, not by copying all
fields from OpenClaw or OpenLLMetry mechanically.

## Naming normalization recommendations

The current draft should be normalized in a few places before generating the
final docs:

| Current name | Recommendation | Reason |
| --- | --- | --- |
| `gen_ai.llm.request` / `gen_ai.llm.response` | Rename to `gen_ai.model.request` / `gen_ai.model.response` | `model` keeps the boundary at model invocation while avoiding LLM-specific wording. It also matches sibling boundary-action names such as `gen_ai.tool.call` and `gen_ai.agent.invoke`. |
| `gen_ai.agent.spawn` | Rename to `gen_ai.agent.invoke` | `spawn` is implementation-specific. `invoke` names the agent-call boundary and works for local agents, remote agents, and in-process agents. |
| `gen_ai.agent.result` | Keep | It names the returned result boundary and pairs naturally with `gen_ai.agent.invoke`. |
| `gen_ai.agent.handoff` | Do not use | Handoff terminology is not accepted for this draft. |
| `gen_ai.skill.use` | Remove from the first core event set | A skill may only be a file read or context injection, so the event boundary is unclear. If a future runtime has a clear skill execution boundary, use `gen_ai.skill.invoke` and `gen_ai.skill.result`. |
| `gen_ai.human.review` | Keep | It matches OpenLLMetry and is a good event-name boundary; variants should stay in attributes. |
| `gen_ai.tool.call` / `gen_ai.tool.result` | Keep | Splitting actual execution request and result is clear for logs, and it distinguishes tool execution from model-emitted tool-call intent. |

The most important OTel naming rule is to keep event names as concrete
occurrences with stable schemas. Avoid adding event names whose last segment is
an attribute-like noun such as `operation`, `type`, or `status` unless it
actually names a distinct occurrence.
