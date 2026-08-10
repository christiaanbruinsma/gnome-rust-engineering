# Project template

## Purpose

Describe the application's job and why it belongs in the suite.

## Reference baseline

State which existing app or baseline the project should compare itself against.

## Layout

Document the planned files, modules, UI areas, and data flow.

## Validation

List the build, runtime, test, packaging, and export checks that define success.

## Learnings

Record project-specific learnings first in `docs/LEARNINGS.md`.

## Reuse path

Move candidate lessons into `learnings/` only when evidence suggests they may be reused.

## Environment reconstruction

Before implementation or audit, follow [`../PROJECT-CONFIGURATION.md`](../PROJECT-CONFIGURATION.md) to reconstruct the GNOME Builder, Rust SDK, Meson, Flatpak, custom command, and QA environment. Do not assume a remembered configuration.
