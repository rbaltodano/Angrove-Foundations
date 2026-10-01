# Finding the failure between retrieval and generation

**A controlled evidence experiment showed that some wrong answers began with selecting the wrong passage, while others persisted even with the right source.**

Scope: local retrieval, evidence ablation, evaluation fixtures, and source-aware routing. Experimental result: September 9, 2026, using the then-shipped Gemma package on simulator CPU; not a benchmark of the current E4B app.

## The problem

Angrove answers questions about texts where attribution and context matter. A plausible explanation is insufficient if it gives the wrong council date, confuses a biblical chapter, or treats an objection in a philosophical text as the author's conclusion.

When an answer was wrong, several components could be responsible: missing source material, poor retrieval, or the model's interpretation of evidence. Changing prompts or replacing the model before separating those causes would make improvement difficult to measure.

## An experiment that isolated the evidence

The evaluation kept the model package, production conversation instructions, deterministic decoding, and empty history fixed. Only the evidence block changed:

1. Passages selected by automatic retrieval, with the curated layer disabled.
2. Manually verified whole chunks from the corpus.
3. No source passages.

Fifteen answerable questions had all three conditions. Two additional questions had automatic-retrieval and no-passage conditions only, yielding 49 recorded responses. Condition order rotated within questions. Answer-specific rubrics counted material errors as failures, even when the surrounding prose sounded reasonable.

The experiment bypassed canned-answer shortcuts and the adapter's later accuracy audit. Its purpose was to measure evidence use through the runtime, not the complete app's answer quality.

## Results

For the same 15 answerable questions:

| Evidence condition | Correct answers |
| --- | ---: |
| Automatic retrieval | 6/15 |
| Verified corpus chunks | 9/15 |
| No passages | 8/15 |

The clearest difference appeared in seven council and creed questions: automatic retrieval produced 2/7 correct answers, verified chunks 6/7, and no passages 3/7.

Those results pointed to a retrieval bottleneck in that subset. The corpus contained useful evidence, but the automatic path often failed to select it. They also exposed a separate generation problem: the model failed the lying question in all three conditions, including the verified-source condition. Supplying the right text did not guarantee a correct interpretation.

## Translating the diagnosis into retrieval design

The current on-device provider combines several kinds of lookup. Explicit scripture citations resolve to actual chapters. Named documents restrict the search to their sources. Authority-section routing selects the relevant primary-text section. Semantic ranking handles broader conceptual questions. Curated notes cover selected known gaps.

These routes recognize that “John 14” is a document address, while a question about justice is a semantic query. Both should not be handled by unconstrained similarity search alone.

Regression tests cover council questions retrieving appropriate evidence, explicit citations reaching the correct text, named documents avoiding irrelevant front matter, and out-of-corpus questions returning no grounding. Curated notes and verified-response shortcuts remain distinct from general corpus retrieval.

## Outcome and boundaries

The documented outcome is a diagnosis and a more source-aware implementation. There is no recorded repeat of the same 49-response fixture establishing a new end-to-end accuracy percentage, so the 2/7-to-6/7 comparison should not be presented as a shipped retrieval improvement.

This was a small, human-assessed diagnostic sample on simulator CPU. It does not establish statistical significance, phone performance, or CPU/GPU parity. Its practical value was identifying which component to investigate next and showing that retrieval quality and evidence interpretation need separate evaluation.

## Evidence

- [Experiment design and execution](https://github.com/rbaltodano/Aquinas-Backend/blob/0d8c08759807a6b651f3235ff7d00a138faffc10/evaluation/evidence_ablation/README.md)
- [Results and limitations](https://github.com/rbaltodano/Aquinas-Backend/blob/0d8c08759807a6b651f3235ff7d00a138faffc10/evaluation/evidence_ablation/RESULTS.md)
- [Per-response assessments](https://github.com/rbaltodano/Aquinas-Backend/blob/0d8c08759807a6b651f3235ff7d00a138faffc10/evaluation/evidence_ablation/assessments.json)
- [Current on-device retrieval provider](https://github.com/rbaltodano/Aquinas-iOS/blob/2513a31ea7dceee65f6384725320d867e8a6b718/Angrove-iOS/Services/MiniLMGroundingProvider.swift)
- [Retrieval regression tests](https://github.com/rbaltodano/Aquinas-iOS/blob/2513a31ea7dceee65f6384725320d867e8a6b718/Angrove-iOSTests/MiniLMGroundingRetrievalTests.swift)
