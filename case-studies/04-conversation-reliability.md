# Keeping answers safe when the user moves on

**Angrove separates the lifetime of a model request from the lifetime of the screen that submitted it.**

Scope: SwiftUI state ownership, asynchronous generation, task scheduling, file persistence, and regression tests. Status: implemented behavior and test coverage; no new test run performed for this writeup.

## The problem

Local inference can take long enough that a person leaves the conversation before the answer arrives. They may open another thread, change branches, edit a new draft, or clear the original question.

A result delivered through the old screen's bindings can disappear, land in a different conversation, overwrite newer text, or recreate content the user deleted. These failures become especially difficult to reproduce when they depend on navigation, rendering, and background work completing in a particular order.

The product requirement was straightforward: leaving a screen should not lose a question, and deleting a question should not be undone by a late answer.

## Persisting the destination before starting work

Submitting a question first persists its response placeholder. The queued job captures the original conversation and branch identifiers, giving completion a destination independent of the currently displayed screen.

If the original column remains visible, the presentation can reconcile normally. If it has been unmounted, completion writes to the existing persisted slot by its original identifiers. It does not infer the destination from the new conversation's message count or an old array position.

Branch bindings also resolve by identity. A delayed reveal or editor callback from a removed page cannot replace a different branch that happens to occupy the same array index.

## Handling stale state and explicit deletion

A page may retain an old snapshot with an empty response while a detached task has already saved the answer. Saving that page must preserve the completed response alongside the person's newer composer draft.

The reconciliation policy distinguishes that case from a different question occupying the same position. It also refuses to recreate a missing response slot after clearing or deletion. This makes completion conditional on the destination still representing the original work.

Automatic conversation naming follows a similar rule. Before persisting a generated title, it rechecks the conversation, branch, initial question, and title eligibility. A delayed naming result must not overwrite a manual rename or replay a handoff after navigation.

## Durable storage and shared work ownership

Conversation branches and message blocks use an atomic Codable file snapshot in Application Support, with up to five rotating backups and migration from the earlier UserDefaults representation. A serial background queue orders writes and reads; the app flushes queued work when the scene moves to the background.

One process-scoped model queue coordinates questions, definitions, and background tree work. Cached-content presentation and queue status are separate concerns: the interface must not imply that every action requires another generation.

This architecture keeps durable data outside view-local state without claiming a completed SwiftData migration. Saved Insights and some coordination stores still use UserDefaults.

## What the tests protect

The regression suite includes three queued questions completing in their original conversations after navigation, stale saves preserving completed answers and newer drafts, and late completion refusing to resurrect cleared content. It also covers ordered writes, stable identifier round-trips, migration, import validation, and recovery from the newest valid backup when the live snapshot is corrupt.

These tests describe meaningful user-visible failures rather than checking only that a storage method returns successfully.

## Outcome and boundaries

Generation can finish after the initiating page disappears, while persistence remains authoritative about where the result belongs. Navigation is no longer treated as implicit cancellation, and explicit deletion remains authoritative over late work.

The evidence establishes implementation and focused coverage. It does not prove every interruption is safe, nor that pending generation survives process termination: the on-device queue is not a persisted job system. That distinction matters for any future promise of background or restart recovery.

## Evidence

- [Ownership and completion architecture](https://github.com/rbaltodano/Aquinas-iOS/blob/2513a31ea7dceee65f6384725320d867e8a6b718/Documentation/App-Architecture.md)
- [Persistence and navigation regression tests](https://github.com/rbaltodano/Aquinas-iOS/blob/2513a31ea7dceee65f6384725320d867e8a6b718/Angrove-iOSTests/InquiryPersistenceStoreTests.swift)
- [Shared model queue](https://github.com/rbaltodano/Aquinas-iOS/blob/2513a31ea7dceee65f6384725320d867e8a6b718/Angrove-iOS/Features/Conversation/ModelTaskQueue.swift)
- [Planned persistence migration](https://github.com/rbaltodano/Aquinas-Foundations/blob/c083606e9c068086455ddceaa41b82d21235e28e/PERSISTENT_MEMORY_IMPLEMENTATION_PLAN.md)
