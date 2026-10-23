# Angrove Design System: "The Scriptorium"

## Design Principles
- **Aesthetic: "Modern Antiquarian" / "Digital Scriptorium"** The app should feel like a tactile, high-quality theological manuscript brought into the digital age. It uses "Academic Minimalism"—low-contrast, warm earthy tones, and typography-driven hierarchies instead of modern, bubbly iOS paradigms.
- **Layout Architecture: The Infinite Canvas**
  Unlike traditional vertical chat feeds, the UI is a cartographic, node-based tree. Content is organized into discrete cards connected by thin, 1px horizontal and vertical structural lines, mimicking a scholar organizing index cards on a desk.
- **Component Restraint:**
  Avoid floating elements. Cards should be flat against the background, defined by thin borders and gentle corner radii rather than heavy drop shadows. The interface must step back and let the textual logic shine.

## Colors

`AngroveTheme.Colors` in `DesignSystem/SharedTypography.swift` is the implementation source of
truth. Important light/dark pairs:

- **Canvas:** `#F3EEE2` / `#120F0C`
- **Canvas Secondary:** `#FFFAF0` / `#24201C`
- **Primary Brown:** `#4A321C` / `#FFFAF0`
- **Heading Text:** `#614C40` / `#FFFAF0`
- **Paragraph Text:** primary brown at 75% opacity / warm cream at 75%
- **Placeholder Text:** primary brown at 50% / warm cream at 50%
- **Light Green:** `#86803E` / `#B7AE78`
- **Accent Red:** `#AF4949` in both modes
- **Borders:** dark brown at low opacity in light mode; warm cream at low opacity in dark mode

## Typography
We use a dual-font system. Libre Baskerville (Serif) is for academic/structural hierarchy, and Figtree (Sans-Serif) is for readable UI data and body copy.

- **Primary Font (Serif):** Libre Baskerville
  - **Title:** 24pt Regular; larger title tokens are 34pt, 36pt, and 40pt
  - **Heading:** 20pt Regular
  - **Quote:** 16pt Italic

- **Paragraph text follows the user's settings.** Body and body-large text use the user's
  *Response font* (Sans/Serif) and *Font size* (Small 12 / Medium 14 / Large 16) through the
  `.paragraphFont(.regular | .large)` modifier, not fixed fonts. Sans + Medium equals the
  tokens below (14 / 16); Small and Large shift both by -2 / +2. Do not use
  `.font(AngroveTheme.Typography.body)` for paragraph copy. Text *alignment* settings apply only
  to conversation text, never to app-wide paragraphs. Headings, labels, chips and UI copy stay
  fixed. Settings prose is an exception to the response-font preference: it always uses
  Figtree Regular, retaining the selected text size and shared paragraph line spacing.
- **SF Symbol icons** are sized with `.font(.system(size:weight:))` (typically 12-16pt,
  semibold/bold); this is the only accepted use of `.system` fonts.

- **Secondary Font (Sans-Serif):** Figtree
  - **UI Heading:** 18pt Bold
  - **UI Subheading / Body:** 14pt Bold / 14pt Regular
  - **Body Large:** 16pt Regular
  - **Inline Insight:** 16pt Bold in light green
  - **Labels / Chips:** 12pt Bold

## Spacing & Layout
Units are SwiftUI points. Observed usage in the app is noted so new work matches it.

- **Base Unit:** 8pt. Common spacing steps: 4, 8, 12, 16, 20, 24, 32, 48.
- **Screen / content padding:** 24pt is the dominant page and card padding (about 3x more common
  than 16pt). `AngroveTheme.Spacing.screenPadding` is 16 and is used for tighter chrome.
- **Corner radius scale** (always `style: .continuous`):
  - **8-12:** small chips, rows, thumbnails, side-menu rows, canvas node hit areas
  - **16:** small cards (`Spacing.smallCardRadius`)
  - **28:** the standard card radius (`Spacing.cardRadius`): cards, sheets, popups, menus
  - **36:** docked cards above the control dock (Insight cards, question bar, Model Tasks)
  - **Capsule:** pills, chips, icon buttons, the Model Controls pill
  - **Nesting rule:** an outer container's radius = the inner element's radius + the gap between
    them (e.g. a 10pt card inset inside a frame gives 28 + 10 = 38).
- **Borders:** 1pt hairline is the default (`AngroveTheme.Colors.divider` / `border`). 1.5pt for
  selected or emphasized outlines. Thicker strokes are reserved for canvas graphics.
- **Docked cards:** sit a fixed gap above the Model Controls (`modelControlsReservedHeight`) and
  inset horizontally; embedded demos use a 10pt bottom inset to match.
- **Minimum touch target:** 44pt (`Spacing.controlHeight`, `iconButtonSize`).

## Components
- **Buttons:** Style: Outline or flat fill. No gradients.
  - Shape: Capsule. Rounded rectangles are for cards, not buttons.
  - Colors: Use `#AF4949` (Accent) for primary actions, `#4A321C` (Primary) for secondary.
  - **Changing label or contents:** Whenever a button's text or contents change (a back button
    retitling, the Model Controls pill swapping its buttons, a status label changing), every
    button updates the same way, all at once:
    - the button pulses, scaling up 5% and springing back to 100%;
    - the old text blurs and fades out while the new text blurs and fades in (never an
      instant swap or a plain crossfade);
    - the button's width springs smoothly to fit the new contents.
    Use the shared `blurFade` transition and a spring for width. Buttons beside a fixed neighbor
    pulse from their anchored edge so they don't overlap it.
- **Cards:** Use semantic canvas/component-background tokens, a quiet 1px semantic border, and
  restrained elevation. Elevation is the shared `.cardGlow()` modifier (radius 24, 15% warm
  brown from `AngroveTheme.Colors.cardGlow`, optional y offset), used only for cards that float
  over the canvas (docked Insight cards, sheets); resting cards have no shadow.
- **Embedded demos (User Guide):** mini Insight Trees use the real `InsightTreeCanvasView`, the
  canvas color, a 1pt `divider` border, a 38pt continuous radius, and a 12pt side gap from the
  screen edge. Tapping an Insight docks the real card inside the frame.
- **Haptics:** light impact for taps and selection changes (the default), medium for committing
  actions, selection feedback for pickers, rigid only for hard stops.
- **Motion:** use the named springs in `DesignSystem/Motion.swift`, never raw `.spring(...)`:
  `springStandard` (0.42/0.86, cards and layout), `springQuick` (0.32/0.86, exits and small
  controls), `springMicro` (0.2/0.86, press feedback), `springLively` (0.36/0.78, selection),
  `springRelaxed` (0.5/0.8, large movement), `springBouncy` (0.4/0.68, playful confirmation).
  Ease-in-out 0.2-0.35s for fades and color changes.
- **Conversation headings:** every displayed name change uses the shared opacity-and-blur
  replacement, including automatic naming and manual renames. The heading container springs
  to the incoming title's measured height, moving the question and transcript smoothly when
  wrapping changes. Outgoing text does not reserve extra layout space during its fade.
- **Inputs (Text Fields / Chat Box):** Style: Minimalist, with the semantic side-menu/input border.
  - Padding: 24px internal padding.
  - Font: Figtree, 16pt.
- **Model Status:** Replaces the old Thinking toggle. Inactive copy is `Idle`; active copy is
  `Thinking` in light green with shimmer. When multiple jobs exist, prefix it with progress such
  as `1/3`. Only `Thinking` shimmers. The control is present in Branch and conversation Insight
  Tree modes, but disappears while an Insight is hovered so the contextual actions have priority.
- **Model Tasks popup:** A 345px-wide card with 28px internal padding, 28pt corner radius, an 8px
  outer vertical gap, and a 4px task-list gap. The heading and completed checkmarks use heading
  text color. Current tasks use the context wheel spinner and a `stop.circle.fill`; upcoming tasks
  use a dotted circle and removable `xmark`. Task-state icons crossfade. The idle card stays full
  width rather than hugging its copy. Development-model availability headings and explanations
  are omitted everywhere, including this popup and model Settings. The task list or idle message
  follows the popup heading directly in development builds.
- **Popup motion:** Model Tasks and Insight hover cards enter with a spring-driven scale-up plus
  upward translation from below and leave with the inverse transition. Switching hovered Insights
  replaces the whole card so outgoing and incoming content animate independently. Opening Model
  Tasks also produces a light haptic tap.
- **Insight card dismissal:** On iPhone and Mac, docked Insight and Node cards end their four-action
  row with an X after read aloud. X closes the card and clears its tree hover/focus without deleting
  the Insight. Tree panning and background taps keep the card open; removal remains in the card's
  context menu. A completed conversation definition first posts its existing notification; opening
  that notification replaces it with the Insight card above Model Controls. This card uses no sheet
  or modal and stays open through scrolling, quoting, and other control interactions until X is pressed.
- **Queued state:** Queued definitions and the `Question queued` response state use the shared
  breathing animation. Cancelling the associated queued/current task must stop the animation.
- **Thinking response state:** The initial `Thinking` placeholder is bold and centered. Once the
  model supplies its user-facing approach summary, that text uses response line height, leading
  alignment, regular weight, and the same shimmer treatment. On completion, the visible live
  progress fades out before collapsing into the Thought duration disclosure. The full saved
  thinking text stays hidden until that disclosure is opened; it never replaces the outgoing
  live summary during collapse.
- **Context gauge:** The control dock retains the wheel as a static usage gauge. It has no visible
  `Context` label and does not spin while the model is loading; tapping it opens the context card.
- **Insight Tree update affordance:** When the tree control reads `Updated`, it follows the
  shared button-change rule above (5% pulse with blurred text exit and entry).

## Guardrails (Do's and Don'ts)
- NEVER use standard Apple iOS colors like `.blue` or `.red`. Use `AngroveTheme.Colors`; do not
  repeat raw hex values in feature code.
- DO NOT use generic `Text()` views without applying the semantic fonts from `SharedTypography.swift`.
- Keep layouts structured and hierarchical; this is a tool for logic, not a playful social media app.
- Keep model activity and context capacity visually distinct: shimmer communicates active model
  work; the context wheel communicates usage only.
- Feature code takes every color from `AngroveTheme.Colors` (including `onAccent`, `scrim`,
  `pulse`, `hairline`, `deepSurface`); no raw hex, `Color(red:)`, `.white`, or `.black`. Use the
  `placeholder(for:)` / `dotGrid(for:)` helpers where a scheme-resolved color is required in
  UIViewRepresentable-hosted views.

## Study Branch

Only a studied Insight offers Branch. Activation fades every other concept away. Confirmation
crossfades the Insight into a Node Concept with the same title; apply the existing loading text
shimmer to the title alone, keeping the icon steady. Hold all children until the complete batch
is ready, then release them 0.1–0.25 seconds apart with an accelerating/decelerating outward
motion, one haptic tap per child, and a small opposite recoil from the Node. Reduce Motion uses
fades without recoil. Burst targets use the tree's actual semantic bond lengths, and viewport
fitting scales the connected cluster uniformly. Once the burst finishes, Study focuses the new
Node in the normal tree, with child connectors and the preserved parent connection visible.
Insight radii remain relatedness-based in Study; they do not converge on an equidistant sphere.
The full tree returns on Study exit.
