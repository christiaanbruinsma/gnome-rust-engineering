# CB Adwaita Pattern Library

The **CB Adwaita Pattern Library (CBAPL)** is a project-authored companion layer for native GTK4/libadwaita applications.

It documents reusable UI, interaction, layout, and semantic patterns that are useful across applications but are not fully specified by the GNOME Human Interface Guidelines. CBAPL does **not** replace GTK, libadwaita, or the GNOME HIG, and it is not official GNOME guidance.

## Relationship to the GNOME stack

CBAPL sits above the upstream application stack:

```text
GTK4
  ↓
libadwaita
  ↓
GNOME Human Interface Guidelines
  ↓
CB Adwaita Pattern Library
  ↓
Applications
```

GTK4 and libadwaita provide the native primitives. The GNOME HIG remains the upstream design foundation. CBAPL adds project-defined contracts for recurring situations where a more explicit, consistent application-level pattern has proven useful.

A CBAPL pattern must not intentionally contradict upstream GNOME guidance. When upstream guidance changes, the pattern must be reviewed.

## Evidence model

CBAPL follows the evidence-first rules of GNOME Rust Engineering.

Each pattern should state:

- its purpose;
- its behavioral contract;
- its upstream foundation;
- what the CBAPL layer adds;
- implementation guidance without over-prescribing unproven details;
- accessibility or semantic constraints where relevant;
- its evidence boundary and current validation status.

A useful implementation in one application can justify documenting a reusable candidate. It does not automatically prove suite-wide correctness.

## Naming

Patterns use stable identifiers:

```text
CBAPL-001
CBAPL-002
CBAPL-003
...
```

Identifiers remain stable even if a pattern is refined later.

## Current patterns

| ID | Pattern | Status |
| --- | --- | --- |
| [CBAPL-001](CBAPL-001-adaptive-scroll-card.md) | Adaptive Scroll Card | Reusable candidate |
| [CBAPL-002](CBAPL-002-severity-eyebrow-dialog.md) | Severity Eyebrow Dialog | Reusable candidate |
| [CBAPL-003](CBAPL-003-adaptive-segmented-strip.md) | Adaptive Segmented Strip | Reusable candidate — runtime-proven in Git Bench |

## Scope

CBAPL is intended for recurring application-level concerns such as:

- adaptive containers and scrolling;
- dialog hierarchy and semantic severity;
- navigation and inspector behavior;
- responsive multi-pane layouts;
- selection and action-state conventions;
- other interaction patterns that can be expressed using native GTK4/libadwaita building blocks.

It is not a replacement widget toolkit and does not require applications to depend on a separate code library.

If a pattern later gains a reusable Rust widget or helper implementation, that code may be documented separately while the behavioral contract remains the authoritative pattern definition.
