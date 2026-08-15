# CBAPL-002 — Severity Eyebrow Dialog

**Status:** Reusable candidate  
**Base:** GTK4 / libadwaita  
**Scope:** Application dialogs with clear semantic severity

## Purpose

A **Severity Eyebrow Dialog** gives important dialogs a consistent semantic hierarchy so users can recognize the kind of moment before reading the full message.

The fixed visual hierarchy is:

```text
EYEBROW
TITLE
BODY
ACTIONS
```

The eyebrow carries the semantic category in text. Color reinforces that meaning but never replaces it.

## Severity mapping

| Severity | Eyebrow | Color intent | Use |
| --- | --- | --- | --- |
| Notice / Information | `NOTICE` or `INFORMATION` | Blue | Neutral information or context |
| Confirmation | `CONFIRMATION` | Green | Normal, valid, non-destructive confirmation |
| Success | `SUCCESS` | Green | Completed positive outcome |
| Warning / Attention | `WARNING` or `ATTENTION` | Yellow/orange | Situation requiring review or caution |
| Error | `ERROR` | Red | Failed operation or invalid state |
| Danger | `DANGER` | Red | Destructive or materially risky confirmation |

## Behavioral contract

- The eyebrow is always visible when a dialog has a meaningful semantic category.
- The eyebrow text communicates the category independently of color.
- Color is supplemental and must not be the only signal.
- The title states the concrete decision or event.
- The body explains only the context needed for the user to decide or recover.
- Actions use native GTK4/libadwaita semantics where possible, including destructive styling for destructive actions.
- Destructive confirmations use **DANGER**, not a green `CONFIRMATION` eyebrow.
- `CONFIRMATION` is reserved for ordinary, valid, non-destructive actions.
- Avoid duplicating the same heading through both a native dialog title and a custom content title.

## Upstream foundation

Use native GTK4/libadwaita dialog and action primitives and follow GNOME HIG guidance for concise dialog content, button semantics, destructive actions, and accessibility.

The pattern does not replace native dialog behavior. It adds a consistent application-level semantic header layer where the dialog type benefits from immediate classification.

## CBAPL extension

GNOME and libadwaita provide dialog primitives and semantic action styling. CBAPL adds a stable severity taxonomy plus the `EYEBROW → TITLE → BODY → ACTIONS` hierarchy so applications in the suite present confirmation, warning, error, success, and danger moments consistently.

## Accessibility and semantics

The severity word must remain readable as text. Do not rely on blue, green, yellow, orange, or red alone to communicate meaning.

Button labels should describe the action itself, for example `Abort Revert`, `Delete`, `Continue`, or `Close`, rather than generic confirmation text where a more specific label is available.

## Evidence boundary

The pattern was refined through repeated dialog work in native GTK4/libadwaita applications, including Git Bench recovery, confirmation, warning, and destructive-operation dialogs. Runtime testing established the usefulness of the fixed hierarchy and exposed cases where destructive confirmations were semantically misclassified by a generic green confirmation treatment.

The semantic contract is considered reusable. Exact typography, spacing, and helper implementation may still evolve as the pattern is exercised across more applications, so the current status is **Reusable candidate**.
