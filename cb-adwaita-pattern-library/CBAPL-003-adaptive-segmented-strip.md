# CBAPL-003 — Adaptive Segmented Strip

**Status:** Reusable candidate — runtime-proven in Git Bench  
**Base:** GTK4 / libadwaita  
**Scope:** Horizontal segmented filters, modes, workflow selectors, and compact tab-like controls

## Purpose

An **Adaptive Segmented Strip** is a rounded horizontal control container that follows the natural width of its segmented contents while those contents fit within the available space. Once the contents exceed the available width, the outer container stops growing and the inner control strip becomes horizontally scrollable.

The pattern is intended for compact groups such as workflow filters, category selectors, modes, and tab-like controls where wrapping would weaken scanability or break the relationship between adjacent segments.

It avoids three recurring failures:

1. a segmented control stretching to fill unnecessary horizontal space when only a few items are present;
2. individual first/last buttons owning rounded outer corners in addition to an already rounded container; and
3. a horizontal scrollbar being drawn over the controls or their labels.

## Behavioral contract

The intended width behavior is:

```text
container width = min(natural content width, available width)
```

Invariants:

- When all segments fit, the container follows the natural width of its contents.
- The container may grow as more segments are added while horizontal space remains available.
- Once natural content width exceeds the available width, the outer container stops growing and only the inner strip becomes horizontally scrollable.
- The component must not expand to full width merely because scrolling is supported when the natural content is narrower.
- The visible outer container owns the rounded shape, background treatment, boundary, and clipping.
- Individual segment buttons do not own the outer border radius.
- Segments remain visually contiguous and must not wrap to multiple lines.
- Selected, hover, focus, and active states remain native interaction states inside the shared outer shape.
- A horizontal scrollbar is shown only when actual overflow exists.
- A scrollbar must never overlap, cover, or reduce the readable/control area of the segment buttons.
- The pattern adapts when the available width changes.

## Shape ownership

CBAPL-003 uses the same single-shape-owner principle as CBAPL-001:

```text
outer segmented container
├── owns surface/background treatment
├── owns border radius
├── owns boundary
├── owns clipping
└── contains
    ├── horizontal scrolling viewport
    │   └── flat contiguous segment buttons
    └── horizontal scrollbar when overflow exists
```

The outer container is the persistent visible shape. Segment buttons may draw selection or interaction backgrounds, but they should not recreate first/last outer corner radii.

When a selected segment touches the left or right edge, the outer container's clipping naturally constrains that selection surface to the shared rounded boundary.

## Scrollbar contract

Scrollbar placement is a first-class part of CBAPL-003.

The horizontal scrollbar must occupy its own layout area **below the segment controls** when overflow exists. It must not be an overlay drawn over button labels or consume the vertical allocation required by the buttons themselves.

The runtime-proven Git Bench composition uses:

- a `GtkScrolledWindow` for the horizontal control viewport;
- horizontal scrolling only;
- `propagate-natural-width` so the component can follow the natural width of the segments while they fit;
- `propagate-natural-height` so the segment row retains its complete natural height;
- `GtkScrolledWindow` horizontal policy `External`; and
- a separate horizontal `GtkScrollbar` connected to the scrolled window's horizontal adjustment.

Conceptually:

```text
┌──────────────────────────────────────────────┐
│ All │ Getting Started │ Changes │ Commits…  │
├──────────────────────────────────────────────┤
│ ───────────── horizontal scrollbar ────────  │  only on overflow
└──────────────────────────────────────────────┘
```

Using only `overlay-scrolling = false` is **not sufficient evidence of a correct implementation**. In Git Bench, that first implementation still allowed the scrollbar to consume the same vertical allocation as the segment row, visually colliding with the controls. Moving the scrollbar to a separate layout child resolved the issue at the structural level.

Alternative widget compositions are allowed only when they preserve the same invariant: the scrollbar is physically separated from the control area and cannot overlap the segments.

## Segment presentation

Prefer a visually flat, contiguous inner control strip:

- no persistent outer border radius on individual buttons;
- no gaps that make the group read as unrelated standalone buttons unless the product intentionally requires separate actions;
- separators or native button boundaries may distinguish adjacent segments;
- one selected segment may use the normal selected/toggled background state;
- the outer surface remains responsible for the component's overall silhouette.

The pattern does not prescribe a specific semantic widget. `GtkToggleButton` groups are suitable for mutually exclusive filters or modes when their semantics match the use case.

## Upstream foundation

Use native GTK4/libadwaita layout, button, scrolling, adjustment, clipping, focus, and state mechanisms.

Typical building blocks can include:

- `GtkFrame` or another suitable outer shape owner;
- `GtkBox` for the horizontal segment row;
- grouped `GtkToggleButton` controls;
- `GtkScrolledWindow`;
- `GtkScrollbar` connected to the horizontal adjustment; and
- native style classes and normal GTK size negotiation.

The exact Rust/widget composition is secondary to the behavioral and shape-ownership contracts.

## CBAPL extension

GTK provides the primitives for horizontal scrolling, adjustments, natural size negotiation, grouping, clipping, and scrollbars. CBAPL adds an explicit application-level contract for how those primitives should be combined when segmented controls must remain compact at wide sizes, bounded at narrow sizes, and visually coherent inside one rounded container.

CBAPL-003 also makes scrollbar placement explicit because a technically scrollable implementation can still be visually invalid when the scrollbar overlaps or compresses the controls.

## Implementation guidance

A correct implementation should be tested in at least these states:

1. few segments where the natural width is substantially smaller than the available width;
2. enough segments to approach the available width without scrolling;
3. overflow at a narrower application width;
4. resizing from wide to narrow while the strip is visible;
5. resizing back from narrow to wide after horizontal scrolling occurred;
6. first, middle, and last segment selected;
7. keyboard focus navigation across the group;
8. scrollbar visibility appearing only when overflow exists;
9. button labels remaining fully unobstructed when the scrollbar is present.

Do not solve scrollbar overlap with arbitrary bottom padding around an overlay scrollbar. Prefer a structural layout where the scrollbar has its own allocation.

## Evidence boundary

CBAPL-003 was formalized from the Git Bench **Git Guide** category strip.

The initial implementation used a horizontally scrolling segmented ToggleButton group inside a rounded container. Runtime testing proved the desired adaptive-width behavior, but also exposed a failure: even after disabling overlay scrolling, the scrollbar shared the control area's vertical allocation and visually crossed the segment buttons.

A second implementation moved the horizontal scrollbar to a separate child below the controls while sharing the same horizontal adjustment. Runtime testing then confirmed both required states:

- at wide widths the complete segmented strip fits naturally with no scrollbar; and
- at narrow widths the outer container is bounded, the segment row scrolls horizontally, and the scrollbar remains physically below the controls without overlapping them.

The behavioral contract and the Git Bench composition are therefore runtime-proven in one application. A single canonical Rust widget implementation has not yet been proven across the application suite, so the current status remains **Reusable candidate** rather than suite-wide implementation standard.
