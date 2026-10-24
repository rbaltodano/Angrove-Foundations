# Angrove-Foundations

Docs-only repo: the product source of truth for the Angrove project. No code lives here.

## Working principles

- Make the best decision that will elevate all three: the user experience (UX), developer
  experience (DX), and agent experience (AX) always, without breaking anything.
- Cut taglines, helper text and secondary labels. If a screen needs them to make sense, fix the
  layout.

## Model integration is live

Before changing model behavior, conversation responses, dynamic definitions, embeddings, or the
Insight Tree, read [MODEL-INTEGRATION.md](MODEL-INTEGRATION.md). It is the cross-repo source of
truth for what is already implemented and what remains.

Current checkpoint: the app runs entirely on-device. The iOS HTTP client for the Mac backend was
removed on September 26, 2026; `../Angrove-Backend` is now offline tooling only (corpus, model
conversion, evaluation). Safe user-facing approach summaries, validated tappable key terms, and
context compaction run through local LiteRT, behind a priority-aware visible task queue; LiteRT
delivers completed text after native decoding rather than reliable token deltas. Bundled MiniLM
(`all-MiniLM-L6-v2`, 384 dimensions) handles Insight Tree relatedness on device. Automatic response extraction creates Node
Concepts, while manual definition saves create Insights. Do not replace these working paths with
mocks or assume they still need their first implementation.

The finalized prompt architecture has one runtime identity assembler, `LiteRTAngroveModel`
(ported from the retired `../Angrove-Backend/main.py`). The global layer contains Angrove's stable reasoning constitution,
hidden-reasoning protection, and structured-task precedence. Conversation personalities are
injected only into ordinary conversation; definitions, daily questions, extraction, labels,
Midpoints, Make Node children, compaction, and repair remain neutral.

The exact fine-tuned 2.72 GB LiteRT package can be tested without a phone cable through the arm64
iOS Simulator on Apple silicon. Ordinary local conversation stays deterministic because sampled
decoding corrupts this 4-bit checkpoint. It uses separated phrase-loop recovery and mixed-script
corruption rejection. Prompt-forced depth and short-answer retries were tested and removed because
they caused slow local failure; depth work now requires a pre-LiteRT-versus-LiteRT quality comparison.
That comparison showed a coherent 338-word Mac-checkpoint answer versus mobile-package failure.

On-device ordinary conversation generates visible prose first, then runs a separate validated
`key_terms` metadata pass. The public approach summary is app-generated, not hidden
chain-of-thought, and is persisted so **Show Thinking** survives response completion and reopening.
`AngroveGroundingProviding` is the new factual-retrieval seam. Its initial offline lexical catalog
grounds the known Nicaea/Constantinople/Didache regressions; production still needs a versioned
trusted corpus ranked with `all-MiniLM-L6-v2`. Do not describe the small bootstrap catalog as broad
RAG coverage or factual verification.

The local adapter isolates clear subject changes from prior history and retries once if the engine
returns an earlier answer verbatim. Direct-definition cards use an intent gate rather than matching
every `what is` phrase. Conversation trees are seeded on device: the local model names each turn's
subject and MiniLM similarity decides whether it becomes a new Node Concept.

The fine-tuned Gemma package remains deployed but is not considered factually authoritative. A
2.66 GB Qwen3-4B mixed-INT4 candidate was rejected before generation: simulator Metal could not
allocate its 389 MB tensor, and the base iPhone 17 terminated during initialization. No replacement
has been selected. The intended quality architecture is a stronger compatible raw text model plus
MiniLM retrieval over an automatically ingested, licensed, versioned source library, citations,
claim validation, and explicit uncertainty when evidence is missing.

Never test external models by using `devicectl ... --domain-type appDataContainer
--remove-existing-content true` against the production bundle. Git source commits do not preserve
phone `UserDefaults`. Use a disposable probe bundle/container and take a verified app-data backup
before physical-device model experiments.

- [MISSION.md](MISSION.md) — why the product exists; use it to judge whether a feature fits.
- [DESIGN.md](DESIGN.md) — visual/interaction principles; the reference for design reviews.
- [SPECIFICATION.md](SPECIFICATION.md) / [FUNCTIONALITY.md](FUNCTIONALITY.md) — what the app should do.
- [MODEL-INTEGRATION.md](MODEL-INTEGRATION.md) — how the Angrove model, MiniLM embeddings,
  persistence, and iOS app work together; includes structured contracts and implementation order.
- [INSIGHT-TREE.md](INSIGHT-TREE.md) — current design spec for the model-driven Insight Tree:
  automatic Node Concept extraction, manual Insights, layout, relatedness, budding, Midpoint, and
  model seams.
- [PERSISTENT_MEMORY_IMPLEMENTATION_PLAN.md](PERSISTENT_MEMORY_IMPLEMENTATION_PLAN.md) — the
  planned SwiftData migration. The current iOS implementation uses an atomic Application Support
  JSON snapshot with rotating backups and legacy `UserDefaults` migration; do not describe it as
  SwiftData-backed.

Implementation happens in the sibling repos: `../Aquinas-iOS` (SwiftUI app — has its own CLAUDE.md with build instructions) and `../Angrove-Backend` (offline corpus, conversion, and evaluation tooling; its FastAPI server is no longer used by the app). If a session here turns into code changes, prefer starting/continuing it from the repo being changed.

Keep these docs in sync with reality: when a session decides something that contradicts or extends
a doc, update the doc as part of the work. Product requirements and implementation status must be
labeled separately. Private, on-device-only operation is both the production requirement and the
current implementation; do not reintroduce an HTTP model or data client.

## Gemma 4 12B QAT case-study chronicle

> **SHELVED 2026-09-25 by the project owner. Do not resume, rerun, or download anything for this
> initiative without explicit new owner direction.** The model path is now Gemma 4 E4B QAT
> (Aquinas-iOS `Documentation/Gemma4-E4B-QAT-Plan.md`). Read the "September 25, 2026 — initiative
> shelved" record in `research/rotated-ternary/STATUS.md` for what was removed and where it lives.

For any work under the rotated-ternary Gemma 4 12B QAT/custom-llama.cpp research initiative, read
`research/rotated-ternary/STATUS.md` and
`research/rotated-ternary/GEMMA4-12B-QAT-CASE-STUDY.md` before acting. The case study is a
required living record, not optional retrospective documentation.

At every material milestone—successful gate, failed gate, blocked dependency, artifact download or
verification, new runtime/format decision, device observation, resource/safety stop, or changed
next hypothesis—update both records in the same turn before handing off. Add a dated evidence-log
entry to the case study that states:

- what was attempted and why;
- exact model/runtime/artifact identities and commands where applicable;
- what went well, what failed or remained uncertain, and the evidence location;
- measured bytes, hashes, timing, memory, quality, or other relevant metrics (clearly label
  estimates); and
- the precise next action and any required cleanup.

For any test that produces a visible response, greedy continuation, or distribution comparison,
also update `research/rotated-ternary/MODEL-RESPONSE-COMPARISON-LEDGER.md`. Preserve the prompt,
template/sampler state, exact model/runtime identity, text or token IDs, metrics, and whether the
prompt set is development, held-out, or sealed. Add entries; never overwrite earlier comparisons.

Do not rewrite history, claim a passed result from a completed command, omit failed attempts, or
describe a stock-QAT control as a custom-format result. Update the date and current task in
`STATUS.md` at the same time. This requirement applies to every agent touching the initiative.
