# Post-Launch Features

This document tracks product ideas and model upgrades intentionally deferred until after launch. These
features should return only when their behavior is implemented, testable, and accurately described
in the app. A stored preference without a runtime consumer is not a finished feature.

## Custom Instructions

Allow a user to provide durable guidance that applies to ordinary conversations.

Before shipping:

- Define precedence relative to Angrove's stable system identity and conversation personality.
- Apply instructions only to ordinary conversation. Structured tasks such as definitions,
  compaction, Insight Tree analysis, labels, and retrieval should remain neutral.
- Set a clear size limit and show the user exactly where the instructions apply.
- Defend against instructions that request fabricated quotations, citations, or hidden reasoning.
- Add prompt-assembly tests for empty, conflicting, oversized, and adversarial instructions.
- Keep the instructions on device with the rest of the user's private conversation data.

## Conversation Memory

Cross-conversation memory needs a typed data model, an explicit review/delete surface, and a clear
privacy contract before it appears in Settings. It must distinguish user-approved personal facts
from inferred themes and from ordinary conversation history. No silent memory extraction.

## Fine-Grained Model Behavior

Deferred controls:

- Conversational initiative
- Knowledge level
- Intellectual challenge
- Theological framing
- Response format
- Personality (Balanced, Scholarly, Socratic, Fun Mode): removed from Settings for launch; the app
  always uses Balanced. The prompt variants remain in the model layer.

These controls should be introduced only with stable prompt contracts and evaluation cases proving that each
choice causes a distinct, predictable behavior without weakening factual safeguards.

## Research and Citations

Deferred controls:

- Citation preference
- Citation format
- Link handling

These depend on a licensed, versioned local source corpus and a response contract that can tie
claims to retrieved passages. Citation formatting should not launch before citation correctness.

## Insight and Study Preferences

Deferred controls:

- Automatic Insight Mapping modes
- Definition Highlighting modes
- Question of the Day focus

Launch behavior should remain one understandable automatic policy. Mapping and highlighting
preferences can return after semantic thresholds and definition-intent behavior are calibrated
against real conversations. Question of the Day focus can return when local generation reliably
supports subject selection.

## Local-First Follow-Up

The iPhone is the canonical source of truth, and the app no longer has a backend client. Home's
Loose Thread, Terms You Glossed Over, Today in History, and Your Quote now run on device. Still
open:

- Broader Today in History coverage (the bundled catalog covers 75 dates)
- Retrieval, citations, and claim support over the local source corpus

Work that depends on the final text model or licensed source library should wait for those inputs.
Deterministic storage, selection, and MiniLM relatedness work can proceed independently.

## EmbeddingGemma 2 Semantic Retrieval Upgrade

**Deferred until after launch by the owner on 2026-10-07.** Evaluation and implementation are
post-launch work; retain MiniLM for the launch checkpoint.

After launch, evaluate the 270M text-only EmbeddingGemma 2 package against MiniLM on our local
source corpus. Adopt it only if relevant evidence delivery improves without regressions in
irrelevant grounding, citations, answer quality, or base-iPhone-17 performance with E4B resident.
Scope includes LiteRT runtime compatibility, corpus re-embedding, and retrieval-threshold
calibration. Decide Insight Tree calibration and persisted-vector migration separately. Add
multimodal packages only when a concrete image, audio, or video feature needs them.

See the [assessment and proposed evaluation scope](../Angrove-iOS-main/Documentation/EmbeddingGemma2-Assessment.md).

## Image and File Inputs

**Deferred until after launch by the owner on 2026-10-07.** Photo-library selection, recent-photo
shortcuts, camera capture, and file attachments are excluded from launch. The plus menu offers
saved Insights and Library passages only; image/file upload actions are removed from other menus
as well. Existing saved attachments remain readable and persisted.

Before reintroducing inputs, define supported formats and size limits, implement reliable
on-device processing, verify model capabilities and resource costs, and provide clear handling
for unsupported or unreadable content. Conversation backup import/export remains available as
data management, independently of model inputs.

## Quantization-Aware Training (QAT) Model Candidate

**Closed for the current shipping checkpoint as of 2026-08-13.** The 4-bit DWQ mechanism worked,
but the retry against the actual production 8-bit quantization scheme did not produce a deployable
checkpoint: attempts encountered non-finite gradients, Metal out-of-memory failures, worse
validation, and nondeterministic validation that prevented exact checkpoint/resume verification.
The shipping 8-bit model already has very little quantization error, so the likely product benefit
does not justify more research or hardware time. Production remains on the verified 3.86 GB
`dynamic_wi8_emb4_afp32` package.

Retain the tooling, but do not schedule another retry for this checkpoint. Revisit DWQ only for a
future checkpoint with a demonstrated quantization problem, an intentional lower-bit package with
a meaningful size/latency payoff, or after deterministic validation is proven and stronger hardware
is already available. Prioritize checkpoint/data quality, production-path evaluation, and runtime
optimization instead.

See [Aquinas-QAT-DWQ-Writeup.md](Aquinas-QAT-DWQ-Writeup.md) for the full experiment history,
memory analysis, retained tooling and patches, controlled-retry rejection, and final recommendation.
