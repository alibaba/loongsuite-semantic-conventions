---
name: otel-semconv-add-conventions
description: Guide for adding new semantic conventions to OpenTelemetry semantic-conventions repositories using the YAML model + Weaver toolchain. Use when adding new attributes, spans, metrics, events, or resources to a semconv project, creating new area conventions, or editing model/*.yaml and docs/*.md files in semantic-conventions repos.
---

# Adding Semantic Conventions

Guide for adding or modifying semantic conventions in any repo following the
`open-telemetry/semantic-conventions` structure (YAML model + Weaver generation).

## Architecture Overview

```
model/*.yaml  ─── (Single Source of Truth)
     │
     ├─ Weaver resolve ──► Resolved Registry
     │                          │
     │                     ├─ Rego policies ──► Validation
     │                     ├─ Jinja2 templates ──► docs/ (Markdown)
     │                     └─ Jinja2 templates ──► SDK code (Go/Java/Python...)
     │
docs/*.md  ─── (Mix of hand-written + auto-generated sections)
```

**Key rule**: YAML under `model/` is the only source of truth. Never hand-edit
content between `<!-- semconv -->` and `<!-- endsemconv -->` markers in docs.

## File Layout

```
model/{namespace}/
  ├── registry.yaml            # Attribute definitions (type: attribute_group, id: registry.{ns})
  ├── spans.yaml               # Span conventions (type: span)
  ├── metrics.yaml             # Metric conventions (type: metric)
  ├── events.yaml              # Event conventions (type: event)
  ├── common.yaml              # Shared attribute groups (optional)
  └── deprecated/
      └── registry-deprecated.yaml

docs/{namespace}/
  ├── README.md                # Hand-written overview + auto-generated tables
  ├── {topic}-spans.md         # Hand-written prose + auto-generated tables
  └── {topic}-metrics.md
```

## Step-by-Step: Add a New Convention

### Step 1: Define attributes in the registry

New attributes MUST be defined in `model/{namespace}/registry.yaml` inside a
group with `type: attribute_group` and `id` starting with `registry.`:

```yaml
groups:
  - id: registry.myns
    type: attribute_group
    display_name: MyNamespace Attributes
    brief: 'Attributes for MyNamespace.'
    attributes:
      - id: myns.operation.name
        type: string
        stability: development
        brief: "Name of the operation."
        examples: ['create', 'read', 'update']
      - id: myns.status_code
        type:
          members:
            - id: ok
              value: 'ok'
              brief: 'Success'
              stability: development
            - id: error
              value: 'error'
              brief: 'Error'
              stability: development
        stability: development
        brief: "Status code of the operation."
```

**Rules for attributes:**

- New attributes use `stability: development`
- Provide realistic `examples`
- Enum types define `members` with `id`, `value`, `brief`, `stability`
- Reuse existing attributes (`server.address`, `error.type`, etc.) when possible
- Follow [naming guidance](../../../docs/general/naming.md): `{namespace}.{noun}.{adjective}`

### Step 2: Define signal conventions (spans/metrics/events)

Create `model/{namespace}/spans.yaml` (or metrics/events) that **reference**
registry attributes via `ref:`:

```yaml
groups:
  - id: span.myns.client
    type: span
    stability: development
    span_kind: client
    brief: "Client span for MyNamespace operations."
    note: |
      **Span name:** `{operation.name}`

      **Span kind** MUST be `CLIENT`.

      **Span status** Refer to the [Recording Errors](/docs/general/recording-errors.md)
      document for details.
    attributes:
      - ref: myns.operation.name
        requirement_level: required
        sampling_relevant: true
      - ref: myns.status_code
        requirement_level:
          conditionally_required: when available
      - ref: server.address
        requirement_level:
          conditionally_required: If available.
        sampling_relevant: true
      - ref: error.type
        requirement_level:
          conditionally_required: If and only if the operation failed.
```

**Inheritance with `extends`**: reuse common attribute groups:

```yaml
  - id: span.myns.server
    type: span
    extends: span.myns.client     # inherits all attributes
    span_kind: server
    brief: "Server span for MyNamespace operations."
    attributes:
      - ref: client.address       # add server-specific attributes
        requirement_level: recommended
```

For **metrics**:

```yaml
  - id: metric.myns.request.duration
    type: metric
    metric_name: myns.request.duration
    instrument: histogram
    unit: "s"
    stability: development
    brief: "Duration of MyNamespace requests."
    attributes:
      - ref: myns.operation.name
        requirement_level: required
```

### Step 3: Create/update documentation markdown

For a **new namespace**, create `docs/{namespace}/{topic}.md`:

```markdown
<!--- Hugo front matter used to generate the website version of this page:
linkTitle: MyNamespace
--->

# Semantic conventions for MyNamespace

**Status**: [Development][DocumentStatus]

Brief description of what these conventions cover.

## Client Span

<!-- semconv span.myns.client -->
<!-- endsemconv -->

## Server Span

<!-- semconv span.myns.server -->
<!-- endsemconv -->

[DocumentStatus]: https://opentelemetry.io/docs/specs/otel/document-status
```

**What is hand-written vs auto-generated:**

| Part | Source | Editable? |
| --- | --- | --- |
| Hugo frontmatter (`<!--- Hugo front matter ... --->`) | Hand-written | Yes |
| Page title, status badge, intro text | Hand-written | Yes |
| Section headers (`## Client Span`) | Hand-written | Yes |
| `<!-- semconv {group_id} -->` marker | Hand-written (one-time) | Only the group_id |
| Content between semconv markers | **Auto-generated by Weaver** | **NO** |
| `<!-- endsemconv -->` end marker | Hand-written (one-time) | No |
| Link references at bottom | Hand-written | Yes |

### Step 4: Run generation

```bash
make table-generation registry-generation
```

This runs Weaver to:

1. Resolve all YAML model files (expand `ref:`, `extends:`)
2. Replace content between `<!-- semconv -->` / `<!-- endsemconv -->` markers
3. Generate `docs/registry/attributes/{namespace}.md` pages

### Step 5: Validate

```bash
make check-policies   # Rego policy checks (naming, compatibility, stability)
make check            # Full validation (links, spelling, formatting, policies)
```

### Step 6: Changelog (if applicable)

```bash
make chlog-new        # Creates .chloggen/{branch-name}.yaml
# Fill in the generated file, then:
make chlog-validate
```

## YAML Syntax Quick Reference

### Group types

| type | Use for | Required fields |
| --- | --- | --- |
| `attribute_group` | Attribute registry or shared groups | `id`, `attributes` |
| `span` | Span conventions | `id`, `span_kind`, `brief`, `stability` |
| `metric` | Metric conventions | `id`, `metric_name`, `instrument`, `unit`, `stability` |
| `event` | Event conventions | `id`, `name`, `stability` |
| `resource` | Resource conventions | `id`, `stability` |

### Attribute definition (in registry)

```yaml
- id: namespace.attribute_name       # dot-separated, snake_case
  type: string | int | double | boolean | string[] | int[] | template[string[]]
  stability: development | stable
  deprecated: "Use X instead."       # optional
  brief: "Short description."
  note: "Longer explanation."        # optional, supports markdown
  examples: ['value1', 'value2']
```

### Attribute reference (in signal groups)

```yaml
- ref: namespace.attribute_name      # must exist in a registry.* group
  requirement_level: required | recommended | opt_in
    # or conditional:
    # conditionally_required: "When X is available."
  brief: "Override brief."           # optional override
  note: "Override note."             # optional override
  sampling_relevant: true            # optional, for spans
```

### Requirement levels

| Level | Meaning |
| --- | --- |
| `required` | MUST be provided |
| `conditionally_required` | MUST if condition met (provide condition text) |
| `recommended` | SHOULD be provided |
| `opt_in` | MAY be provided, not by default |

## Common Patterns

### Extending a base group (inheritance)

Used when multiple protocols share common attributes (e.g., RPC):

```yaml
# Base group (attribute_group, not a signal)
- id: common.myns.attributes
  type: attribute_group
  attributes:
    - ref: server.address
    - ref: error.type

# Protocol-specific span extends base
- id: span.myns.grpc.client
  type: span
  extends: common.myns.attributes
  span_kind: client
  attributes:
    - ref: myns.grpc.status_code   # add protocol-specific attrs
```

### Adding system-specific enum member

When adding a new system (e.g., new database, new RPC framework):

1. Add enum member in existing registry:

   ```yaml
   # model/rpc/registry.yaml, under rpc.system.name members:
   - id: my_protocol
     value: 'my_protocol'
     brief: '[MyProtocol](https://example.com)'
     stability: development
   ```

2. Add span/metric definitions in `model/rpc/spans.yaml`
3. Create docs page `docs/rpc/my-protocol.md` with semconv markers

## External References

- [Semconv YAML syntax](https://github.com/open-telemetry/weaver/blob/main/schemas/semconv-syntax.md)
- [Weaver template docs](https://github.com/open-telemetry/weaver/blob/main/crates/weaver_forge/README.md)
- [How to write conventions](../../../docs/how-to-write-conventions/README.md)
- [Naming guidelines](../../../docs/general/naming.md)
- [Attribute requirement levels](../../../docs/general/attribute-requirement-level.md)
