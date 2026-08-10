# Release and development app IDs should be distinct but derived from the same base identity

## Context / problem

A project can easily drift into duplicated UI or resource trees when trying to support both development and release identities.

## What happened?

Data Inspector used a base application ID with a `.Devel` variant derived from the same source tree and templates.

## Cause

The project needed two identities but one implementation.

## Proven solution

Derive the development ID from the base ID in Meson and use the same source tree for both variants.

## Evidence

The Data Inspector manifest and Meson configuration show stable and development identities built from one source tree.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: yes

## Pitfalls

Do not duplicate UI or resource files just to change the application ID.
