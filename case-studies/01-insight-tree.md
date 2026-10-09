# Turning conversations into an explorable map

**Angrove connects a reading-oriented AI conversation to a local semantic map, while leaving the decision to save an idea with the user.**

Scope: product design, SwiftUI interaction, local embeddings, graph layout, and persistence. Status: implemented tree and Study interactions, with additional Study tools still planned.

## The problem

A conversation can produce useful explanations without producing a useful collection of knowledge. Important definitions get buried in the transcript, and returning to an earlier idea means finding the right message again.

Angrove's product hypothesis is that a person studying philosophy or theology should be able to move between an explanation, a contextual definition, and the relationships among saved ideas. The challenge was to make that transition useful without automatically filling a graph with every term the model mentions.

## Two different kinds of knowledge

The interface distinguishes a **Node Concept**, a broader subject organizing a conversation, from an **Insight**, a contextual definition the user chooses to keep.

After an eligible response, a background task asks the language model for the turn's broader subject. Application code then compares its embedding with existing subjects. The documented new-subject policy uses cosine similarity of 0.60: sufficiently related subjects stay within the existing concept; distinct subjects become new seeds.

Highlighted terms follow a separate path. Tapping one opens a contextual definition; saving it adds an Insight to the user's collection. Merely generating or highlighting a term does not bookmark it.

This distinction controls graph growth and gives the map two meanings: what the conversation is about, and what the person deliberately kept from it.

## Dividing responsibility between AI and application code

The language model proposes concepts and explains them. Bundled MiniLM produces normalized, 384-dimensional embeddings. Application code calculates similarity, handles membership and identifiers, and arranges the visual structure.

That separation avoids asking the language model to invent relationship scores. It also makes individual responsibilities testable: a label can be rejected for repeating its member's title without changing the member's identity or position.

The map balances semantic structure with spatial stability. Existing concepts act as anchors rather than being completely rearranged after each addition. Collision handling keeps labels apart. Visual distances express approximate relatedness, subject to readability constraints; they are not a precise measurement of conceptual truth.

## Moving into Study without losing place

Study reuses the existing tree and changes its camera instead of opening a disconnected copy of the selected concept. The camera moves from overhead to eye level, surrounding Insights spread over a sphere, and the rest of the tree fades. A draggable ring rotates the cluster; tapping an Insight brings it into focus.

On exit, the camera returns to the tree and preserves the selected item. The interaction is designed to retain context through a change in view. Camera tests cover projection equivalence, keeping the target centered, and changing the rotation center without shifting the visible scene.

![Angrove Study view](https://raw.githubusercontent.com/rbaltodano/Aquinas-iOS/2513a31ea7dceee65f6384725320d867e8a6b718/Documentation/Screenshots/study-3d.jpg)

## Outcome and boundaries

The implemented experience links conversation, saved definitions, semantic grouping, and 3D inspection on-device. Conversation, global-library, and Study Topic trees have distinct membership boundaries. Global and topic updates use explicit acceptance rather than silently replacing the user's accepted collection.

The evidence establishes working interactions and regression coverage, not improved learning outcomes. External usability testing is still needed to determine whether people understand the distinction between Nodes and Insights and whether the map helps them return to ideas. The Study tool switcher includes planned capabilities that are not yet wired to the 3D view.

The central design decision was to make automatic organization assist deliberate collection. An AI-generated explanation becomes part of the user's knowledge library only through a clear interaction.

## Evidence

- [Insight Tree specification](https://github.com/rbaltodano/Aquinas-Foundations/blob/c083606e9c068086455ddceaa41b82d21235e28e/INSIGHT-TREE.md)
- [Study interaction documentation](https://github.com/rbaltodano/Aquinas-iOS/blob/2513a31ea7dceee65f6384725320d867e8a6b718/Documentation/Study-Tool.md)
- [App ownership and persistence](https://github.com/rbaltodano/Aquinas-iOS/blob/2513a31ea7dceee65f6384725320d867e8a6b718/Documentation/App-Architecture.md)
- [Camera regression tests](https://github.com/rbaltodano/Aquinas-iOS/blob/2513a31ea7dceee65f6384725320d867e8a6b718/Angrove-iOSTests/OrbitCameraTests.swift)
- [Canvas persistence tests](https://github.com/rbaltodano/Aquinas-iOS/blob/2513a31ea7dceee65f6384725320d867e8a6b718/Angrove-iOSTests/InsightTreeCanvasPersistenceTests.swift)
