<!--- Hugo front matter used to generate the website version of this page:
linkTitle: Inference
--->

# Semantic Conventions for GenAI Inference Engine

**Status**: [Development][DocumentStatus]

<!-- toc -->

- [Inference Engine Metrics](#inference-engine-metrics)
  - [Token Metrics](#token-metrics)
  - [Request Metrics](#request-metrics)
  - [Latency Metrics](#latency-metrics)

<!-- tocstop -->

## Inference Engine Metrics

These metrics are designed to monitor GenAI inference engines such as vLLM and SGLang.
They provide visibility into token processing, request queueing, and end-to-end latency.

### Token Metrics

| Name | Instrument Type | Unit | Description |
| --- | --- | --- | --- |
| `gen_ai.server.prompt_tokens` | Counter | `{token}` | Number of prompt/input tokens processed by the inference engine. |
| `gen_ai.server.generation_tokens` | Counter | `{token}` | Number of generation/output tokens produced by the inference engine. |
| `gen_ai.server.cached_tokens` | Histogram | `{token}` | Number of tokens served from cache by the inference engine. |

### Request Metrics

| Name | Instrument Type | Unit | Description |
| --- | --- | --- | --- |
| `gen_ai.server.num_requests_running` | Gauge | `{request}` | Number of requests currently being processed by the inference engine. |
| `gen_ai.server.num_requests_waiting` | Gauge | `{request}` | Number of requests waiting to be processed by the inference engine. |

### Latency Metrics

| Name | Instrument Type | Unit | Description |
| --- | --- | --- | --- |
| `gen_ai.server.e2e_request_latency` | Histogram | `ms` | End-to-end request latency including queue time and processing time. |

### Metric Attributes

The following attributes are used with inference engine metrics:

| Attribute | Requirement Level | Description |
| --- | --- | --- |
| `gen_ai.provider.name` | Required | The Generative AI provider name. |
| `gen_ai.operation.name` | Required | The name of the operation being performed. |
| `gen_ai.request.model` | Conditionally Required: if available | The name of the model a request is being made to. |
| `gen_ai.response.model` | Recommended | The name of the model that generated the response. |
| `server.address` | Recommended | GenAI server address. |
| `server.port` | Conditionally Required: if server.address is set | GenAI server port. |
| `error.type` | Conditionally Required: if error | Error type if the operation ended in an error. |

## Compatibility

These metrics are designed to be compatible with popular open-source inference engines:

- **vLLM**: Metrics align with vLLM's Prometheus exporter
- **SGLang**: Compatible with SGLang's monitoring interface

[DocumentStatus]: https://github.com/open-telemetry/opentelemetry-specification/blob/v1.37.0/specification/document-status.md
