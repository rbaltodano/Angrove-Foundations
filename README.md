<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="brand/angrove-app-icon-dark.png">
    <img src="brand/angrove-app-icon-light.png" alt="Angrove app icon" width="128">
  </picture>
</p>
<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="brand/angrove-logo-light-text.png">
    <img src="brand/angrove-logo-dark-text.png" alt="Angrove" width="360">
  </picture>
</p>

# Angrove Foundations

The shared product, design, and architecture source of truth for Angrove.

Angrove is a private, local-first study and conversation environment for serious
questions about philosophy, theology, Scripture, meaning, and human flourishing.
This repository defines the product and technical contracts for the iOS app,
which runs entirely on-device.

This repository is the source of truth used across the iOS app, the offline
tooling repository, Codex, and Claude. Agents should begin with [`CLAUDE.md`](CLAUDE.md), which explains
the current architecture and routes work to the relevant document.

## Current app captures

The iPhone app and native Mac app share the contracts documented here. October 22, 2026 captures show the current interface, including Mac Boards for arranging notes, images, Insights, and conversations.

[![Angrove Mac Board with Common Good study notes](https://raw.githubusercontent.com/rbaltodano/Angrove-iOS/main/Documentation/Screenshots/2026-10-22/mac-boards-1.png)](https://github.com/rbaltodano/Angrove-iOS/blob/main/Documentation/Screenshots/2026-10-22/mac-boards-1.png)

[Mac and iPhone screenshots](https://github.com/rbaltodano/Angrove-iOS/blob/main/Documentation/Screenshots/2026-10-22/README.md) · [Lay your ideas out — 11 seconds](https://github.com/rbaltodano/Angrove-iOS/blob/main/Documentation/Screenshots/2026-10-22/lay-your-ideas-out.mp4) · [Learn easier — 19 seconds](https://github.com/rbaltodano/Angrove-iOS/blob/main/Documentation/Screenshots/2026-10-22/learn-easier.mp4)

## Case studies

Four case studies document the product decisions, engineering work, measured results, and remaining limitations behind Angrove:

- [Turning conversations into an explorable map](case-studies/01-insight-tree.md)
- [Finding the failure between retrieval and generation](case-studies/02-evidence-and-retrieval.md)
- [Making a local model behave on a real phone](case-studies/03-on-device-runtime.md)
- [Keeping answers safe when the user moves on](case-studies/04-conversation-reliability.md)

[Read the overview and evaluation context](case-studies/README.md).

## Documentation

- [`MISSION.md`](MISSION.md) — product purpose and guiding principles
- [`DESIGN.md`](DESIGN.md) — visual design system
- [`FUNCTIONALITY.md`](FUNCTIONALITY.md) — functional blueprint
- [`SPECIFICATION.md`](SPECIFICATION.md) — technical specification
- [`MODEL-INTEGRATION.md`](MODEL-INTEGRATION.md) — model architecture and integration
- [`INSIGHT-TREE.md`](INSIGHT-TREE.md) — Insight Tree design, semantic layout, and data contract
- [`PERSISTENT_MEMORY_IMPLEMENTATION_PLAN.md`](PERSISTENT_MEMORY_IMPLEMENTATION_PLAN.md) — memory implementation plan

When architecture or behavior changes, update these documents alongside the
corresponding application or tooling work.

## The semantic layer

One of Angrove's defining ideas is that important concepts should be explorable
as relationships, not just as a sequence of messages. The system uses local
embedding and relatedness tools to place concepts in a semantic vector space.
Cosine similarity, centroids, clustering, and graph-layout rules help determine
which ideas belong together, how strongly they relate, and how those
relationships appear in the Insight Tree.

The language model helps identify and explain meaningful concepts, while the
embedding and graph layers handle numeric relatedness and topology. This
separation lets the app expose an interpretable map of a person's developing
ideas without asking the language model to invent relationship scores.

## The project at a glance

| Repository | Purpose |
| --- | --- |
| [Angrove iOS](https://github.com/rbaltodano/Angrove-iOS) | The SwiftUI iPhone app and native Mac study workspace. |
| [Angrove Backend](https://github.com/rbaltodano/Angrove-Backend) | Offline tooling: grounding-corpus pipeline, model conversion, and evaluation. Its FastAPI service is retired; the app does not call it. |
| **Angrove Foundations** | Shared product requirements, design language, data contracts, and model-integration decisions. |

## Where to start

- Read [`MISSION.md`](MISSION.md) for the product purpose and boundaries.
- Read [`FUNCTIONALITY.md`](FUNCTIONALITY.md) for the intended user experience.
- Read [`MODEL-INTEGRATION.md`](MODEL-INTEGRATION.md) before changing model or grounding behavior.
- Read [`SPECIFICATION.md`](SPECIFICATION.md) for the technical system contract.

This is an active research and product-development project. Some model weights,
corpora, evaluation outputs, and other generated assets live outside Git because
of their size and licensing constraints.
