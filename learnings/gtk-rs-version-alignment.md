# gtk-rs version alignment matters

## Context / problem

A Rust/GTK application can build a mismatched dependency graph if the crate versions do not line up with the target SDK and the rest of the project.

## What happened?

Signature Designer's handover recorded a GTK4/libadwaita version conflict that had to be resolved before the application could be treated as stable.

## Cause

The Rust bindings and native library expectations were not aligned tightly enough with the project's actual environment.

## Proven solution

Align gtk-rs/libadwaita versions with the target SDK and the rest of the project before using the app as a reference for further work.

## Evidence

The Signature Designer migration notes explicitly mention a gtk4/libadwaita version conflict and its resolution boundary.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: no

## Pitfalls

Do not assume a Rust crate version is safe just because the code compiles locally in one environment.
