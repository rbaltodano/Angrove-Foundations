# Conversation tree lifecycle — October 7, 2026

The current app builds every tree on-device. After persisting a newly completed answer, it
captures the originating conversation ID, question and answer directly in a background job on
the shared Model Task queue. Leaving the page does not discard that job. Another turn's mapping
job cannot suppress it; foreground questions retain priority. Corpus-scope abstentions and
general-knowledge responses remain excluded from automatic subject extraction.

Returning through a completion notification restores the durable answer. A mounted conversation
tree observes changes to its persisted Node seeds, including work finishing in an older page
instance. Tree preparation waits for asynchronous graph application before reporting completion,
even when its embeddings already match the provider.

Bookmarked Insight content and identity take precedence over an older copy embedded in response
markup. The saved Insight stays a visible chip even when a cached Node's label is identical.
Label equality never promotes or hides a saved Insight. Explicit Branch/Make Node promotion
continues to replace its source through the existing topology rules. Automatic broader labels,
semantic thresholds, encryption and atomic persistence remain unchanged.

The navigation and saved-chip fix is covered by 108 focused tests across seven suites, including
multiple detached mapping jobs, old-versus-saved definition identity, asynchronous graph refresh,
membership, persistence, and Branch behavior. Actual phone navigation has not been visually
verified because UI hierarchy access is unavailable.
