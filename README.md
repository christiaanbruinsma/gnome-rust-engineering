# GNOME Rust Engineering

**Current repository release: v1.1.0**

This is the current stable public baseline of GNOME Rust Engineering. Data Inspector v0.9.0 remains the approved reference application; repository and application versions are intentionally independent.

Practical, evidence-first engineering knowledge for building and migrating native GNOME applications with Rust, GTK4, libadwaita, Meson and Flatpak.

This repository is not a generic best-practices list. It records lessons discovered in real application development and only promotes them when the technical cause, solution and verification are known.

## What “Golden Standard” means

The **Golden Standard** is this project's current evidence-backed and continuously revisable engineering baseline. It is **not** a claim of universal best practice, permanent correctness, or official GNOME endorsement.

A rule becomes Golden Standard only when its evidence supports promotion beyond an observation, project-specific learning, or reusable candidate. “Proven” always means proven within a documented evidence boundary.

Data Inspector v0.9.0 is the first approved reference implementation, but the standard sits above any single application: not every Data Inspector implementation detail is automatically a suite-wide rule.

Read [GOLDEN-STANDARD.md](GOLDEN-STANDARD.md) for the full definition, evidence/promotion model, conformance rules, and relationship to reference implementations.

## Important notice

This repository is engineering reference material, not a guarantee. Platform versions and environments change, and guidance must be verified for the project and validation boundary in which it is used. Consult current official upstream documentation where relevant and perform your own build, runtime, packaging, security, and release checks.

Read [DISCLAIMER.md](DISCLAIMER.md) before relying on the material.

## Start here

If you are starting, recovering, or auditing a Rust GNOME project, begin with [PROJECT-CONFIGURATION.md](PROJECT-CONFIGURATION.md). It is the application-neutral setup and recovery guide for GNOME Builder, Rust, Meson, Flatpak, custom QA commands, identities, validation gates, icons, and localization.

## CB Adwaita Pattern Library

The [CB Adwaita Pattern Library](cb-adwaita-pattern-library/README.md) is the repository's project-authored companion layer for reusable GTK4/libadwaita UI, interaction, layout, and semantic patterns.

It complements the GNOME Human Interface Guidelines rather than replacing them. Patterns remain evidence-bounded and must not be presented as official GNOME guidance.

Current entries include **CBAPL-001 Adaptive Scroll Card** and **CBAPL-002 Severity Eyebrow Dialog**.

## Goals

- Preserve proven engineering knowledge for reuse across GNOME applications.
- Make migration and debugging decisions traceable to evidence.
- Separate project-specific fixes from reusable patterns and suite-wide standards.
- Share practical GNOME/Rust engineering experience publicly without presenting assumptions as facts.

## Evidence model

A claim can be supported by one or more of:

- runtime output or logs;
- a reproducible test;
- source-code analysis;
- a working patch followed by retesting.

A workaround is not automatically a pattern, and a pattern is not automatically a standard.

## Promotion path

1. **Project-specific**: record it first in the originating project's `docs/LEARNINGS.md`.
2. **Possibly reusable**: when the lesson appears useful outside that project, add it to `learnings/`.
3. **Proven suite-wide**: only after repeated evidence across projects may it be promoted to `patterns/` or `standards/`.

When evidence is incomplete, the document must say so. Unknowns must not be filled in from assumption.

### Evidence provenance rule

For concrete learnings, retain the originating project/gate and distinguish:

- compiler proof;
- executed test proof;
- runtime proof;
- full parity proof.

Authored tests are not executed tests. A successful compile is not a runtime pass. Exact test counts are only recorded when the executed output or a retained project handover/log supports them.

## What changed in v1.1.0

The 1.1.0 baseline adds newer release evidence from Image Bench, Snippet Manager, Signature Designer, PDF Workbench and Delivery Hub.

The most important revision is localization packaging: repository-based `.Locale` extensions and standalone self-contained `.flatpak` downloads are now documented as separate distribution boundaries. For this suite's standalone GitHub bundles, embedded catalogs with `separate-locales: false` are the evidence-backed default; repository distribution may continue to use Flatpak's normal Locale extension model.

The release also adds learnings for manual SDK linker diagnostics, test-fixture branch isolation, Meson profile contracts, native Flatpak dependency discoverability and keeping localization at the presentation boundary.

## Golden reference implementation

**Data Inspector v0.9.0** is the approved Golden Standard reference baseline for the standalone Rust + GTK4 + libadwaita application stack. New applications should compare their project structure, native UI integration, i18n wiring, packaging discipline and QA gates against this baseline where those concerns are applicable.

See `docs/` and the repository root documents for the standards, patterns, and configuration guidance derived from that baseline.
