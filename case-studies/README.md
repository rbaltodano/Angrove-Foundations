# Angrove: portfolio case studies

**Private, on-device AI for study and reflection.**

Angrove is an independent iOS product by Ryan Baltodano, combining a reading-oriented conversation interface, local language generation, source retrieval, and an interactive map of saved ideas. These case studies explain four product and engineering problems behind that experience.

| Case study | What it demonstrates |
| --- | --- |
| [Turning conversations into an explorable map](01-insight-tree.md) | Product design, semantic systems, interaction design, and preserving user control |
| [Finding the failure between retrieval and generation](02-evidence-and-retrieval.md) | Controlled AI evaluation, source grounding, and diagnosis across system boundaries |
| [Making a local model behave on a real phone](03-on-device-runtime.md) | Runtime integration, memory debugging, numerical correctness, and performance tradeoffs |
| [Keeping answers safe when the user moves on](04-conversation-reliability.md) | Asynchronous workflows, state ownership, persistence, and regression coverage |

For an AI Product Engineer application, lead with the runtime or retrieval case study. For a Design Engineer application, lead with the Insight Tree. The conversation reliability story supplies evidence of engineering beyond the visible interface.

## Reading these results

Written October 1, 2026, from project documentation, recorded experiments, and representative implementation and test sources. No new model experiments or test runs were performed for these writeups. Experimental results belong to the dated configurations described in each article; they are not blanket claims about the current app.

The project was previously named Aquinas, and its repository URLs retain that name. The implementation snapshot reviewed selects the stock LiteRT Community Gemma 4 E4B package. Some older runtime and progress summaries still describe E2B as current or E4B promotion as blocked. The articles preserve the original failed gate and later diagnostics rather than treating either as a complete release-validation record. E4B fine-tuning remains future work; the package is not described here as a custom-trained model or as confirmed QAT.

Evidence links point to fixed repository snapshots so the implementation and evaluation records remain inspectable as the project evolves.

## Project links

- [Angrove iOS](https://github.com/rbaltodano/Aquinas-iOS)
- [Offline tooling](https://github.com/rbaltodano/Aquinas-Backend)
- [Product and architecture documentation](https://github.com/rbaltodano/Aquinas-Foundations)
- [Website](https://angrove.app)
