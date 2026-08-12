# Localize presentation, not domain contracts

## Context / problem

Application logic can accidentally depend on human-readable strings. If those strings are translated directly at the service or domain layer, localization can change behavior instead of only changing presentation.

## What happened?

Delivery Hub v0.9.0 had verification logic and UI handling around stable verifier result messages. Translating those raw service/domain messages directly would have changed values that existing UI logic matched or classified.

The localization integration therefore kept the verifier's internal/domain-facing messages stable and localized them at the UI presentation boundary instead.

## Cause

Human-readable strings were serving two roles at once:

- machine-facing behavior/state contract;
- user-facing presentation.

Those concerns must not be coupled through translated text.

## Proven solution

Prefer typed state for machine behavior and translate only the presentation:

```text
domain enum / typed result
    ↓
UI mapping
    ↓
gettext-localized label/message
```

When a legacy implementation already relies on stable string values, keep those values unchanged during localization and add a presentation-localization layer until the domain contract can be refactored to typed state safely.

## Evidence

Delivery Hub v0.9.0 localization was integrated without translating raw verifier domain messages directly. UI-facing verification text was localized at the presentation layer while verification behavior and the complete 46-test suite remained stable.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: reusable architecture rule

## Pitfalls

- Do not use translated strings as identifiers, enum substitutes or branching keys.
- Do not compare localized UI labels to decide application behavior.
- Do not move gettext into domain/service layers merely for convenience when those values participate in logic.
- If a string contract already exists, preserve behavior first and refactor toward typed state separately.

## Provenance

- Originating project: Delivery Hub v0.9.0
- Gate or phase: localization integration before release
- Evidence type: source architecture, executed tests, packaged runtime localization QA
- Validation result: localization PASS with behavior preserved
