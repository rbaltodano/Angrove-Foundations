# Angrove Functional Blueprint

> Current implementation reference for user-visible behavior. Product intent comes from
> `MISSION.md`; technical contracts live in `MODEL-INTEGRATION.md`; detailed tree behavior lives in
> `INSIGHT-TREE.md`.

## 1. Product mandate

Angrove is a sovereign cognitive interface: a private, local-first place for deep, uninterrupted
philosophical and theological inquiry. Its pillars are sovereignty, permanence, and rigorous
inquiry. The UI should communicate the structure of thought, not merely display model output.

Angrove currently supports portrait orientation only; landscape rotation is disabled.

## 2. Main application areas

- **Home** — Question of the Day, recent work, saved Insights, and entry points into new questions.
- **Conversation** — the active branching dialogue, contextual definitions, model
  activity, and entry into the conversation's Insight Tree.
- **Open Conversations** — search, selection, rename, pin/unpin, study-topic assignment, and
  deletion.
- **Study Topics** — topic containers and conversations assigned to them.
- **Insight Library** — globally bookmarked contextual definitions and its in-memory semantic
  canvas. Bookmark changes are applied to the Global Insight Tree only after the user accepts the
  update prompt shown above its model controls.
- **Settings** — appearance, Personalization (the preferred name Angrove uses to address you), response typography/alignment, related preferences, and a User Guide (Support) explaining the app's tools. Privacy & Data can delete all conversations and their conversation-specific history while preserving saved Insights, Clipped Passages, and Study Topics; delete all Clipped Passages while preserving conversations and saved Insights; or delete all app data, including saved content, settings, app-managed model downloads, and caches. Each action requires confirmation. Model Behavior has been removed because its only control was the preferred name. Model Activity and its display preference have been removed; the standard activity indicator remains available. Every internal page has introductory copy; settings prose uses Figtree with the selected text size, independently of the conversation response font.

Appearance includes separate background selections for the Conversation Thread and Insight
Tree: Light, Dark, System, and Clouds (the first image preset, shared with the website).
Selections persist across launches. System follows the app's color scheme; Light and Dark
set the selected surface's palette independently. Clouds uses a stationary sky image with a
dark tint and cream text for readability. The Insight Tree choice applies to global,
conversation, and Study Topic trees. Reset Settings restores both choices to System.

The sidebar's scrolling conversations fade into its own adaptive surface at the bottom in
light and dark mode. The pinned Settings and New Conversation buttons sit above the fade in a
transparent container; bottom content padding lets the final conversation scroll clear of them.

All Node Concepts are automatically bookmarked, including completed Midpoints. Unbookmarking a
concept from its card or long-press menu asks whether to redistribute its attached Insights,
showing their count. Redistribute moves them to relevant nearby concepts; Delete Insights removes
the group; Cancel keeps it. The Insight Library uses the same confirmation. Concept bookmarks
remain anchors rather than duplicate Insight chips.

The Insight Tree gradually increases all concept-to-concept spacing targets by one common factor
when cross-cluster label collisions persist after local settling. Insight lengths remain unchanged;
expansion is capped, pauses during interaction/generation, and never automatically contracts.
Concept connectors also bend around other groups' labels and show small gaps at remaining
crossings so they do not resemble joined relationships. Their actual graph connections stay intact.
Clear saved trees remain still, while crowded ones receive a bounded cleanup. Settled positions
continue to use the existing per-tree persistence.

Library catalog and reader loading states show the centered growing-leaf logo animation used by Home refresh, with “Loading” underneath in the app’s paragraph typography. Reduce Motion shows the completed leaf.

The Library reader provides complete bundled editions of its 37 works, organized by books,
chapters or sections, with original paragraph boundaries. It reads separate full-document assets
instead of presenting retrieval chunks as reading pages. "Complete" refers to the surviving
historical text and the named edition or collection section; lost ancient books are not fabricated.

The Library's illustrated listening card remains at the top after playback stops and shows the
most recently played work. Its button reads **Listen** while stopped or paused and **Listening**
while playing; tapping it resumes or pauses playback. A play icon appears while paused or stopped;
while playing, a context-style ring shows the current word's position through the complete work,
including skipped-ahead sections, rather than accumulated listening time. Passage of the Day is
not displayed in the Library. Tapping the rest of the card opens the saved listening position. Pausing saves
the exact word and marks its paragraph **Last listened here** in the reader; listening resumes
there even after the paused audio session expires or the app restarts. Each work retains its own
bookmark. Opening a work shows its saved listening marker without highlighting previously spoken
text. Spoken-word highlighting appears during listening after the person chooses Listen, and
clears when playback pauses or stops. The Home card remains tied to the active listening session.
When Listening appears, its ring animates from empty to the current work percentage. Library
spoken words ease into their 2-point lift over 0.4 seconds with the conversation reader's timing
curve; Reduce Motion applies the final positions immediately.

Only inline source/citation links and the expanded source's **Read More** button, opened from a
conversation thread into the Library, show **Return to Conversation** above Model Controls.
Other navigation, including Home entries, Insight actions, model-task entries, sidebar buttons,
and navigation from a conversation's local tree, omits return notifications and clears any pending
prompt. The source return prompt appears one second after the destination
finishes its page transition. The return pill uses Model Controls’ adaptive one-point capsule
border. Pending prompts are cancelled by another redirect or explicit navigation. The pill uses the return destination's
sidebar icon (Conversations for an individual conversation) and the Insight Tree confirmation's **Yes**/**No** buttons. Yes returns to the
conversation and restores its selection; No dismisses the prompt.
A subsequent redirect replaces the outstanding return prompt. Explicit sidebar navigation and
return actions clear it without creating another prompt. Startup selection does not show it.

Completed question and Insight Definition notifications open their completed content when tapped
anywhere on the pill except its **X**. The X collapses only that notification without opening its
content. Completion pills have no View button; return prompts retain their Yes/No actions.

User Guide topic rows show a blue dot until that topic is tapped. Visited topic IDs persist across
launches; opening the guide index alone does not mark any topic visited.
Home's Getting Started card is shown only during the first seven days and while at least one
User Guide topic remains unopened. Opening every topic removes it immediately. Its first-use
date persists across updates; older installs initialize that date from their data container's
Documents creation date when available.

## 3. Conversation and branching

The conversation plus menu offers saved Insights, Library passages, and Photos. Native image
recognition is enabled on iPhone builds after native recognition and memory checks. Simulator
builds keep the explicit text-only fallback by default. Camera is available when vision and
a camera are available.
Recent-photo shortcuts and file upload controls remain deferred. Previously saved attachments
remain readable. Conversation backup
import/export in Settings remains available for data management.

Submitted questions remain read-only and support native text selection and Copy. Their press
interaction no longer scales the question; the draft editor retains its press feedback.

With Automatic conversation titles enabled, the first completed answer is persisted and its
Insight Tree processing finishes before Angrove names the conversation from the user's initial
question. The generated title is saved for the conversation header and menus. Manual renames,
pinned entry-point titles, and the First Question/Manual settings take precedence. Failed naming
leaves the existing title intact; cancelled answers do not trigger naming.

The default view is a vertically scrolling dialogue. Each conversation can contain horizontally
arranged branches. A branch can begin from a quoted concept without destroying its parent line of
inquiry.

The model returns plain response text plus validated key-term metadata. Client code turns those
terms into underlined `aq://` links. Tapping one opens a contextual definition flow:

1. Check the active conversation's cache for the normalized term and source response.
2. If defined already, open it immediately even when another model task is active.
3. If absent, enqueue a `Define “…”` model task.
4. The completed definition can be quoted, saved, or read aloud. The last card action uses
   the same speaker icon as model responses and replaces the former Branch button. Insight,
   Node Concept, and Clipped Passage cards all offer it, including their Library and docked
   presentations. Reading uses the full definition or passage even when the card truncates it;
   tapping the active speaker again stops playback. The active speaker is accent green. Card
   words use the conversation reader’s green fill, glow, and lift as they are spoken; reading
   expands each card to its full text without an inner scrolling container. The word glow and
   hold-to-scrub transform can draw outside the card’s bounds. An expandable definition’s ellipsis
   fades and blurs out while reading; when reading ends, the definition collapses and the ellipsis
   fades and blurs back in. Hold the reading text and drag horizontally to rewind or fast forward.

The reader can also define a word the model did not mark. Long-pressing any plain word of a
completed response opens an edit menu with **Copy** and **Define**. Define wraps that word in the
same `aq://` link in the stored response and runs the flow above, so from then on it behaves as a
model-marked term. Surrounding punctuation, emphasis, and a possessive stay outside the link.

When a response sentence quotes a retrieved corpus passage, the client places an inline source
chip after that sentence (for example `John 14` or `Nicomachean Ethics, Book V`). Tapping the chip
opens the passage in the Library reader. Chips appear only where the response reproduces the
passage's own words; paraphrase and curated reference notes never get one.
The retrieved Source rows' **Read More** action also opens the containing section and scrolls
to the retrieved passage. Its work ID and chunk index survive saving and reopening the response.
Older saved excerpts recover their location only when they uniquely match a passage in that work;
ambiguous or unavailable locations retain the work-level fallback.
Sources in **Show Thinking** play their staggered entrance when the disclosure opens; scrolling
them out of and back into view does not replay the entrance.
5. Saving adds it to the global Insight Library and the current conversation's persistent tree.

Definitions contain a title, source-context label, and concise contextual meaning—not
pronunciation, part of speech, or an example. Saved Insights deduplicate by normalized title. If
the same term is saved again from a materially different context, the new meaning is appended to
the existing card as a secondary context-definition entry.

The conversation model also recognizes broad direct-definition wording such as “Define X,” “What
does X mean?”, and “What is X?” when X is a concise term rather than a broader inquiry or interface
control. Those answers include an in-text Insight card for the requested definition.

Queued questions display `Question queued` with the shared breathing animation. Queued definition
affordances breathe as well. Cancelling their model task clears the corresponding pending state.

## 4. Model behavior and task queue

The old Thinking toggle has been removed. Conversation responses use automatic routing: routine
questions take the direct-JSON fast path, while complex questions retain deep reasoning. Both may
show a short approach line and the retrieved sources, followed by the model's own reasoning from
Gemma 4's native thinking channel: live, one line at a time, then in full under **Show Thinking**.

Ordinary conversation follows a permanent reasoning constitution: represent the strongest
reasonable version of the user's argument, distinguish invalid inference from disputed premises
or missing evidence, revise conclusions when the user's reasoning warrants it, propagate the
revision through dependent conclusions, and keep confidence proportional to the evidence.
Conversation personalities may change expression but not this standard. Structured application
actions remain neutral regardless of personality.

Current conversation model work is scheduled through one priority-aware `ModelTaskQueue`:

- `User Question`
- `Define “<term>”`
- `Update Insight Tree`
- `Consolidate information`

Questions and definitions are foreground work. Tree updates wait for a five-second idle window and
yield when foreground work arrives.

The dock's Model Status reads `Idle` when empty and `Thinking` while active. With multiple jobs it
shows progress such as `1/3 Thinking`; only `Thinking` shimmers and active copy is light green.
The status is available in Branch and conversation Insight Tree modes, except while an Insight is
hovered.

Tapping status opens the Model Tasks card. Completed tasks show a checkmark, the current task shows
a spinner and stop action, and upcoming tasks show a dotted circle and remove action. Upcoming
tasks can be reordered by drag with haptic feedback. Task/icon changes fade, and completed rows
clear shortly after the queue becomes idle. The empty popup remains 345px wide.

Structured generation receives one narrow repair attempt when required output is malformed. Repair
preserves valid substance and fixes only the contract surface; optional content is omitted rather
than invented. If the on-device model is unavailable or output remains invalid, live actions fail
explicitly instead of substituting preview/mock knowledge. Definitions and canvas actions offer
retry; failed Midpoint and Make Node work rolls back temporary Tree state.

## 5. Home reflection and discovery

Home shows one open-ended Question of the Day grounded in an unresolved idea, distinction,
assumption, tension, consequence, or application from prior conversation. Generation consults
one conversation chosen at random from the five most recently created conversations (excluding
Study Topic containers), plus at most four relevant saved Insights. Conversations without a
submitted question and a substantive answer are skipped within that five-conversation window;
older conversations are not used as a fallback. With fewer than five conversations, it chooses
from those available. The selected question stays fixed until the next eligible daily refresh.
It asks one thing, supports
reflection rather than recall, avoids yes/no or leading framing, and remains neutral.

Answering the question opens its conversation and removes the card from Home. The model engages
with the user’s reflection using conversation context and general knowledge when no relevant
Library passage exists; missing passages alone do not prevent discussion. Explicit source
verification and quotations still require evidence. Once the current
question has been answered or has expired into the next day, an eligible non-conversation page
schedules a new `Consolidate information` task after the model has been idle for 15 seconds. The
active status reads `Consolidating...`. Opening an active conversation does not cancel an already
queued consolidation task. Failed generation does not persist a placeholder question and remains
retryable.

Home also shows four optional, fail-quiet discovery cards, all selected on device. The
conversation-based cards are scoped to the active non-study conversation (or the most recently
created eligible one):

- **Today in History** presents the entry for today's date from a small, hand-curated catalog
  bundled with the app (not model-generated) and opens a conversation with its context when
  selected. Days without an entry show no card. It is dismissed by answering it and stays hidden
  for the rest of that day.
- **Loose Thread** surfaces a Node Concept with no strong MiniLM connection (≥ 0.70) to any other
  Node Concept in the conversation, preferring the one with more attached Insights. Selecting it
  opens that conversation's Insight Tree with the Node Concept selected. It needs at least two
  Node Concepts.
- **Terms You Glossed Over** resurfaces the oldest definition looked up in the conversation at
  least 24 hours ago and never saved as an Insight. Selecting it opens the definition card, where
  it can be saved.
- **Your Quote** surfaces one of the user's own messages that the local model judged to be an
  original synthesis. A cheap pre-filter (at least 40 characters, not a question, not filler)
  gates the model check, which runs in the background tree task. The same quote stays up for the
  day, and a quote waits 14 days before it can resurface.

Each card is absent when nothing qualifies; Home shows no error state for these supplementary
sections.

## 6. Context controls and slash commands

The context control is a non-spinning usage gauge without a visible `Context` label. Tapping it
opens the context card, which can clear the active conversation.

Eight slash commands are supported:

- `/define <text>` creates one complete Insight card inline in the response, with no surrounding
  prose. A single word is defined directly; for a noun phrase or sentence, the on-device model
  selects its noun or main subject and defines that. A bare `/define` remains editable until a
  subject is entered. The card uses the existing quote, bookmark, and speech actions; it is not
  automatically bookmarked.
- `/compact` asks the on-device model to combine the previous compacted checkpoint and turns
  since that checkpoint into a new hidden model context. The visible transcript remains unchanged.
- `/clear` cancels applicable model work and resets the active conversation in place, retaining
  the conversation identity/study-topic relationship.
- `/rename` opens a prompt to name the active conversation; `/rename <name>` sets it directly.
  A manually chosen name persists when the first question is submitted.
- `/new` saves the current conversation and starts a new one, without cancelling queued work.
- `/tree` opens the active conversation's Insight Tree.
- `/topic` opens the Study Topic picker for the active conversation.
- `/insights` opens the Insight library.

Commands other than `/rename` and `/define` take no argument; with trailing text they are
sent as an ordinary question.

Compaction state is persisted per branch as `compactedContext` plus
`compactedThroughBlockCount`. Future requests send that checkpoint and only later transcript
blocks.

## 7. Insight Tree

Canvas Mode is the current conversation's zoomable semantic map:

- **Insights** are manually saved contextual definitions.
- **Node Concepts** are broader subjects that organize Insights or pivotal ideas extracted from a
  response.

After a completed answer is persisted, iOS queues a background on-device analysis of that turn.
It may add zero or one Node Concept, decided by MiniLM similarity to the existing Node Concepts. It
never creates an automatic Insight and does not backfill old transcript history. The explicit Mac
Board-to-sidebar transfer maps the Board conversation's eligible completed turns. Returning to
an existing sidebar conversation preserves its tree and checks only new or changed turns, using
persisted fingerprints and successful-analysis checkpoints (see `INSIGHT-TREE.md`).

Insight-to-Node and Node-to-Node line lengths communicate semantic distance. Conversation
Node-to-Node connections have a 360px minimum so broader subjects remain visually separated.
The graph uses sparse persisted topology rather than all-pairs connections.

Users can inspect and hover cards, quote concepts into conversation, select multiple targets,
create Midpoints, and promote an Insight with Make Node.

Midpoint accepts two through eight selected Insights or Node Concepts. The selection’s weights are
derived from handle geometry, normalized by application code, and used to calculate a weighted
embedding centroid. Angrove supplies five substantive candidates; MiniLM selects the candidate
nearest that centroid. This is a mathematical vector-space operation, not a prompt asking for a
verbal “50/50 mixture.” Every candidate integrates the two dominant sources; larger selections use
an integrated weighted-center pass and require the remaining sources to contribute across the
pool. At eight selected items, add-to-selection remains hidden. The selection also retains a
visible cancel control. The completed Midpoint is a Node Concept with the selected Insights as
its members. New connectors first extend from the concept to the Insights. Their old connectors
then fade away, and the new lines pull the Insights to distances recomputed from their embeddings
against the new concept. Original
Node Concepts remain visible, and the new concept connects to each source Insight’s former parent
with one link per parent. The transfer settles with the same rubberband spring as releasing an
Insight after dragging it. The result creates no synthesized saved Insight; ownership and the
new Node Concept persist per tree scope. Selected Node Concepts retain concept-to-concept links.

Study's **Branch** tool is available only for an Insight. It isolates that Insight, accepts
2–6 children, and promotes the idea to a Node Concept with the same text and a connection to its
old parent. Angrove decomposes it into exactly that many distinct, more fundamental points as
Insights. Only the Node Concept text shimmers while loading. After the complete batch is ready,
Insights burst out 0.1–0.25 seconds apart, each with a haptic and opposing node recoil. Failure
rolls the promotion back. Legacy Make Node callers retain their three-child default.

## 8. Current persistence boundaries

- iOS conversation and branch state is a Codable JSON snapshot at
  `Application Support/Aquinas/ConversationStore/conversations-v1.json`, written atomically by
  `InquiryPersistenceStore`. It retains up to five rotating JSON backups and imports either legacy
  `UserDefaults` snapshot on first successful load. It is no longer stored in `UserDefaults`, but
  it is not yet normalized SwiftData storage.
- The global Insight Library, conversation-to-global-Insight membership, and on-device tree state
  (Node Concept seeds, positions, labels, Make Node children, placed Midpoints) are dedicated
  `UserDefaults` stores. Membership holds only identifiers; the library stores encoded
  contextual-definition records.
- Nothing is stored off the device. There is no server-side database.

Not yet done: the planned SwiftData migration, file-backed attachment storage, dedicated
cross-conversation Insights popup, and an on-device cache of conversation-scoped definitions. The current
snapshot abstraction keeps call sites insulated from the eventual storage replacement.

Conversation response disclosures show “Thought for <minutes>m <seconds>s” using recorded
preparation time from the active response job's start until its final answer is ready, including
context compaction, writing, and response checks. Waiting in the model queue and the answer's
reveal animation are excluded. Saved responses retain this
duration; durations below one minute omit the minutes, and older responses without timing
show “Thought”. The thinking/writing footer retains elapsed time only during loading, fades
out before the answer reveal, and omits estimated token counts and their separator.

The response preparation preview presents numbered Focus, Approach, Context (when included), and
Evidence notes. These are a public outline derived from the request, not a model reasoning trace.
Actual draft source-review activity is shown only when that review runs, with its outcome retained
in the completed disclosure. Retrieval alone does not advance the UI to Writing Response.
