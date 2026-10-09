# Making a local model behave on a real phone

**Moving AI onto the iPhone exposed failures that simulator quality scores and successful model loading could not predict.**

Scope: LiteRT-LM integration, model evaluation, memory instrumentation, GPU precision, and lifecycle ownership. Evidence: September 25–29, 2026, principally on a base iPhone 17. Current local implementation: stock LiteRT Community E4B with F32 GPU activations; no E4B fine-tune.

## The product constraint

Angrove's privacy model requires generation, source retrieval, and semantic exploration to run locally. A remote model cannot serve as an invisible recovery path. The earlier Mac-hosted development client was removed, leaving one process-scoped native runtime and a shared queue for model work.

The challenge was larger than fitting weights on disk. The app had to preserve conversations, avoid overlapping native engines, survive lifecycle changes, and produce correct answers under the phone's actual numerical configuration.

## Keeping a failed performance gate visible

The original E4B physical-device gate recorded a 12.824-second cold load and a 21.910-second cold short-answer completion, exceeding its 12-second and 4-second budgets. The valid comparison trial's observed peak footprint was approximately 1.09 GB.

The gate remained failed. Later cached-load diagnostics provided additional information rather than retroactively passing the original criteria. Three cached trials completed their short answers in 16.13–17.44 seconds, including loading. This separated the cost of acquiring an engine from the cost of generating with one already resident.

## Discovering that measurement could distort the result

A separate memory problem came from DEBUG identity logging. The SHA-256 routine read the model in chunks, but temporary buffers accumulated across the task. A local reproduction recorded approximately 3.87 GB of peak physical footprint in the original implementation, compared with approximately 14 MB when each chunk was bounded by an autorelease pool. Both produced the same digest.

That result explained why a diagnostic build could impose severe memory pressure independently of model inference. The affected comparison was marked invalid, and later trials used the corrected harness. It did not establish the exact cause of every earlier process exit or daily-use failure.

## Correct retrieval, wrong chapter

Phone testing then exposed a correctness failure: the GPU answered a John 14 question as John 4 even when the rendered prompt contained the right retrieved evidence. Similar failures appeared for Psalm 23 and John 11. Phone and simulator CPU paths answered the identical prompt correctly.

Changing the phrasing did not resolve it. Forcing F32 GPU activations did: eight diagnostic chapter questions were answered correctly in the recorded follow-up.

The tradeoff was measurable. Observed peak footprint increased to 1.29–1.33 GB; short answers took about half a second longer, and long-answer decoding was approximately 20% slower in the recorded comparison. The implementation chose correctness over the faster F16 configuration.

## Evaluating an optimization instead of assuming a win

Speculative decoding was also tested on the phone. Three on/off trials per condition showed a small median generation improvement of roughly 0.4 seconds, with greater variance and changes in greedy wording. It was disabled for the launch configuration rather than justified by external throughput claims.

## Outcome and boundaries

The current runtime encodes F32 activation selection and disables speculative decoding by default. A later 40-case phone-GPU run passed 26 objective cases; the earlier simulator E4B run passed 28. The phone run had no recorded crashes, but did not repeat the blind rubric review or compare B0 on the same phone at F32.

The work demonstrates component-level fixes and informed deployment choices. It does not establish that every original promotion requirement passed. The unresolved questions include sustained multi-turn behavior, broader source accuracy, and a complete comparison under matching device conditions.

## Evidence

- [Original execution plan](https://github.com/rbaltodano/Aquinas-iOS/blob/2513a31ea7dceee65f6384725320d867e8a6b718/Documentation/Gemma4-E4B-QAT-Plan.md)
- [Original gate and hashing investigation](https://github.com/rbaltodano/Aquinas-iOS/blob/2513a31ea7dceee65f6384725320d867e8a6b718/Documentation/Gemma4-E4B-QAT-Progress.md)
- [Later phone diagnostics and precision results](https://github.com/rbaltodano/Aquinas-iOS/blob/2513a31ea7dceee65f6384725320d867e8a6b718/Documentation/Gemma4-E4B-Post-C9-Diagnostics.md)
- [Current runtime configuration](https://github.com/rbaltodano/Aquinas-iOS/blob/2513a31ea7dceee65f6384725320d867e8a6b718/Angrove-iOS/Services/LiteRTAngroveRuntime.swift)
- [Current package identity](https://github.com/rbaltodano/Aquinas-iOS/blob/2513a31ea7dceee65f6384725320d867e8a6b718/Angrove-iOS/Services/LiteRTModelStore.swift)
