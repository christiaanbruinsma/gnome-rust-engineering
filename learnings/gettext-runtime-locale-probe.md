# Localization needs a runtime locale probe, not only a compiled catalog

## Context / problem

A translation catalog can exist while the packaged app still fails to show the expected runtime locale behavior.

## What happened?

Data Inspector had a documented language set and localized metadata, and the runtime verification also included launching the installed app under specific locale settings.

## Cause

Catalog presence alone does not prove the packaged runtime is actually loading the expected language data and metadata.

## Proven solution

Validate localization inside the packaged/runtime environment with the intended language settings.

## Evidence

The reference baseline documents a locale smoke against the installed Flatpak, not just the source files.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: no

## Pitfalls

Do not treat `.po` completeness as proof that the shipped package behaves correctly at runtime.
