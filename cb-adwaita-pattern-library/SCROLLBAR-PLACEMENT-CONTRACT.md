# CBAPL — Scrollbar Placement Contract

**Status:** Shared CBAPL design contract  
**Applies to:** Scrollable CBAPL components, including CBAPL-001 and CBAPL-003

## Purpose

CBAPL supports two scrollbar placement variants:

- **Overlay**
- **Reserved / Integrated**

Both are valid. The choice must be made consistently from the content and interaction model rather than from local visual preference.

This contract defines when each placement should be used and preserves one shared scrollbar visual identity across both variants.

## Core rule

> **Use Overlay when scrolling is secondary and edge overlap is harmless. Use Reserved / Integrated when the scrollbar must not compete with content or controls.**

Placement may vary. Scrollbar identity must not.

## Default choices

Use these defaults unless a concrete content or interaction reason overrides them:

| Situation | Default |
| --- | --- |
| Vertical lists, documents, history, passive content | Overlay |
| Horizontal interactive controls, segmented strips, tab-like selectors | Reserved / Integrated |
| Scrollbar can cover labels, controls, focus, selection, or important edge content | Reserved / Integrated |
| Scrollability is an important task affordance and may otherwise be missed | Reserved / Integrated |
| Passive content with a safe, non-critical edge | Overlay |

These are defaults, not orientation rules. Horizontal Overlay and vertical Reserved / Integrated remain supported when their concrete content model justifies them.

## Decision sequence

Choose placement in this order.

### 1. Can an overlay interfere with important content or interaction?

Examples:

- button labels;
- row actions;
- selection indicators;
- focus rings;
- resize or drag handles;
- code or data that must remain readable to the edge;
- any control whose pointer/touch target occupies the scrollbar edge.

If **yes**, use **Reserved / Integrated**.

Do not attempt to compensate with arbitrary padding around an overlay scrollbar when a reserved layout can remove the conflict structurally.

### 2. Is scrolling itself an important or non-obvious affordance?

Horizontal overflow is often less obvious than vertical overflow, especially for interactive control strips.

If users need clear evidence that additional task-relevant content or controls exist outside the viewport, prefer **Reserved / Integrated**.

This is why CBAPL-003 Adaptive Segmented Strip uses Reserved / Integrated as its default: hidden workflow/category controls are task-relevant and an overlay scrollbar can collide with the controls themselves.

### 3. Is the viewport primarily passive reading/browsing content with a safe edge?

Examples:

- a vertical list;
- history entries;
- document/reference content;
- passive search results;
- similar content where the scrollbar occupies only a non-critical edge.

If **yes**, prefer **Overlay**. It preserves content area and provides the quieter modern presentation appropriate to secondary scrolling affordances.

This is the normal default for CBAPL-001 Adaptive Scroll Card.

### 4. If neither choice is clearly required

Use this tie-breaker:

- **passive vertical content → Overlay**;
- **interactive horizontal controls → Reserved / Integrated**;
- otherwise choose the variant already established for the same component role elsewhere in the application or suite.

Do not introduce a new placement variant merely for local visual preference.

## Consistency rule

Scrollbar placement is part of the component's presentation contract.

For the same component role:

- keep the chosen placement consistent across normal responsive states;
- do not switch between Overlay and Reserved / Integrated merely because the window crosses an arbitrary width breakpoint;
- the scrollbar may appear or disappear naturally depending on whether overflow exists;
- change the placement only when the content/interaction model changes or runtime evidence proves the existing choice unsuitable.

This avoids unnecessary layout shifts and makes scrolling behavior predictable across the application suite.

## Overlay contract

Overlay is appropriate when:

- scrolling is secondary to the content;
- edge overlap is harmless;
- the scrollbar does not obscure or compete with controls;
- discoverability of overflow does not depend on a persistent scrollbar gutter.

Overlay may use native reveal/hide behavior.

The scrollbar must still use the shared CBAPL thumb identity.

## Reserved / Integrated contract

Reserved / Integrated is appropriate when:

- overlay would interfere with content or interaction;
- the scrollbar needs a dedicated hit area;
- horizontal overflow contains important task controls;
- clear scroll affordance materially improves usability.

The scrollbar receives its own allocation **inside the same outer component**:

- right-side gutter for typical vertical scrolling;
- bottom row for typical horizontal scrolling.

It remains visually integrated with the component and must not look like a detached auxiliary control.

## Shared scrollbar identity

Placement does not define appearance.

Overlay and Reserved / Integrated scrollbar handles must use the same visual language:

- equal cross-axis thumb thickness;
- equal rounded/pill geometry;
- consistent resting contrast and opacity;
- consistent hover emphasis;
- consistent dragging emphasis;
- restrained trough treatment;
- orientation changes thumb direction and length, not its visual identity.

> **Placement may vary. Scrollbar identity must not.**

## Runtime evidence

Git Bench provided the initial runtime evidence for this contract in the Git Guide:

- the vertical topic list uses an **Overlay** scrollbar because it is passive vertical browsing content and the right edge is non-critical;
- the horizontal category strip uses a **Reserved / Integrated** scrollbar because its contents are interactive controls and an overlay implementation collided with the controls;
- both use the same visible drag-handle thickness and rounded geometry.

The result preserves different placement behavior while making both scrollbars visibly part of the same design system.

## Review checklist

Before choosing or changing scrollbar placement, verify:

1. Does the scrollbar overlap any task-relevant content or control?
2. Does it interfere with focus, selection, drag, resize, or pointer targets?
3. Is overflow obvious enough without a reserved scrollbar area?
4. Is the content passive or interactive?
5. Is the choice consistent with the same component role elsewhere?
6. Does the thumb retain the shared CBAPL scrollbar identity?
7. Has the chosen placement been checked at both overflowing and non-overflowing sizes?

If answers 1 or 2 are yes, Reserved / Integrated is the safe choice.
