# Golden Standard

## Definition

The **GNOME Rust Engineering Golden Standard** is this project's current evidence-backed engineering baseline for building, validating, packaging, and maintaining native GNOME applications.

It describes the strongest practices, architecture, tooling, workflows, and quality gates that this project has verified through real engineering work.

**Golden Standard does not mean universal, permanent, infallible, or officially endorsed by GNOME.** It means that, within the explicitly documented evidence boundary, this is the strongest verified baseline currently maintained by this project.

The standard is deliberately revisable. New platform capabilities, official guidance, stronger evidence, or demonstrated failures can change it.

## What “proven” means

A statement is **proven** only within its stated evidence boundary.

Acceptable evidence can include:

- source inspection;
- compiler/build output;
- static validation;
- automated tests that were actually executed;
- runtime output and behavior;
- official validation tooling;
- packaged application inspection;
- installed Flatpak behavior;
- a working, retested patch.

“Proven” does **not** mean universally correct. It means demonstrated under the documented conditions and at the documented validation gate.

Examples:

```text
compile PASS ≠ runtime PASS
runtime PASS ≠ installed Flatpak PASS
authored tests ≠ executed tests
hardware/backend available ≠ backend actually used
```

## Evidence and promotion model

Knowledge should move upward only when the evidence supports it:

```text
Observation
    ↓
Project-specific learning
    ↓
Reusable candidate
    ↓
Repeated or sufficiently strong evidence
    ↓
Golden Standard
```

Not every lesson must reach the top.

### Observation

Something has been seen in code, output, behavior, documentation, or tooling. Its cause or reusability may still be unknown.

### Project-specific learning

The cause and solution are sufficiently demonstrated for one project, but the lesson may depend on that project's architecture or constraints.

### Reusable candidate

There is a technical reason to expect reuse across projects, backed by evidence, but the project does not yet claim it as a general engineering requirement.

### Golden Standard

The evidence is strong enough for this project to adopt the rule as its current default engineering baseline.

Promotion is not based on preference, repetition in documentation, or one developer's confidence.

## Normative language

When a Golden Standard document uses these terms:

- **MUST** — required for conformance when applicable.
- **SHOULD** — the preferred evidence-backed default; deviation requires a clear technical reason.
- **MAY** — optional and context-dependent.
- **N/A** — genuinely not applicable.
- **NOT TESTED** — applicable but not yet verified.

An unknown or untested item must never be silently converted to PASS or N/A.

## Relationship to official platform guidance

This repository is an independent engineering knowledge project. It is not official GNOME, GTK, libadwaita, Rust, Meson, or Flatpak documentation.

Relevant current official documentation and platform behavior should be checked when implementing or revisiting a rule, especially when versions have changed.

If current official platform capabilities or documentation conflict with an older local pattern, investigate the difference rather than preserving the old pattern by default.

## Reference implementations

A **reference implementation** is a real application that demonstrates a substantial portion of the Golden Standard through actual source, build, runtime, packaging, and QA evidence.

A reference implementation is evidence for the standard; it is not the definition of the standard.

Therefore:

```text
reference implementation ≠ every implementation detail is a standard
Golden Standard ≠ exact copy of the reference application's UI or feature model
```

Applications may legitimately differ when their requirements differ.

## Data Inspector v0.9.0

**Data Inspector v0.9.0 is the first approved Golden Standard reference implementation.**

It provides a concrete, published baseline for the Rust + GTK4 + libadwaita application stack and demonstrates many of the repository's engineering, packaging, native-integration, and QA practices.

This means:

- other projects may use it as a concrete technical reference;
- its verified engineering decisions can supply evidence for central standards and patterns;
- its release process can inform validation gates;
- its literal application ID, feature set, data formats, screen layout, and app-specific implementation are not suite-wide requirements;
- a Data Inspector detail is not automatically a Golden Standard rule merely because Data Inspector uses it;
- Data Inspector can eventually become older than the current Golden Standard as the standard evolves.

The maintained standard therefore sits above any single reference application.

## Conformance

When auditing an application against the Golden Standard, classify each applicable requirement as:

- **PASS** — the named requirement was actually verified.
- **DEVIATION** — the implementation differs; document the technical reason and impact.
- **NOT TESTED** — applicable but not verified.
- **N/A** — genuinely not applicable.

A project should not be called Golden Standard compliant while required applicable gates remain NOT TESTED.

Conformance is evaluated against the current repository baseline, not against visual similarity to Data Inspector.

## Change policy

The Golden Standard is expected to evolve.

A rule may be:

- clarified when wording is ambiguous;
- expanded when new evidence strengthens its scope;
- downgraded when evidence proves narrower than previously claimed;
- superseded when a better native/platform mechanism is demonstrated;
- removed when it is no longer correct or relevant.

Changes must preserve evidence provenance. Historical success is useful evidence, but it does not override newer demonstrated platform behavior.

## Practical interpretation

The Golden Standard should answer:

> Given what we have actually demonstrated so far, what is the strongest responsible default for this engineering decision?

It should never mean:

> Do this because this repository says so.

Users and maintainers are expected to verify applicability in their own environment and at the appropriate validation boundary.

See [DISCLAIMER.md](DISCLAIMER.md) for warranty, verification, and responsibility boundaries.
