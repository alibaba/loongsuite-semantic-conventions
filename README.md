# LoongSuite Semantic Conventions

LoongSuite Semantic Conventions is an **open-source semantic conventions repository from Alibaba** that provides enhanced GenAI (Generative AI) observability standards, building upon the foundation of [OpenTelemetry Semantic Conventions](https://github.com/open-telemetry/semantic-conventions).

## About

This repository supplements the OpenTelemetry Semantic Conventions with additional GenAI-specific semantic conventions derived from Alibaba's internal observability practices. It defines a comprehensive set of semantic attributes, spans, metrics, and events specifically designed for tracing and monitoring Generative AI applications and services.

### Key Features

- **GenAI-Focused**: Specialized semantic conventions for LLM applications, model interactions, and AI service observability
- **Production-Tested**: Based on real-world experience from Alibaba's internal AI infrastructure
- **OTel-Compatible**: Built on OpenTelemetry standards, ensuring seamless integration with the OTel ecosystem
- **Community-Driven**: Open-sourced to benefit the broader GenAI observability community

### Use Cases

- Tracing LLM API calls (OpenAI, Alibaba Cloud Bailian, Spring AI, etc.)
- Monitoring token usage and model performance
- Tracking prompt engineering and response quality
- Observing multi-modal AI interactions
- Measuring AI service latency and costs

## Documentation

The human-readable semantic conventions documentation is available in the [docs](docs/README.md) folder. These Markdown documents are generated from YAML model definitions located in the [model](model/README.md) folder, following the OpenTelemetry Semantic Conventions toolchain approach.

## Repository Structure

```
├── model/          # YAML definitions for semantic conventions
├── docs/           # Generated documentation (Markdown)
├── templates/      # Weaver templates for code generation
└── schemas/        # JSON schemas for validation
```

## Relationship with OpenTelemetry

LoongSuite Semantic Conventions extends and supplements the official [OpenTelemetry Semantic Conventions](https://github.com/open-telemetry/semantic-conventions). While OTel provides foundational semantic conventions for distributed tracing, metrics, and logs, LoongSuite focuses specifically on GenAI observability patterns that emerged from Alibaba's production AI workloads.

### What LoongSuite Adds

- **Extended GenAI Attributes**: Additional attributes for AI model characteristics, prompt engineering, and response quality
- **Alibaba Cloud Integration**: Specific conventions for Alibaba Cloud Bailian and other cloud AI services
- **Multi-Modal Support**: Conventions for image, video, and audio AI interactions
- **Cost Tracking**: Detailed attributes for tracking AI service costs and resource consumption
- **Quality Metrics**: Conventions for measuring AI output quality, relevance, and safety

## Contributing

We welcome contributions from the community! Whether you're adding new GenAI conventions, improving documentation, or fixing issues, your input is valuable.

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines on:

- How to propose new semantic conventions
- Code of conduct
- Development workflow
- Testing and validation requirements

## License

This project is licensed under the Apache License 2.0 - see the LICENSE file for details.

## Acknowledgments

This project builds upon the excellent work of the [OpenTelemetry Semantic Conventions](https://github.com/open-telemetry/semantic-conventions) project and the broader OpenTelemetry community. We are grateful for their foundational contributions to observability standardization.

---

Maintained by Alibaba Observability Team

For enterprise support and integration assistance, please contact: [support information]
