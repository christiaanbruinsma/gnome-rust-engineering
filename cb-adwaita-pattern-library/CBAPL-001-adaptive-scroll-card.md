# CBAPL-001 — Adaptive Scroll Card

**Status:** Reusable candidate  
**Base:** GTK4 / libadwaita  
**Scope:** Bounded, scrollable list and card-style content

## Purpose

An **Adaptive Scroll Card** is a rounded content container that grows naturally with its contents while those contents fit within the available space. Once the contents exceed the available space, the container stops growing and only its contents become vertically scrollable.

The pattern avoids two common failures:

1. a list forcing its parent window to become unnecessarily tall; and
2. rounded list/card corners disappearing because the rounded child itself extends beyond a scroll viewport.

## Behavioral contract

The intended height behavior is:

```text
container height = min(natural content height, available height)
```

Invariants:

- Little content: the container height follows its content.
- More content: the container grows naturally while space remains available.
- Too much content: the container is capped by the available height and only then becomes vertically scrollable.
- The pattern must not introduce unnecessary empty vertical space simply because scrolling is supported.
- The visible outer viewport/container owns the rounded shape, background, and clipping.
- A scrolling child must not be responsible for an outer border radius that can move outside the visible viewport.
- Scrolled content remains clipped within the visible rounded container.
- The behavior adapts when the application window grows or shrinks.
- Horizontal scrolling should normally remain disabled for ordinary list/card content unless the content model explicitly requires it.

## Upstream foundation

Use native GTK4/libadwaita layout and scrolling primitives. Typical building blocks can include a `GtkScrolledWindow`, `GtkListBox`, `GtkFrame`, `AdwClamp`, native card styling, separators, and normal GTK size negotiation.

The exact widget composition is application-dependent. The behavioral contract is more important than one fixed implementation.

## CBAPL extension

GTK and libadwaita provide the primitives for natural sizing, scrolling, clipping, and card/list presentation. CBAPL adds an explicit cross-application contract for how those primitives should behave together when a rounded content surface must remain compact with little content and bounded with much content.

The pattern also defines a preferred ownership boundary for scrollable lists:

```text
outer card / viewport
├── owns background
├── owns border radius
├── owns clipping
└── contains
    └── flat scrolling list
        ├── separators provide structure
        └── native hover / active states provide row feedback
```

This avoids a visually nested card-inside-card result and prevents first/last rows from drawing a second outer radius inside an already rounded container.

## Preferred list presentation

When the Adaptive Scroll Card contains a row-based list, prefer a structurally flat inner list when native interaction states remain clear:

- the outer card is the only persistent rounded surface;
- inner rows do not own permanent outer corner radii;
- inner rows may remain transparent at rest;
- native separators distinguish adjacent rows;
- native hover, active, focus, or selection states remain responsible for transient interaction feedback;
- do not retain a second `boxed-list`-style outer frame merely to obtain row backgrounds if the outer card already provides the visible surface.

This presentation was runtime-verified in Git Bench's **Recent Projects** Adaptive Scroll Card: removing the inner boxed-list presentation removed the duplicate first/last-row radii and persistent row background, while native row hover feedback and separators remained clear and visually coherent.

This is the preferred CBAPL-001 presentation for comparable scrollable lists. Applications may use a different inner presentation when their content or selection model requires it, provided the outer shape ownership and adaptive sizing contract remain intact.

## Implementation guidance

Prefer native GTK4/libadwaita behavior over hard-coded heights or arbitrary CSS.

A correct implementation should be tested in at least these states:

1. one or two items;
2. enough items to grow the container but not require scrolling;
3. enough items to exceed the available space and require scrolling;
4. window resized smaller while the content is visible;
5. window resized larger again after scrolling was required;
6. pointer/focus interaction on first, middle, and last rows when a row-based list is used.

The outer visible surface should retain its rounded bottom corners in every state. Row interaction feedback should remain clear without introducing a second persistent card shape.

## Evidence boundary

The pattern was formalized from repeated application UI work and from a Git Bench home-screen case where a Recent Projects list first contributed its complete natural height to the window and then, after becoming scrollable, exposed the separate issue of the list's rounded bottom corners existing outside the visible scroll viewport.

Git Bench subsequently provided runtime evidence for the preferred flat-inner-list presentation: the outer card retained the visible rounded surface and clipping, the inner list used separators without its own boxed outer frame, and native hover feedback remained clear.

The behavioral and presentation contracts are considered reusable. A single canonical Rust widget implementation has not yet been proven across the application suite, so the current status remains **Reusable candidate** rather than suite-wide implementation standard.
