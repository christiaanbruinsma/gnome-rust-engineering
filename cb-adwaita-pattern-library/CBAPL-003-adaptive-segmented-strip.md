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
3. a horizontal scrollbar obscuring controls or using a visually unrelated drag-handle style.

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
- A scrollbar must never obscure labels or interfere with control operation.
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
    └── horizontal scrolling viewport
        └── flat contiguous segment buttons
```

The outer container is the persistent visible shape. Segment buttons may draw selection or interaction backgrounds, but they should not recreate first/last outer corner radii.

When a selected segment touches the left or right edge, the outer container's clipping naturally constrains that selection surface to the shared rounded boundary.

## Scrollbar presentation variants

CBAPL-003 supports two scrollbar placement variants. Placement may differ, but scrollbar identity must not.

The actual choice between them follows the shared [CBAPL Scrollbar Placement Contract](SCROLLBAR-PLACEMENT-CONTRACT.md). For CBAPL-003, **Reserved / Integrated is the normal default** because the contents are interactive horizontal controls and hidden overflow is task-relevant. Overlay remains valid only when it can be proven not to interfere with labels, focus, selection, or pointer/touch targets.

### Overlay Scrollbar

Use **Overlay Scrollbar** when the horizontal scrollbar can sit over the lower edge of the control viewport without obscuring labels, focus indication, or other important interaction areas.

- The scrollbar remains part of the scrolling viewport.
- It may use native overlay visibility behavior.
- The strip must reserve enough usable control area that the scrollbar does not visually collide with button contents.
- The visible drag handle uses the shared CBAPL scrollbar identity defined below.

### Reserved / Integrated Scrollbar

Use **Reserved / Integrated Scrollbar** when the scrollbar should occupy its own layout area inside the same outer rounded component.

- For horizontal scrolling, the reserved area normally sits below the segment row.
- The scrollbar remains visually integrated with the component rather than appearing detached from it.
- It appears only when actual overflow exists unless the application has a specific reason to keep it persistent.
- The visible drag handle uses the same shared CBAPL scrollbar identity as the overlay variant.

The runtime-proven Git Bench composition uses:

- a `GtkScrolledWindow` for the horizontal control viewport;
- horizontal scrolling only;
- `propagate-natural-width` so the component can follow the natural width of the segments while they fit;
- `propagate-natural-height` so the segment row retains its complete natural height;
- `GtkScrolledWindow` horizontal policy `External`; and
- a separate horizontal `GtkScrollbar` connected to the scrolled window's horizontal adjustment.

Conceptually:

```text
Overlay
┌──────────────────────────────────────────────┐
│ All │ Getting Started │ Changes │ Commits…  │
│                ━━━━━━━━━━━━━                 │  scrollbar overlays lower edge
└──────────────────────────────────────────────┘

Reserved / Integrated
┌──────────────────────────────────────────────┐
│ All │ Getting Started │ Changes │ Commits…  │
├──────────────────────────────────────────────┤
│                ━━━━━━━━━━━━━                 │  own internal row
└──────────────────────────────────────────────┘
```

Use the shared placement contract rather than local visual preference.

Using only `overlay-scrolling = false` is **not sufficient evidence of a correct reserved implementation**. In Git Bench, that first implementation still allowed the scrollbar to consume the same vertical allocation as the segment row, visually colliding with the controls. Moving the scrollbar to a separate layout child resolved the issue at the structural level.

## Shared scrollbar identity

Scrollbar placement is allowed to vary. The **drag handle must remain visually consistent** across CBAPL components and placement variants.

Shared invariants:

- equal cross-axis thumb thickness for horizontal and vertical scrollbars;
- equal thumb corner radius / pill geometry;
- consistent resting contrast and opacity;
- consistent hover emphasis;
- consistent dragging emphasis;
- a subtle trough treatment that does not visually overpower the controls;
- orientation changes the direction and length of the thumb, not its visual identity.

In short:

> **Placement may vary. Scrollbar identity must not.**

Git Bench runtime testing established a shared application scrollbar geometry in which the reserved horizontal scrollbar in the Git Guide and the vertical overlay scrollbar in its topic list use the same visible thumb thickness and rounded handle treatment while preserving their different placement behavior.

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
- `GtkScrollbar` connected to the horizontal adjustment for reserved placement; and
- native style classes and normal GTK size negotiation.

The exact Rust/widget composition is secondary to the behavioral, shape-ownership, and scrollbar-identity contracts.

## CBAPL extension

GTK provides the primitives for horizontal scrolling, adjustments, natural size negotiation, grouping, clipping, and scrollbars. CBAPL adds an explicit application-level contract for how those primitives should be combined when segmented controls must remain compact at wide sizes, bounded at narrow sizes, and visually coherent inside one rounded container.

CBAPL-003 also makes scrollbar placement and visual identity explicit because a technically scrollable implementation can still be visually invalid when the scrollbar overlaps controls or uses a different visual language from other scrollable components.

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
9. button labels remaining fully unobstructed when the scrollbar is present;
10. shared thumb geometry verified against vertical CBAPL scrollable components;
11. both Overlay and Reserved / Integrated placement where an application uses both.

Do not solve scrollbar overlap with arbitrary padding around a conflicting scrollbar. Prefer a composition where the chosen placement has a clear, stable allocation and preserves the control area.

## Evidence boundary

CBAPL-003 was formalized from the Git Bench **Git Guide** category strip.

The initial implementation used a horizontally scrolling segmented ToggleButton group inside a rounded container. Runtime testing proved the desired adaptive-width behavior, but also exposed a failure: even after disabling overlay scrolling, the scrollbar shared the control area's vertical allocation and visually crossed the segment buttons.

A second implementation moved the horizontal scrollbar to a separate child below the controls while sharing the same horizontal adjustment. Runtime testing then confirmed both required sizing states:

- at wide widths the complete segmented strip fits naturally with no scrollbar; and
- at narrow widths the outer container is bounded, the segment row scrolls horizontally, and the reserved scrollbar remains physically below the controls without overlapping them.

Git Bench later provided additional runtime evidence for the shared scrollbar identity: the reserved horizontal thumb and the vertical overlay thumb in the Git Guide were styled with the same cross-axis thickness and rounded handle geometry while retaining their different placement behavior.

The behavioral contract and the Git Bench Reserved / Integrated composition are therefore runtime-proven in one application. Overlay remains a supported presentation variant that must be verified in each concrete segmented-control composition before that implementation is treated as canonical.

A single canonical Rust widget implementation has not yet been proven across the application suite, so the current status remains **Reusable candidate** rather than suite-wide implementation standard.
