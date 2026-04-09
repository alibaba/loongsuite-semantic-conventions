<!--- Hugo front matter used to generate the website version of this page:
linkTitle: GenAI Metrics
--->

# Semantic Conventions for GenAI Metrics

**Status**: [Development][DocumentStatus]

<!-- toc -->

- [Metric Attributes](#metric-attributes)
- [Client Metrics](#client-metrics)
  - [Metric: `gen_ai.client.token.usage`](#metric-gen_aiclienttokenusage)
  - [Metric: `gen_ai.client.operation.duration`](#metric-gen_aiclientoperationduration)
  - [Metric: `gen_ai.client.time_to_first_token`](#metric-gen_aiclienttime_to_first_token)
  - [Metric: `gen_ai.client.time_per_output_token`](#metric-gen_aiclienttime_per_output_token)
- [Server Metrics](#server-metrics)
  - [Metric: `gen_ai.server.request.duration`](#metric-gen_aiserverrequestduration)
  - [Metric: `gen_ai.server.time_to_first_token`](#metric-gen_aiservertime_to_first_token)
  - [Metric: `gen_ai.server.time_per_output_token`](#metric-gen_aiservertime_per_output_token)
- [Workflow and Skill Metrics](#workflow-and-skill-metrics)
  - [Metric: `gen_ai.workflow.duration`](#metric-gen_aiworkflowduration)
  - [Metric: `gen_ai.skill.duration`](#metric-gen_aiskillduration)

<!-- tocstop -->

## Metric Attributes

The following attributes are common to GenAI metrics:

| Attribute | Requirement Level | Description |
| --- | --- | --- |
| `gen_ai.provider.name` | Required | The Generative AI provider name. |
| `gen_ai.operation.name` | Required | The name of the operation being performed. |
| `gen_ai.request.model` | Conditionally Required: if available | The name of the model a request is being made to. |
| `gen_ai.response.model` | Recommended | The name of the model that generated the response. |
| `server.address` | Recommended | GenAI server address. |
| `server.port` | Conditionally Required: if `server.address` is set | GenAI server port. |
| `error.type` | Conditionally Required: if error | Error type if the operation ended in an error. |

## Client Metrics

### Metric: `gen_ai.client.token.usage`

Number of input and output tokens used.

| Name | Instrument Type | Unit |
| --- | --- | --- |
| `gen_ai.client.token.usage` | Histogram | `{token}` |

| Attribute | Requirement Level |
| --- | --- |
| `gen_ai.provider.name` | Required |
| `gen_ai.operation.name` | Required |
| `gen_ai.token.type` | Required |
| `gen_ai.request.model` | Conditionally Required: if available |
| `gen_ai.response.model` | Recommended |
| `server.address` | Recommended |
| `server.port` | Conditionally Required: if `server.address` is set |

### Metric: `gen_ai.client.operation.duration`

GenAI operation duration from the client perspective.

| Name | Instrument Type | Unit |
| --- | --- | --- |
| `gen_ai.client.operation.duration` | Histogram | `s` |

| Attribute | Requirement Level |
| --- | --- |
| `gen_ai.provider.name` | Required |
| `gen_ai.operation.name` | Required |
| `gen_ai.request.model` | Conditionally Required: if available |
| `gen_ai.response.model` | Recommended |
| `server.address` | Recommended |
| `server.port` | Conditionally Required: if `server.address` is set |
| `error.type` | Conditionally Required: if error |

### Metric: `gen_ai.client.time_to_first_token`

Time to generate the first token from the client perspective, including network latency.

| Name | Instrument Type | Unit |
| --- | --- | --- |
| `gen_ai.client.time_to_first_token` | Histogram | `ms` |

| Attribute | Requirement Level |
| --- | --- |
| `gen_ai.provider.name` | Required |
| `gen_ai.operation.name` | Required |
| `gen_ai.request.model` | Conditionally Required: if available |
| `gen_ai.response.model` | Recommended |
| `server.address` | Recommended |
| `server.port` | Conditionally Required: if `server.address` is set |
| `error.type` | Conditionally Required: if error |

### Metric: `gen_ai.client.time_per_output_token`

Average time per output token from the client perspective, including network latency.

| Name | Instrument Type | Unit |
| --- | --- | --- |
| `gen_ai.client.time_per_output_token` | Histogram | `ms` |

| Attribute | Requirement Level |
| --- | --- |
| `gen_ai.provider.name` | Required |
| `gen_ai.operation.name` | Required |
| `gen_ai.request.model` | Conditionally Required: if available |
| `gen_ai.response.model` | Recommended |
| `server.address` | Recommended |
| `server.port` | Conditionally Required: if `server.address` is set |
| `error.type` | Conditionally Required: if error |

## Server Metrics

### Metric: `gen_ai.server.request.duration`

Generative AI server request duration such as time-to-last byte or last output token.

| Name | Instrument Type | Unit |
| --- | --- | --- |
| `gen_ai.server.request.duration` | Histogram | `s` |

### Metric: `gen_ai.server.time_to_first_token`

Time to generate first token for successful responses.

| Name | Instrument Type | Unit |
| --- | --- | --- |
| `gen_ai.server.time_to_first_token` | Histogram | `s` |

### Metric: `gen_ai.server.time_per_output_token`

Time per output token generated after the first token for successful responses.

| Name | Instrument Type | Unit |
| --- | --- | --- |
| `gen_ai.server.time_per_output_token` | Histogram | `s` |

## Workflow and Skill Metrics

### Metric: `gen_ai.workflow.duration`

GenAI workflow invocation duration.

| Name | Instrument Type | Unit |
| --- | --- | --- |
| `gen_ai.workflow.duration` | Histogram | `ms` |

| Attribute | Requirement Level |
| --- | --- |
| `gen_ai.provider.name` | Required |
| `gen_ai.operation.name` | Required |
| `gen_ai.workflow.name` | Required |
| `gen_ai.request.model` | Conditionally Required: if available |
| `gen_ai.response.model` | Recommended |
| `server.address` | Recommended |
| `server.port` | Conditionally Required: if `server.address` is set |
| `error.type` | Conditionally Required: if error |

### Metric: `gen_ai.skill.duration`

GenAI skill execution duration.

| Name | Instrument Type | Unit |
| --- | --- | --- |
| `gen_ai.skill.duration` | Histogram | `ms` |

| Attribute | Requirement Level |
| --- | --- |
| `gen_ai.provider.name` | Required |
| `gen_ai.operation.name` | Required |
| `gen_ai.skill.name` | Required |
| `gen_ai.request.model` | Conditionally Required: if available |
| `gen_ai.response.model` | Recommended |
| `server.address` | Recommended |
| `server.port` | Conditionally Required: if `server.address` is set |
| `error.type` | Conditionally Required: if error |

[DocumentStatus]: https://github.com/open-telemetry/opentelemetry-specification/blob/v1.37.0/specification/document-status.md
