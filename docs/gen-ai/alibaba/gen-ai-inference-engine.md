<!--- Hugo front matter used to generate the website version of this page:
linkTitle: Alibaba Inference Engine
--->

# Semantic Conventions for Alibaba GenAI Inference Engine

**Status**: [Development][DocumentStatus]

<!-- toc -->

- [Overview](#overview)
- [Inference Engine Metrics](#inference-engine-metrics)
  - [Token Metrics](#token-metrics)
  - [Request Metrics](#request-metrics)
  - [Latency Metrics](#latency-metrics)
- [Alibaba-specific Attributes](#alibaba-specific-attributes)

<!-- tocstop -->

## Overview

This document defines Alibaba-specific extensions to the [GenAI Inference Engine semantic conventions](../gen-ai-metrics.md).
These metrics are designed to monitor GenAI inference engines such as vLLM and SGLang deployed on Alibaba infrastructure.
They provide visibility into token processing, request queueing, and end-to-end latency with additional Alibaba context.

## Inference Engine Metrics

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

### Alibaba Inference Engine Metric Attributes

In addition to the standard [inference engine metric attributes](../gen-ai-metrics.md), the following
Alibaba-specific attributes SHOULD be populated for inference engine metrics on Alibaba infrastructure:

| Attribute | Requirement Level | Description |
| --- | --- | --- |
| `alibaba.base.env` | Recommended | Environment identifier (DEV, PRE, PROD) |
| `alibaba.base.idc` | Recommended | Data center identifier where the inference engine is deployed |

## Alibaba-specific Attributes

See [Alibaba GenAI Extension Attributes](../../model/gen-ai/alibaba/) for full attribute definitions.

## Compatibility

These metrics are designed to be compatible with popular open-source inference engines deployed on Alibaba infrastructure:

- **vLLM**: Metrics align with vLLM's Prometheus exporter
- **SGLang**: Compatible with SGLang's monitoring interface

[DocumentStatus]: https://github.com/open-telemetry/opentelemetry-specification/blob/v1.37.0/specification/document-status.md
