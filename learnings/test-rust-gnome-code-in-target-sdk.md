# Rust GNOME code should be tested in the target SDK

## Context / problem

Host-side Cargo or library checks can validate the wrong environment when the real app ships inside a GNOME SDK/Flatpak boundary.

## What happened?

The project notes show that host Cargo could fail for missing native GNOME dependencies while the Builder/SDK environment was the one that actually matched the runtime target.

## Cause

The host environment and the target GNOME SDK environment were not the same.

## Proven solution

Run the GNOME/Rust validation path inside the target SDK / Builder pipeline, not only on the host.

## Evidence

The recorded GNOME Builder and SDK notes explicitly treat the target SDK as the source of truth for native dependency checks and runtime validation.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: yes

## Pitfalls

Do not mistake a host compile for a correct Flatpak/runtime validation.
