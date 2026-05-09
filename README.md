# LoongSuite GenAI Semantic Conventions

LoongSuite GenAI Semantic Conventions is an **open-source GenAI semantic conventions repository from Alibaba** that provides enhanced Generative AI observability standards, building upon the foundation of [OpenTelemetry GenAI Semantic Conventions](https://github.com/open-telemetry/semantic-conventions-genai).

## About

This repository extends the [OpenTelemetry GenAI Semantic Conventions](https://github.com/open-telemetry/semantic-conventions-genai) with additional GenAI-specific semantic conventions derived from Alibaba's internal observability practices. It defines a comprehensive set of semantic attributes, spans, metrics, and events specifically designed for tracing and monitoring Generative AI applications and services, using [Weaver](https://github.com/open-telemetry/weaver) to manage dependencies on the core semantic conventions.

### Key Features

- **GenAI-Focused**: Specialized semantic conventions for LLM applications, model interactions, and AI service observability
- **Production-Tested**: Based on real-world experience from Alibaba's internal AI infrastructure
- **OTel-Compatible**: Built on OpenTelemetry standards, ensuring seamless integration with the OTel ecosystem
- **Community-Driven**: Open-sourced to benefit the broader GenAI observability community

## Documentation

The human-readable version of the semantic conventions resides in the [docs](docs/README.md) folder. These Markdown documents are generated from the YAML definitions located in the [model](model/README.md) folder, following the OpenTelemetry Semantic Conventions toolchain approach.

## Relationship with OpenTelemetry

LoongSuite GenAI Semantic Conventions extends and supplements the official [OpenTelemetry GenAI Semantic Conventions](https://github.com/open-telemetry/semantic-conventions-genai). While OTel GenAI Semantic Conventions provides foundational semantic conventions for GenAI clients, MCP (Model Context Protocol), and provider-specific conventions (OpenAI, etc.), LoongSuite focuses on additional GenAI observability patterns that emerged from Alibaba's production AI workloads.

## Contributing

We welcome contributions from the community! Whether you're adding new GenAI conventions, improving documentation, or fixing issues, your input is valuable.

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines on:

- How to propose new semantic conventions
- Code of conduct
- Development workflow
- Testing and validation requirements

## Acknowledgments

This project builds upon the excellent work of the [OpenTelemetry GenAI Semantic Conventions](https://github.com/open-telemetry/semantic-conventions-genai) project, the [OpenTelemetry Semantic Conventions](https://github.com/open-telemetry/semantic-conventions) project, and the broader OpenTelemetry community. We are grateful for their foundational contributions to observability standardization.

## Community

We are looking forward to your feedback and suggestions. You can join
our [DingTalk group](https://qr.dingtalk.com/action/joingroup?code=v1,k1,mexukXI88tZ1uiuLYkKhdaETUx/K59ncyFFFG5Voe9s=&_dt_no_comment=1&origin=11?) or scan the QR code below to engage with us.

| GenAI SemConv SIG | Python SIG | Java SIG | Go SIG |
| --- | --- | --- | --- |
| <img src="docs/assets/img/loongsuite-semconv-sig-dingtalk.png" alt="GenAI SemConv SIG" height="150"> | <img src="docs/assets/img/loongsuite-python-sig-dingtalk.jpg" alt="Python SIG" height="150"> | <img src="docs/assets/img/loongsuite-java-sig-dingtalk.jpg" alt="Java SIG" height="150"> | <img src="docs/assets/img/loongsuite-go-sig-dingtalk.png" alt="Go SIG" height="150"> |

## Resources

* AgentScope: <https://github.com/modelscope/agentscope>
* Observability Community: <https://observability.cn>

## License

This project is licensed under the [Apache License 2.0](LICENSE)
