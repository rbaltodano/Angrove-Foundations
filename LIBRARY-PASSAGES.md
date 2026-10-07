# Library passages: implemented contract

As of October 6, 2026, Library reader selections offer **Ask** alongside the native Copy menu.
Ask switches the shared Model Controls to New Conversation, Existing Conversation, and Cancel.
Selecting a destination attaches the exact selected text as a Library quote, with the work title
and a books icon. Opening the chip shows the quoted passage and a separate author/date line.
Approximate dates use `~`; unidentified authors or works use **Unknown**.

Library quote cards have Bookmark and Ask actions. Bookmark stores the excerpt in **Clipped
Passages** on the Library home page. Conversation Plus → **Passages** opens the same swipeable
picker used for Insights, starting on its Passages tab. Ask on a passage card in a conversation
attaches it to that conversation. Library passages do not offer Branch and are not contextual
definitions or automatically generated Insights.

The existing Codable `ConceptDefinition` carries two optional-compatible fields:
`isLibraryQuote` defaults to false when absent; `libraryAttribution` defaults to nil. Stable quote
identity, work title, selected text, and attribution survive conversation snapshots. Existing
Insights continue to decode with their existing meaning-deduplication behavior.

Clipped passages use the key `aquinas.library.clipped-passages.v1` through the same protected
atomic file store used for other personal collections. Existing UserDefaults clips migrate only
after a successful protected write. Encrypted envelopes, integrity checks, Complete File
Protection, and failure preservation remain active; damaged collections must never be replaced
with an empty collection. Clipping the same work/excerpt twice stores it once.

The Library corpus and model assets are ignored local resources, separate from Git-tracked
source. A physical-phone handoff must verify that the exact signed app contains the required
resources. A worktree display-name suffix does not give that build a separate data container:
builds sharing the production bundle ID replace each other when installed.
