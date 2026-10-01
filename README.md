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

The shared product, design, and architecture documentation for Angrove.

This repository is the source of truth used across the iOS app, backend, Codex,
and Claude. Agents should begin with [`CLAUDE.md`](CLAUDE.md), which explains
the current architecture and routes work to the relevant document.

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
- [`INSIGHT-TREE.md`](INSIGHT-TREE.md) — Insight Tree design and data contract
- [`PERSISTENT_MEMORY_IMPLEMENTATION_PLAN.md`](PERSISTENT_MEMORY_IMPLEMENTATION_PLAN.md) — memory implementation plan

When architecture or behavior changes, update these documents alongside the
corresponding application or backend work.
