# Icon theme verification must happen in the runtime boundary

## Context / problem

An icon can exist in source or in a theme audit while still failing to resolve in the installed application runtime.

## What happened?

Data Inspector's icon policy was verified against the active runtime icon theme, and the installed Flatpak had to follow the same semantic icon rules.

## Cause

Theme availability outside the actual installed application environment is not enough to prove final icon resolution.

## Proven solution

Verify semantic icon names in the app runtime and then verify the installed Flatpak against the host's active icon theme.

## Evidence

The Data Inspector Golden Standard and changelog both require runtime icon audits and installed-Flatpak icon checks.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: yes

## Pitfalls

Do not hardcode a theme or assume that a semantic icon name resolves everywhere without checking.
