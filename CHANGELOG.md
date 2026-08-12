# Changelog

All notable changes to GNOME Rust Engineering are documented here.

The repository uses semantic versioning for the engineering knowledge baseline. Reference applications are versioned independently.

## Unreleased

No unreleased changes recorded.

## 1.1.0 - 2026-08-13

Evidence-backed update from Image Bench, Snippet Manager, Signature Designer, PDF Workbench and Delivery Hub release work.

### Added

- Standalone Flatpak localization learning with `separate-locales: false` for the suite's self-contained GitHub `.flatpak` distribution model.
- Manual target-SDK linker-boundary learning, including targeted `-C linker-features=-lld` diagnosis without prematurely patching the product build.
- Test-fixture branch-isolation learning: earlier validation guards must remain satisfied when a test intends to exercise a downstream failure mode.
- Meson profile contract learning: profile names must be read from `meson.options`, not inferred from words such as release or development.
- Flatpak native dependency discoverability learning based on PDF Workbench Poppler/pkg-config evidence.
- Localization architecture learning: keep machine-facing domain contracts stable and translate at the presentation boundary.
- Proven local-repository Flatpak localization QA workflow based on Signature Designer v0.9.0 runtime evidence.
- Explicit verification of `.Locale` catalog resolution through `/app/share/locale` symlinks using `find -L`.
- Application-neutral `LANG`/`LANGUAGE` smoke workflow for the suite localization matrix.

### Changed

- Revised the Golden Standard localization model to distinguish repository/Flathub `.Locale` distribution from standalone self-contained GitHub bundles.
- Superseded the earlier locale-splitting-only guidance in the gettext runtime learning.
- Updated project configuration with distribution-specific locale packaging, standalone-bundle QA, linker-boundary diagnostics, test-fixture isolation and native dependency discoverability.
- Strengthened the requirement that localization remains a presentation concern rather than a machine-facing state contract.
- Clarified that manual SDK-shell failures must be reproduced in the actual Builder/Flatpak pipeline before persistent build workarounds are adopted.

### Evidence boundary

The `separate-locales: false` rule in this release is intentionally scoped to this suite's standalone GitHub `.flatpak` distribution contract. Flatpak's default `.Locale` extension model remains valid for repository-based distribution. The update records both models rather than claiming one universal packaging rule.

## 1.0.0 - 2026-08-10

Initial stable public baseline.

### Included

- Evidence-first engineering and promotion model.
- Standards for architecture, Rust, GTK4, libadwaita, async/threading, error handling, Flatpak, and Meson.
- Reusable patterns for GTK state/signals, background work, file handling, drag and drop, portals, and i18n.
- Evidence-bounded learnings gathered from real Rust/GNOME development and migration work.
- Application-neutral GNOME Builder + Rust project configuration and recovery guide.
- Exact Builder custom-command configuration for strict Clippy, Cargo tests, and AppStream validation.
- Development/release identity model with `.Devel` separation.
- Target-SDK validation guidance and Rust SDK-extension configuration.
- Semantic GNOME/GTK icon policy and installed-Flatpak runtime verification boundary.
- Localization baseline: English fallback plus Dutch, German, French, Spanish, Italian, and Portuguese.
- Release validation ladder from source/static checks through exported and installed Flatpak QA.
- Templates for project onboarding and evidence-backed learnings.
- Data Inspector v0.9.0 as the approved Golden Standard reference application.

### Version boundary

This `1.0.0` version belongs to **GNOME Rust Engineering**, not to Data Inspector or another application. Reference applications retain their own release histories and versions.
