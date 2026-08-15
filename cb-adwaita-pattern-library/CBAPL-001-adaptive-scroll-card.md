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
- The visible outer viewport/container owns the rounded shape and clipping.
- A scrolling child must not be responsible for an outer border radius that can move outside the visible viewport.
- Scrolled content remains clipped within the visible rounded container.
- The behavior adapts when the application window grows or shrinks.
- Horizontal scrolling should normally remain disabled for ordinary list/card content unless the content model explicitly requires it.

## Upstream foundation

Use native GTK4/libadwaita layout and scrolling primitives. Typical building blocks can include a `GtkScrolledWindow`, `GtkListBox`, `AdwClamp`, native card/boxed-list styling, and normal GTK size negotiation.

The exact widget composition is application-dependent. The behavioral contract is more important than one fixed implementation.

## CBAPL extension

GTK and libadwaita provide the primitives for natural sizing, scrolling, clipping, and card/list presentation. CBAPL adds an explicit cross-application contract for how those primitives should behave together when a rounded content surface must remain compact with little content and bounded with much content.

## Implementation guidance

Prefer native GTK4/libadwaita behavior over hard-coded heights or arbitrary CSS.

A correct implementation should be tested in at least these states:

1. one or two items;
2. enough items to grow the container but not require scrolling;
3. enough items to exceed the available space and require scrolling;
4. window resized smaller while the content is visible;
5. window resized larger again after scrolling was required.

The outer visible surface should retain its rounded bottom corners in every state.

## Evidence boundary

The pattern was formalized from repeated application UI work and from a Git Bench home-screen case where a Recent Projects list first contributed its complete natural height to the window and then, after becoming scrollable, exposed the separate issue of the list's rounded bottom corners existing outside the visible scroll viewport.

The behavioral contract is considered reusable. A single canonical Rust widget implementation has not yet been proven across the application suite, so the current status is **Reusable candidate** rather than suite-wide implementation standard.
