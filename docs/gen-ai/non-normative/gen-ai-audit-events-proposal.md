# GenAI audit events semantic conventions proposal

This proposal defines a first GenAI audit events semantic convention for agent
and model runtime behavior. It should be read together with
[GenAI audit events research](gen-ai-audit-events-research.md), which records the
OpenTelemetry, OpenLLMetry, and OpenClaw research that informed the design.

## Problem

Existing GenAI semantic conventions cover model spans, metrics, and content
events, but they do not provide a focused audit event model for GenAI
applications and agents. Agent runtimes need to record model calls, tool
execution, agent-to-agent invocation, and final outputs in a form that can be
queried and audited without relying on vendor-specific JSONL formats.

The OpenClaw trajectory sample is used as a reference input for identifying
important runtime boundaries and known gaps. The first event set intentionally
does not attempt to cover every OpenClaw runtime entry. Instead, it standardizes
the boundaries that are stable, broadly useful, and practical for
instrumentations to collect today.

## Goals

- Define GenAI audit events as OpenTelemetry Events identified by `event.name`.
- Use event names for clear runtime boundaries and attributes for variants.
- Keep event names fully qualified under `gen_ai.*`.
- Represent core model, tool, and agent delegation boundaries in one GenAI
  audit events page.
- Support content capture through existing structured message attributes.
- Minimize first-version fields to data that implementations can reliably
  collect.

## Non-goals

- Define session lifecycle events.
- Define context compilation events.
- Define runtime model change events.
- Define human review or approval events.
- Define memory events or attributes.
- Define artifact lifecycle events.
- Define labels/custom entries as first-class events.
- Define coding-agent workspace or Git metadata under the `gen_ai` namespace.
- Define token cost attributes or payload hash attributes.
- Define skill events. Skill loading can be represented by tool execution
  attributes today; future runtimes with an explicit skill boundary can add
  `gen_ai.skill.invoke` and `gen_ai.skill.result`.
- Replace existing GenAI spans, metrics, or upstream GenAI content events.

## Proposed event set

| Event name | Purpose |
| --- | --- |
| `gen_ai.model.request` | Model request boundary. |
| `gen_ai.model.response` | Model response boundary. |
| `gen_ai.tool.call` | Actual tool execution request initiated by an agent. |
| `gen_ai.tool.result` | Result of an actual tool execution. |
| `gen_ai.agent.invoke` | Invocation or delegation of work to another GenAI agent. |
| `gen_ai.agent.result` | Result returned by another GenAI agent. |

## Naming decisions

- Use `model` instead of `llm` because the boundary is model invocation, not an
  LLM-specific implementation detail. This also works for multimodal model
  calls.
- Use `agent.invoke` instead of `agent.spawn` because invocation does not imply
  process or worker creation.
- Keep `agent.result` because it pairs naturally with `agent.invoke`.
- Do not use handoff terminology in this draft.

## Attribute model

The proposal adds common audit event correlation attributes:

- `gen_ai.session.id`
- `gen_ai.turn.id`
- `gen_ai.step.id`
- `gen_ai.agent.parent.id`
- `gen_ai.agent.parent.invocation.id`
- `event.id`
- `gen_ai.event.original_name`

It also adds event-specific attributes for agent peer invocation correlation,
tool execution correlation and duration, and token usage on model response
events.

Agent-to-agent invocation events use `gen_ai.agent.invocation.id` for the
invocation represented by the event. The callee side can use
`gen_ai.agent.parent.invocation.id` to correlate with the parent-side
invocation. This avoids using `gen_ai.agent.invocation.id` as a generic common
attribute when the current audit event does not itself represent an agent
invocation boundary.

Existing content attributes such as `gen_ai.input.messages`,
`gen_ai.input.messages_delta`, `gen_ai.system_instructions`,
`gen_ai.tool.definitions`, and `gen_ai.output.messages` are reused.

To reduce duplicated content without introducing hash fields,
`gen_ai.system_instructions` and `gen_ai.tool.definitions` SHOULD be recorded on
the first request where they apply and whenever their value changes. They SHOULD
NOT be repeated on later requests when unchanged. This lets consumers compare a
small amount of newly recorded request content while still detecting instruction
or tool-definition drift.

## OpenClaw reference mapping

| OpenClaw item | Proposed mapping |
| --- | --- |
| `session.started` | Deferred; sessions can be long-lived or resumable, so no first-version lifecycle event is defined |
| `session.ended` | Deferred; sessions can be long-lived or resumable, so no first-version lifecycle event is defined |
| `trace.metadata` | Common session/agent/user attributes where applicable; workspace and Git metadata are not standardized under `gen_ai` |
| `context.compiled` | Deferred; first version records visible request content on `gen_ai.model.request` |
| `prompt.submitted` | `gen_ai.model.request` |
| `model.completed` | `gen_ai.model.response` |
| `model.fallback_step` | Deferred; runtime model change semantics need more implementation evidence |
| `trace.artifacts` | Deferred; only durable artifacts need future event support |
| user message | `gen_ai.input.messages_delta` or `gen_ai.input.messages` |
| assistant message text | `gen_ai.output.messages` |
| assistant tool-call intent | `gen_ai.output.messages` tool-call parts |
| tool result | `gen_ai.tool.result` |
| runtime model change | Deferred; runtime model change semantics need more implementation evidence |
| human approval, denial, or stop | Deferred; review/intervention boundaries need clearer examples |
| labels/custom session entries | Deferred; use attributes or raw runtime entries until examples are available |
| top-level `id` | `event.id` |
| top-level `parentId` | Deferred; keep runtime-specific until there is an established generic event-parent attribute |

## Agent peer reference mappings

For Codex-style agent spawning, the parent-side event maps:

- `gen_ai.agent.peer.id` = `spawn_agent` output `agent_id`
- `gen_ai.agent.peer.name` = `spawn_agent` output `nickname`
- `gen_ai.agent.invocation.id` = `spawn_agent` call id

The child side maps:

- `gen_ai.agent.id` = child thread id
- `gen_ai.agent.parent.id` = parent thread id
- `gen_ai.agent.parent.invocation.id` = `spawn_agent` call id

For Claude-style Agent tool usage, the parent-side event maps:

- `gen_ai.agent.peer.id` = `claude:{sessionId}:{toolUseResult.agentId}`
- `gen_ai.agent.peer.name` = Agent tool input `description`
- `gen_ai.agent.invocation.id` = Agent `tool_use.id`

The child side maps:

- `gen_ai.agent.id` = `claude:{sessionId}:{agentId}`
- `gen_ai.agent.parent.id` = `claude:{sessionId}:root`
- `gen_ai.agent.parent.invocation.id` = Agent `tool_use.id`

## Rollout

1. Add the event groups in `model/gen-ai/audit-events.yaml`.
2. Add missing attributes in `model/gen-ai/registry.yaml`.
3. Generate `docs/gen-ai/gen-ai-audit-events.md` and registry documentation with Weaver.
4. Keep the research and proposal documents as non-normative PR context.
5. Validate with policy checks, markdown checks, and local link checks.

## Open questions

- Whether artifact events should be added later for durable run outputs with
  their own identity or URI.
- Whether labels/custom session entries appear across enough runtimes to justify
  a generic annotation event.
- Whether source runtime parent event ids should be added later through a
  generic event-parent attribute or kept in runtime-specific extensions.
- Whether context compilation, human review, and model change events should be
  added later once there are enough reliable producer examples.
