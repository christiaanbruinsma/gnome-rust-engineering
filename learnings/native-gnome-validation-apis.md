# Native GNOME validation APIs should be used before custom shortcuts

## Context / problem

A project can be tempted to skip the platform's own validation helpers and trust custom checks too early.

## What happened?

The GNOME Rust work repeatedly used platform-native validation points such as AppStream validation, runtime icon audit, and installed Flatpak checks rather than trusting only source-level assumptions.

## Cause

Custom checks are often incomplete unless anchored to the actual platform/runtime boundary.

## Proven solution

Use native platform validation APIs and tools where available before inventing a project-specific proxy check.

## Evidence

The Data Inspector Golden Standard requires generated AppStream validation, runtime icon audit, and installed Flatpak QA.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: yes

## Pitfalls

Do not claim a release is validated because one internal smoke test passed.
