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

- **MUST** means required for conformance when applicable.
- **SHOULD** means the preferred evidence-backed default; deviation requires a clear technical reason.
- **MAY** means optional and context-dependent.
- **N/A** means genuinely not applicable.
- **NOT TESTED** means applicable but not yet verified.

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

## Localization and packaged Flatpak boundary

For localized applications, the localization system is a complete delivery chain, not only a source-code concern:

```text
PO catalogs
    ↓
MO generation
    ↓
Meson install manifest
    ↓
Flatpak packaging model
    ↓
installed locale files or extension subpaths
    ↓
runtime locale
    ↓
gettext lookup
    ↓
translated UI
```

The application SHOULD use GNU gettext through a small i18n boundary and keep English as the source/fallback language. The suite localization order is English, Dutch, German, French, Spanish, Italian, Portuguese.

### Distribution model decides locale packaging

Flatpak Builder defaults to separating locale data into a `.Locale` extension. That remains a valid and appropriate model for repository-based distribution such as Flathub or another OSTree repository where application extensions are part of the distribution contract.

For this suite's standalone GitHub `.flatpak` releases, where one downloadable file is intended to be the complete installable release artifact, supported gettext catalogs SHOULD be embedded in the application payload with:

```yaml
separate-locales: false
```

This rule is distribution-specific, not universal. Do not disable locale splitting merely because it is inconvenient to debug, and do not retain locale splitting merely because it is Flatpak's default when the actual release contract is a single self-contained bundle.

### Repository / Locale-extension validation

When locale splitting is enabled, distinguish these gates:

1. `.po` completeness;
2. `.mo` generation;
3. Meson install registration;
4. Flatpak Locale extension creation;
5. installed Locale subpaths/catalogs;
6. runtime locale availability;
7. gettext lookup inside the packaged runtime;
8. application UI translation.

A `.Locale` extension may exist in a local OSTree repository while only a subset of its language subpaths is installed locally. Testing a language that is not present in the installed Locale extension cannot establish an application gettext failure.

For local repository testing, it is valid to update/install the Locale extension separately and then repeat runtime locale smoke.

### Standalone-bundle validation

When `separate-locales: false` is used for a standalone GitHub release, the release gate MUST validate the actual standalone artifact, not only the source tree or local repository:

1. confirm every supported `.mo` file is present in the built application payload;
2. run explicit locale smoke for the supported matrix from the packaged application;
3. build the final standalone `.flatpak`;
4. install or reinstall that exact bundle;
5. perform at least one explicit locale sanity check on the installed standalone bundle;
6. record and publish the final artifact checksum.

Delivery Hub v0.9.0 provides direct evidence for this boundary: six non-source catalogs were embedded, all six supported translations passed packaged runtime smoke, and the exact final standalone bundle passed an installed Dutch locale sanity check before publication.

A successfully generated `.mo` file does **not** prove that the installed application can use it. A successful local-repository test also does not, by itself, prove that the exact downloadable standalone artifact is complete.

Supported locale runtime smoke SHOULD include each supported non-source language using explicit locale environment variables, for example:

```bash
flatpak run \
  --env=LANG=nl_NL.UTF-8 \
  --env=LANGUAGE=nl \
  <APP_ID>
```

The equivalent check SHOULD be repeated for `de`, `fr`, `es`, `it`, and `pt` when those languages are supported. User data, file contents, identifiers, and source values MUST remain untranslated.

Localization MUST remain a presentation concern. Machine-facing state, identifiers, enum values and domain contracts MUST NOT depend on translated strings. When legacy behavior already depends on stable human-readable strings, preserve those values and localize at the UI boundary until the domain contract can be typed safely.

## Conformance

When auditing an application against the Golden Standard, classify each applicable requirement as:

- **PASS** means the named requirement was actually verified.
- **DEVIATION** means the implementation differs; document the technical reason and impact.
- **NOT TESTED** means applicable but not verified.
- **N/A** means genuinely not applicable.

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
