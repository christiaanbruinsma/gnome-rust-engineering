# Stale overlay/module files can survive if the source tree is not cleaned deliberately

## Context / problem

A source overlay or imported tree can leave behind obsolete module files even after the visible code path has moved on.

## What happened?

Data Inspector had a `src/window.rs` versus stale `src/window/mod.rs` mismatch that needed cleanup.

## Cause

The release source set still contained old module layout artifacts.

## Proven solution

Audit the tree for stale module files and remove old variants explicitly rather than assuming the overlay or patch process removed them.

## Evidence

The Data Inspector reference work recorded a stale overlay/module-file issue as part of its source hygiene pass.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: yes

## Pitfalls

Do not rely on one build path to clean up another file layout automatically.
