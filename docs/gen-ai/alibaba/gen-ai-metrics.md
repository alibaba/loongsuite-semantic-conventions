<!--- Hugo front matter used to generate the website version of this page:
linkTitle: Alibaba Metrics
--->

# Semantic Conventions for Alibaba GenAI Metrics

**Status**: [Development][DocumentStatus]

<!-- toc -->

- [Overview](#overview)
- [Client Metrics](#client-metrics)
  - [Token Usage](#token-usage)
  - [Operation Duration](#operation-duration)
  - [Time to First Token](#time-to-first-token)
  - [Time per Output Token](#time-per-output-token)
- [Workflow and Skill Metrics](#workflow-and-skill-metrics)
- [Alibaba Metric Attributes](#alibaba-metric-attributes)

<!-- tocstop -->

## Overview

This document defines Alibaba-specific extensions to the [GenAI Metrics semantic conventions](../gen-ai-metrics.md).
These extensions add Alibaba infrastructure context to standard GenAI client and server metrics.

## Client Metrics

### Token Usage

#### Metric: `gen_ai.client.token.usage`

Number of input and output tokens used.

| Name | Instrument Type | Unit |
| --- | --- | --- |
| `gen_ai.client.token.usage` | Histogram | `{token}` |

| Attribute | Requirement Level |
| --- | --- |
| `gen_ai.token.type` | Required |
| `gen_ai.provider.name` | Required |
| `gen_ai.operation.name` | Required |
| `gen_ai.request.model` | Conditionally Required: if available |
| `gen_ai.response.model` | Recommended |
| `alibaba.base.env` | Recommended |
| `alibaba.base.idc` | Recommended |

### Operation Duration

#### Metric: `gen_ai.client.operation.duration`

GenAI operation duration from the client perspective.

| Name | Instrument Type | Unit |
| --- | --- | --- |
| `gen_ai.client.operation.duration` | Histogram | `s` |

| Attribute | Requirement Level |
| --- | --- |
| `gen_ai.provider.name` | Required |
| `gen_ai.operation.name` | Required |
| `error.type` | Conditionally Required: if error |
| `alibaba.base.env` | Recommended |
| `alibaba.base.idc` | Recommended |
| `alibaba.experiment.id` | Opt-in |
| `alibaba.group.id` | Opt-in |

### Time to First Token

#### Metric: `gen_ai.client.time_to_first_token`

Time to generate the first token from the client perspective, including network latency.

| Name | Instrument Type | Unit |
| --- | --- | --- |
| `gen_ai.client.time_to_first_token` | Histogram | `ms` |

### Time per Output Token

#### Metric: `gen_ai.client.time_per_output_token`

Average time per output token from the client perspective, including network latency.

| Name | Instrument Type | Unit |
| --- | --- | --- |
| `gen_ai.client.time_per_output_token` | Histogram | `ms` |

## Workflow and Skill Metrics

### Metric: `gen_ai.workflow.duration`

| Name | Instrument Type | Unit |
| --- | --- | --- |
| `gen_ai.workflow.duration` | Histogram | `ms` |

### Metric: `gen_ai.skill.duration`

| Name | Instrument Type | Unit |
| --- | --- | --- |
| `gen_ai.skill.duration` | Histogram | `ms` |

## Alibaba Metric Attributes

The following Alibaba-specific attributes extend standard GenAI metric attributes:

| Attribute | Type | Description | Requirement Level |
| --- | --- | --- | --- |
| `alibaba.base.env` | string | Environment identifier (DEV, PRE, PROD) | Recommended |
| `alibaba.base.idc` | string | Data center identifier | Recommended |
| `alibaba.experiment.id` | string | RL experiment identifier | Opt-in |
| `alibaba.group.id` | string | RL group identifier | Opt-in |
| `alibaba.job.id` | string | RL job identifier | Opt-in |

[DocumentStatus]: https://github.com/open-telemetry/opentelemetry-specification/blob/v1.37.0/specification/document-status.md
