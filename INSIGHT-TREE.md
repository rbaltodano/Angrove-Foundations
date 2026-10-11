# Insight Tree — Design Spec

> Status: active spec for the response-driven, model-assisted Insight Tree.
> Every tree is built on the phone with the bundled MiniLM embeddings; there is no backend.
> Conversation trees cluster saved Insights around on-device Node Concept seeds. The global
> Insight Library canvas uses the same clustering; its accepted bookmark snapshot is persisted
> separately from the live Insight Library. Study Topic trees are scoped by the topic UUID and
> render the accepted aggregate of their conversations' saved Insights.
> See `MODEL-INTEGRATION.md` for the system-wide model contracts, persistence, and implementation
> order. This file remains the source of truth for Insight Tree behavior.

## 1. Product model

The Insight Tree is the Canvas Mode view of a single conversation's conceptual structure. Two
levels of concept:

- **Insight** — a specific idea pulled from the conversation. Concrete; can be inspected, saved,
  quoted, or used to start new inquiry.
- **Node Concept (Node)** — a higher-level subject that groups related Insights or preserves a
  broader concept extracted from a response. Automatically generated labels name the nearest
  useful broader concept, even for a single saved Insight, rather than repeating the Insight's
  name. A bookmarked Insight stays a visible chip even when an older Node label repeats its
name; label equality never promotes or hides it. Explicit Make Node promotions retain the
promoted Insight's chosen name.

Edges encode relationship strength via **length**:

- **Insight → Node** line length ∝ relatedness to that Node. Shorter = more related. Floor **~48px**.
- **Node → Node** line length ∝ relatedness between the subjects. Floor **360px**.

Everything is **per-conversation**.

## 2. Lifecycle: response → automatic Node Concepts

After each newly completed assistant response is persisted, iOS schedules an on-device analysis
job, shown as `Update Insight Tree` in the shared Model Task queue, at background priority.
Each completed turn captures its own conversation ID and content directly in the queue; leaving
the page cannot discard it, and another mapping job cannot suppress it. Reopened trees refresh
when their persisted seeds change. It never backfills historical responses. Corpus-scope abstentions and general-knowledge answers are
skipped.

Opening or closing a tree does not duplicate an active or queued cluster-label/definition job.
These jobs share a reservation by tree scope and Node ID in the model queue; a reopened canvas
receives the existing job's persisted result. Queued work survives closure of its original canvas.
Cancellation releases the reservation for a later retry, while foreground preemption preserves it.
Distinct Nodes, trees, and completed-turn analyses remain independent jobs.

The on-device model returns the nearest useful broader concept organizing the turn and a summary
defining that concept in context (`insightTreeSeedCandidate`). It
does not choose IDs, membership, similarity, or layout. Highlighted terms remain definition
affordances, not automatic Insights. Application code then decides with bundled MiniLM:

- similarity **≥ 0.60** (`InsightTreeSemanticPolicy.newSubjectSimilarity`) to an existing Node
  Concept: the turn stays within that subject, and nothing is added;
- otherwise: store the subject as a new Node Concept seed (`LocalInsightTreeSeedStore`).

The first substantive question-answer pair therefore seeds the conversation's first Node Concept.
Later answers add one only when their subject is genuinely new. Examples, applications,
restatements, and merely useful facts stay out of the Tree unless the user bookmarks them as
Insights. Seeds and saved Insights go through the same clustering pass, so a related saved Insight
attaches under a seeded subject. Membership waits until both the saved Insight and its seeds
use the configured embedding provider; fallback and MiniLM vectors are never mixed. A seeded
Node remains a semantic membership anchor as Insights join, alongside its member centroid.
Assignments are persisted per conversation (separately from the global canvas); conversation
membership from the legacy shared assignment store is rebuilt once to repair misplaced clusters.
Bookmark changes on a mounted conversation tree prepare and reveal the updated snapshot immediately,
without waiting behind generative model jobs. Saves made while the tree is closed retain a pending
reveal for its next opening.

This choice uses embedding similarity rather than asking the model "is this new?", because the
model's judgment was order-dependent. (The retired development backend ran a richer, idempotent
analysis with exact-evidence validation and a 0.86 duplicate threshold; the on-device path does
not persist pending jobs across launches.)

### Duplicate parent labels

Cluster labels and conversation seed prompts ask for a specific broader concept, not a synonym or
a vague catch-all. Cluster generation rejects normalized title copies and superficial wrappers,
then permits one corrective attempt. A saved Insight that matches its existing Node's name causes
that Node to be relabelled through the shared background queue. Its stable ID, member Insights,
and position are preserved; the revised label and fresh definition survive reopening. This also
repairs previously cached duplicate labels. Explicit Make Node promotions and placed Midpoints
are exempt. Arbitrary semantic paraphrases cannot be guaranteed away by string validation.

When a displayed Node Concept name changes, its canvas label, docked card heading, and sheet
headings use the conversation title's shared fade-and-blur replacement animation. Reduce Motion
uses a short opacity transition instead.

### Manual save remains separate

1. A model response renders certain key terms/phrases **highlighted + underlined**. Foundational
   or prerequisite concepts receive priority, while genuinely interesting discovery terms may
   also be included to encourage useful rabbit holes. There is no quota.
2. Tapping one first checks the active conversation/source cache. A cached definition opens
   immediately, even while the model is occupied. Only a cache miss queues the model to
   **generate a definition of that term as it relates to this conversation** (contextual, not a
   dictionary lookup).
3. Choosing to **save** bookmarks that definition in the global Insight Library **and** attaches
   it to the active conversation's Insight Tree. The client persists that conversation association
   by Insight ID. It is not the trigger for automatic response analysis.
   If that term is already saved with a different contextual meaning, saving adds the new
   context-definition entry to the existing Insight card rather than creating a second card.
   Repeated meanings ignore source-context labels, case, whitespace, and punctuation. Long
   near-verbatim restatements also collapse, including a repeated substantive opening definition
   with different elaboration. The first saved entry and its ID survive. This check runs on save
   and when stored definitions are decoded, repairing existing duplicate cards' entries. Short
   meanings require exact normalized wording; differing explicit negation is preserved. This is
   a conservative text check, not a guarantee against arbitrary semantic paraphrases.

### Global Insight Tree updates

Saving or removing a global bookmark updates the Insight Library immediately but does not
silently regroup the Global Insight Tree. When Global Insights opens with bookmark changes that
are not in its last accepted snapshot, a confirmation pill appears above the model controls.
Choosing **Yes** replaces the snapshot with the current deduplicated bookmark library and rebuilds
the semantic canvas; choosing **No** preserves the current tree. Either answer settles the prompt
for that set of saved Insights: reopening the page does not ask again until an Insight is saved or
removed. Bookmark changes made inside Global Insights apply to the tree directly and do not raise
the prompt. A prompt left unanswered is offered again the next time the page opens.

Entrance history is persisted across global and conversation tree visits. Loading an accepted
snapshot into an empty canvas restores previously presented Insights and Nodes immediately;
only newly introduced items animate. Record the presentation when its entrance starts so leaving
mid-animation does not replay it on reopening. Discovery dots remain until the corresponding
card is opened. Opening an Insight also records it as seen and clears any pending entrance.

### Study Topic Insight Tree updates

A Study Topic tree aggregates every bookmarked Insight from every conversation whose
`studyTopicID` matches that topic: Insights saved from the conversation, plus any bookmarked
Insight that appears in its transcript (branch concepts, quoted chips, inline Insights), however it
was bookmarked. Unbookmarked Insights never enter the tree. Opening a topic offers an explicit
update, only when this aggregate differs from the last accepted one, before replacing it. Once
accepted, the aggregate is persisted on device under the topic UUID and rendered through the same
on-device clustering, so topics cannot borrow membership from one another or from a conversation
tree. The live
Angrove model and embedding provider are injected from the application composition root, never
replaced by preview mocks.

Seam: `openDynamicDefinition` first calls `AngroveModel.cachedDefinition`; on a miss,
`requestDynamicDefinition(for:)` calls the conversation-scoped `defineTerm` overload.
`LiteRTAngroveModel` is the live model (`UnavailableAngroveModel` when no package is installed),
and the definition path has no mock availability fallback. The response's id is re-stamped via
`ConceptDefinition.stableID(forTerm:)` rather than kept as the model's own id, so re-saving the
same term always dedups (`MODEL-INTEGRATION.md`: "Persistent IDs generated by application code,
never by either model").

The on-device conversation response marks key terms inline; those validated terms become the
highlighted, tappable terms that open this definition flow. There is currently no on-device
definition cache, so `cachedDefinition` always misses.

## 3. Relatedness (the core signal)

Every Insight has a relatedness to its Node, and every Node pair has a relatedness. Both come from
a single **relatedness provider**:

- **Decided provider:** `sentence-transformers/all-MiniLM-L6-v2` supplies normalized embeddings;
  application code calculates cosine similarity. Node embeddings are normalized centroids of their
  member Insight embeddings.
- **Current wiring:** `MiniLMEmbeddingProvider` runs the bundled MiniLM model on device for every
  tree: conversation, Study Topic, and the global Insight Library canvas. `NLEmbeddingProvider` is a
  degraded last resort when the MiniLM assets fail to load, and the default for previews. The two
  vector spaces are never compared with each other.

The fine-tuned Angrove language model generates contextual definitions, Node labels, blended
Insights, and Make Node children. It does **not** invent relatedness numbers.

Relatedness is a **soft target**, never an exact measurement — a set of pairwise distances almost
never embeds exactly in 2D, so line length reads as "roughly proportional." All thresholds/floors
live as **named constants** in one place.

## 4. Node membership & creation (lazy)

- If the tree has no existing Node, its first manually saved Insight creates one. A response-driven
  subject/seed Node may already exist before the first manual Insight.
- Each **new** Insight joins its most-related existing Node (member centroid or subject) when
  similarity is **≥ 0.35** (`InsightTreeGrowth.attachSimilarity`); otherwise it starts its own
  Node. At **≥ 0.50** (`tightSimilarity`) it is a tight member; between the two it hangs
  **loosely** and is a budding candidate (§6). Calibrated on a real device tree: Insights score
  0.50–0.78 against their own concept, related concepts 0.45–0.70 against each other.
- A concept's subject vector is its "label. summary" (seed) or generated "label. definition".
  A saved concept with an empty vector is re-embedded; it never counts as embedded.
- An Insight has exactly one **owning** Node (it may still have secondary "bridge" lines — see §8).
- **Near-duplicate Nodes fold together.** Membership is greedy and sticky, so an early split
  (different turns seeding neighboring subjects, or Insights saved before the Node they belong
  under existed) would otherwise last forever. After membership, two automatic Nodes merge when
  either signal clears its bar:
  - their member centroids reach **≥ 0.65** (`InsightTreeSemanticPolicy.nodeMergeSimilarity`),
    just above the highest "related but separate" calibration pair (Creation / Providence,
    0.64); or
  - their subjects — the seed's "label. summary", or a generated label with its definition —
    reach **≥ 0.75** (`nodeSubjectMergeSimilarity`). Members of one subject can be loosely
    related to each other (Thucydides / Polis, 0.31), so the subject signal is what folds
    "Ancient Greek Politics", "Ancient Greek States", and "Ancient Greek City-State" (0.80–0.89).
    It is set higher because a shared subject word inflates the score: "Divine Human Nature" /
    "Divine Essence" (0.70) stay apart.

  A Node with no Insights yet has no member evidence (its "centroid" is just its name), so it
  folds only on the subject bar: "Theological Virtues" and "Moral Virtues" (0.72) stay separate.
  A saved fold of a memberless seed that the subject bar does not justify is released.

  The larger Node survives (a seeded subject wins ties) and keeps its id, label, and position,
  and the absorbed Node's Insights move under it. The merge is persisted per tree, so an
  absorbed seed stays folded on reopening while its survivor exists. Make Node promotions and
  placed Midpoints are explicit and never merged.

## 5. Layout rules

- Insight→Node and Node→Node lengths follow §1's relatedness-with-floor mapping. Reuse
  `insightBondLength` / `mapDistanceToLength` (already min-floored) in `InsightTreeViewModel.swift`.
  Node→Node lengths are multiplied by `nodeNodeSpacingScale` (1.4), floor included, so concepts
  have more room while every connector keeps its length relative to the others. Saved layouts are
  scaled once about the world origin to match; Insight→Node lengths are unchanged.
- **The tree grows from saved parents** (`InsightTreeGrowth`, applied in `growTree`). Every Node
  Concept hangs from exactly one parent, and its position is placed off that parent, so lines
  and positions come from one decision and never drift apart:
  - Explicit parents come first: a Make Node origin, a Branch parent, a Midpoint's most similar
    source. A budded Node's parent is its origin (§6).
  - Any other Node keeps the parent saved for it (`aquinas.insight-tree.parents.v1:<scope>`); a
    Node without one joins the attached Node with the most similar subject (Prim's growth from
    the root). Concepts that share a Midpoint get +0.12 affinity so the Midpoint stays short.
    Parents chosen before a vector exists are never saved.
  - A newly placed Node goes off its parent at the parent bond length, in the direction where
    neither its parent line nor its lines to already-placed secondary partners cross any drawn
    line (tree or secondary), keeps clear of other Nodes' cores, leaves an open angle, and
    follows the branch onward (a Midpoint leans toward its other sources). Only a concept with a
    vector can be the root.
  - The first time a scope grows with real vectors, every Node is placed fresh from the root
    and recentered on the old tree (`grown-layout.v2`), so older layouts become coherent once.
    Far moves glide on the canvas, which reframes the tree if none of it is on screen.
  - A Midpoint's lines to its other sources are drawn fainter (`EdgeModel.isSecondary`) so the
    tree reads first.
- **No crossings (safety net):** after overlap separation, every rebuild turns the smaller
  hanging branch of a crossing Node→Node connector rigidly around the node it hangs from, which
  keeps bond lengths. A node on a loop may swing alone around one neighbor. A connector stretched
  beyond 1.4× its semantic length slides its hanging branch back toward that length. Each move
  is accepted only if it adds no crossing and no Node core overlap (half the orbit footprint).
  Only legacy Insight-style Midpoints stay pinned. Moved nodes anchor at their new positions.
- **Anchor & nudge (stability):** existing nodes keep their positions as **anchors** and are only
  weakly pulled toward updated targets (the existing anchor spring, `anchorSpringK = 0.04` in
  `InsightTreeCanvasView.swift`). Only **new / budded** items are freshly placed. A full re-solve
  runs **only** on an explicit "reorganize" action or a topology change (bud / merge). Adding one
  thing never reshuffles the map.
- **Persistent crowding:** cross-cluster Insight-label and concept-label collisions first use
  local rotation and separation. Contact lasting 0.75 seconds increases every concept anchor
  distance by the same 6% factor, up to 2×; Insight-to-parent lengths are unchanged. Expanded
  anchors keep their relative distances; actual positions remain soft targets. Spacing never
  automatically contracts. Expansion pauses during dragging, open-card focus, Study, Midpoint
  selection, and generation/transfer. Clear saved trees stay still; crowded saved trees get a
  bounded cleanup pass. Settled positions use the existing position persistence.
- **Crowding:** the floor is a *minimum*, not a target. The chip ring grows with count
  (`insightOrbitRadius`, `InsightTreeModels.swift`). Every visible Insight label participates in
  one global footprint collision pass, including Insights owned by different Node Concepts;
  contact rotates both free Insight bonds and separates their parent nodes. Within a Node Concept,
  Insight bonds repel one another more strongly than visible node-to-node edges, and new bonds
  seed into the largest open angular gap. When a Node is dense, the shorter=closer ordering is
  locally approximate — accepted.
- **Exact title match:** when an Insight's normalized title (ignoring case, punctuation, and a
  leading article) matches its owning Node label, the canvas draws only the Node. The Insight
  remains stored and available in the Node card for its contextual definition; only the redundant
  orbiting chip and connector are collapsed. Placed Midpoints are exempt because their Insight
  chip is their only visible representation.
- **Node graph is a tree:** one parent line per Node, plus a Midpoint's fainter lines to its
  other sources. It is never recomputed as a whole, so adding a concept never rewires others.
- **Hover-card replacement:** switching directly between hovered Insights animates the current
  card out and the new card in; do not mutate only the text of one persistent card.
- **Return to the tree:** remember the last hovered Insight or Node Concept independently for
  the global tree and each conversation tree, using protected local canvas storage. Reopening
  resolves its current content and position after the tree loads, restores its card, and focuses
  it without first fitting the whole tree. Explicit navigation requests take priority. A deleted
  target falls back to normal framing. The dot grid covers the full viewport at every zoom.

## 6. Automatic budding

When a Node has **3 or more loose** Insights (similarity to its subject below 0.50), they can
split into a new Node (`InsightTreeGrowth.budGroup`, run on every graph build):

- **Cohesion gate:** only bud a subgroup of those loose Insights that is *also* **mutually
  related**: 3+ members each **≥ 0.55** to the group's own centroid (a chain A~B~C does not
  count), averaging at least 0.05 closer to each other than to the origin, and with the group
  **< 0.62** to the origin so the §4 merge cannot fold it straight back. "Far from the parent" ≠
  "belongs together" — with no cohesive subgroup, **no bud**.
- If the group fits another existing Node better (≥ 0.50 and closer than to its origin), it
  moves there instead of budding, so no duplicate subject appears in another branch.
- **Only the loose, cohesive Insights move.** Well-related Insights stay on the original Node.
- The new Node's **label is generated from the shared subject** of the moved Insights (model).
- The new Node **stays linked to its origin Node** (parent line) plus any other related Nodes — the
  tree stays connected.
- **Sticky + hysteresis:** budding does **not** auto-reverse. Re-evaluate membership only for
  **newly-added** Insights, not settled ones. Reversal happens only by explicit user action; the
  only automatic merge is the near-duplicate fold in §4, whose threshold sits above any bud's
  similarity to its origin.
  This prevents Insights ping-ponging between Nodes on successive rebuilds.

## 7. Branch in Study (manual decomposition)

Branch is offered only for a studied Insight, never a Node Concept. It isolates the selected
Insight, takes a count of **2–6**, and promotes it to its own Node Concept with the same title.
The new Node stays linked to the original owning Node, including the single-member case.
Children are the idea's distinct, more fundamental subordinate points, not nearby associations,
examples, consequences, or a generic taxonomy. Legacy Make Node callers use three children.

All child content must be validated before release. During generation only the Node's text
shimmers. Children then burst out with 0.1–0.25 s gaps, a separate haptic and opposing recoil
for each. Presentation state is temporary; promotion metadata, stable child IDs, generated
content, and the parent connection survive tree recreation. Failure/cancellation restores the
original Insight and removes the pending promotion.

Branch children are embedded with the configured provider before they are released. Their
relatedness determines connector lengths through the same mapping as other tree members; Study
preserves those individual radii instead of placing them all at the cluster's mean radius.
The burst uses the tree's bond directions and radii, fitted uniformly to the viewport. Once it
finishes, the normal tree renders the children, their connectors, and the stable parent edge.
Study focuses the promoted Node and keeps its connected parent visible as context. Saved
children refresh stale embeddings when preparing the semantic tree.

## 8. Midpoint tool

Select **2–8** targets (Insights and/or Node Concepts). The draggable handle sets blend weights
(linear along the segment for two targets; normalized inverse-distance for larger selections).
Committing generates five shared-region candidates and uses the configured embedding provider to
select the candidate nearest the weighted source-vector centroid.

The completed result is a **Node Concept**, with its own generated title, definition, and embedding.
It creates no synthesized saved Insight. Each selected Insight transfers to this concept as its
single owner; their former Node Concepts remain visible even when emptied. Selected Node Concepts remain connected by concept-to-concept edges. Source Insights
retain their IDs, content, and bookmarks. Their distances are recomputed against the new concept's
embedding, through the normal bond-length mapping. All selected Insights remain visible, up to eight.

Generation and embedding must succeed before ownership changes. The canvas preserves each source's
old position, extends a line from the new concept until it reaches the Insight, fades the old parent
line, then pulls the Insight
to its new semantic orbit using the same 0.5-response, 0.8-damping rubberband spring as Insight
drag release. Chips and connector endpoints share one progress clock, including overshoot and settle. The loading
presentation flashes the Node Concept symbol (`brain.head.profile`) for Midpoints and Make Node/Branch children, and retains its minimum dwell; reopening renders the committed topology without replaying
the transfer. Reduce Motion applies the final arrangement immediately. Failed or cancelled generation removes the placeholder and preserves the original tree.

Node Concepts and source IDs persist independently in each tree scope. Reusing an Insight in another
Midpoint gives the latest surviving Midpoint ownership. Deleting a source drops its member connection;
removing a Midpoint releases its members back to normal membership or an earlier surviving Midpoint.
Legacy saved Midpoint Insights retain their old representation until explicitly replaced.

## 9. Deletion & edge cases

- Remove an Insight → repair its Node centroid (sticky; no auto-un-bud). Legacy
  `automatic_response` Insight rows still use source-response tombstones so old/reconciled data
  cannot be recreated; the current response-analysis flow creates Node Concepts instead.
- Remove a Midpoint's source → drop that one connector, keep the Node Concept.
- Every Node Concept is automatically bookmarked, including response seeds, automatic clusters,
  Make Node/Branch, and completed Midpoints. Its bookmark is a concept anchor, never an additional
  Insight chip. Bookmarks and removal decisions persist in the existing encrypted tree stores.
- Unbookmarking a Node Concept, from its card, its long-press menu, or the Insight Library, asks
  whether to redistribute its actual attached Insight count (singular/plural). **Delete Insights**
  removes the concept and those Insight bookmarks. **Redistribute** removes only the concept;
  **Cancel** keeps the group. Each retained Insight attaches once to the most
  semantically relevant of the nearest three surviving concepts, falling back to spatial proximity
  when vectors are unavailable. With no surviving concept, normal clustering creates a new home.
  Explicit removals suppress old seeds and historical Midpoint parents across reloads. No automatic
  reattachment to an earlier Midpoint overrides this choice.
- Concept connectors preserve relationship endpoints and semantic spacing. Presentation routing
  tries bounded quadratic bends around other groups' labels. Bends are chosen in world space
  (label footprints at the default zoom), so panning and zooming never change a line's curve; a
  layout change bends it smoothly into its new curve. Remaining connector crossings
  receive a small gap on one line, while shared endpoints stay joined. Reveal and pulse effects
  follow the routed curve. Dense layouts may still need the existing adaptive spacing expansion.
- Nothing related to anything (early, or weak stub relatedness) → the nearest-neighbor fallback (§5)
  keeps the map connected and anchored.

## 10. Model seams

Every generative call funnels through the `AngroveModel` protocol (`Angrove-iOS/Services/`);
swapping in the real model is one conforming type plus overriding
`EnvironmentValues.angroveModel` — see `MODEL-INTEGRATION.md` §3 for the structured contract each
method follows, and `AngroveModel.swift`'s doc comments for the current wiring
detail per method.

| Seam | `AngroveModel` method | Called from | Wired into the live flow? |
| --- | --- | --- | --- |
| Conversation response + key terms | `respond(to:)` | `ConversationComponents.appendSimulatedResponse` | Yes |
| Conversation compaction | `compact(_:)` | `CurrentConversation.compactContext` | Yes |
| Cached contextual-definition lookup | `cachedDefinition(for:in:conversationID:)` | `CurrentConversation.openDynamicDefinition` | Yes |
| Term → contextual definition | `defineTerm(_:in:conversationID:)` | `CurrentConversation.requestDynamicDefinition` | Yes |
| Node subject / label | `labelSubject(forTitles:)` | `InsightTreeViewModel.generateSuggestedNode` | Yes |
| Midpoint blend | `blendConceptCandidates(_:weights:)` | `InsightTreeView.requestMidpointDefinition` | Yes — generated candidates are selected by vector distance and replace an identity-only loading item |
| Make Node children | `generateChildren(for:)` | `InsightTreeViewModel.generateMakeNodeChildren` | Yes — exactly three validated children replace identity-only reveal state |

Relatedness has one boundary in the `Services/` folder: `EmbeddingProvider`, implemented by
`MiniLMEmbeddingProvider` (live) and `NLEmbeddingProvider` (fallback and previews).

Persistent IDs (Insight/Node/term ids) are always assigned by application code, never returned by
the model — see `stableUUID(from:)` (`InsightTreeViewModel.swift`) and
`ConceptDefinition.stableID(forTerm:)` (`ContentView.swift`), which re-stamps a saved term's id
from its canonical text so re-saving the same term always dedups instead of minting a duplicate.

## 11. Code map

- `Angrove-iOS/Services/AngroveModel.swift` — the model boundary protocol, `ConversationContext`,
  `KeyTerm`/`ModelResponse` (see §10), and the `EnvironmentValues` injection points.
- `Angrove-iOS/Services/LiteRTAngroveModel.swift` — live on-device model.
- `Angrove-iOS/Services/UnavailableAngroveModel.swift` — explicit failures when no model is
  installed.
- `Angrove-iOS/Services/MockAngroveModel.swift` — previews and tests only.
- `Angrove-iOS/Services/EmbeddingProvider.swift` — the relatedness-vector boundary protocol +
  `NLEmbeddingProvider`; `MiniLMEmbeddingProvider.swift` is the live implementation.
- `Angrove-iOS/Features/Conversation/ModelTaskQueue.swift` — serialized question, definition, and
  tree-update jobs, including stop/remove/reorder state.
- `Angrove-iOS/Persistence/ConversationInsightMembershipStore.swift` — client-side association
  between global saved Insights and a conversation.
- `Angrove-iOS/Persistence/LocalInsightTreeSeedStore.swift` — on-device Node Concept seeds per
  conversation.
- `InsightTreeViewModel.swift` — membership, layout targets, budding, promotion, midpoint placement,
  `ensureEmbeddings`/`refreshEmbeddingsIfNeeded` (the versioned embedding pipeline).
- `InsightTreeCanvasView.swift` — rendering + physics (anchor spring, VSEPR chip-angle spread,
  simulated-annealing cooling), gestures, Make Node / Midpoint interaction.
- `InsightTreeView.swift` — SwiftUI shell, docked Insight / Node cards, orchestration.
- `SemanticLayout.swift` — MDS + Procrustes engine, retained for future "advanced folding."
- `InsightTreeModels.swift` — `InsightModel` / `NodeModel` / `EdgeModel`, `insightOrbitRadius`.
- Reusable primitives: `insightBondLength`, `mapDistanceToLength`, `semanticDistance`,
  `chainExtensionPosition` / `maximizedGapAngle`, `stableUUID(from:)`.

## 12. Tunable constants (one home)

`insightNodeMinLength ≈ 48` · `nodeNodeMinLength = 360` ·
`newSubjectSimilarity = 0.60` · `attachSimilarity = 0.35` · `tightSimilarity = 0.50` ·
`nodeMergeSimilarity = 0.65` · `nodeSubjectMergeSimilarity = 0.75` ·
`budMinimumCount = 3` · `budCohesion = 0.55` · `budMaximumParentSimilarity = 0.62` ·
`budSeparationMargin = 0.05` · `sharedMidpointAffinity = 0.12` · `nodeNodeSpacingScale = 1.4` ·
`anchorSpringK = 0.04`. All placeholders until re-tuned against the real model's relatedness.
