# Changelog

All notable changes to GNOME Rust Engineering are documented here.

The repository uses semantic versioning for the engineering knowledge baseline. Reference applications are versioned independently.

## 1.0.0 — 2026-08-10

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
