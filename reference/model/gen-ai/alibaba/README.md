# Alibaba GenAI Semantic Conventions Extension

This directory contains Alibaba-specific extensions to the OpenTelemetry GenAI semantic conventions.

## Overview

These attributes are specific to Alibaba Cloud infrastructure and internal systems. They extend the standard GenAI semantic conventions with additional context relevant to Alibaba's observability platform.

## Attributes

| Attribute | Type | Description |
| --- | --- | --- |
| `alibaba.base.env` | string | Environment identifier (DEV, PRE, PROD) |
| `alibaba.base.idc` | string | Data center identifier |
| `alibaba.user.data` | string | User-defined custom data |
| `alibaba.experiment.id` | string | RL experiment identifier |
| `alibaba.group.id` | string | RL group identifier |
| `alibaba.job.id` | string | RL job identifier |
| `alibaba.instance.id` | string | RL instance identifier |

## Usage

These attributes should be used alongside the standard GenAI semantic conventions when instrumenting applications running on Alibaba Cloud infrastructure.

## Compatibility

These attributes are maintained separately from the community semantic conventions to:

1. Avoid polluting the standard namespace with vendor-specific attributes
2. Allow for independent versioning and evolution
3. Make it clear which attributes are Alibaba-specific

## Future Considerations

If any of these attributes become relevant to the broader community, they may be proposed for inclusion in the standard GenAI semantic conventions.
