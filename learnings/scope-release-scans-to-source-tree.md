# Release scans must be scoped to the actual source tree

## Context / problem

Searches for legacy product names can report false positives if they run across caches, build directories, or other generated artifacts.

## What happened?

A Data Inspector release scan found historic File Auditor material in `.flatpak-builder` cache paths rather than in the actual release source tree.

## Cause

The scan boundary was too broad.

## Proven solution

Scope release-name and source-hygiene scans to the real source tree and treat cache hits separately from product-source hits.

## Evidence

The Data Inspector launch/release work explicitly separated source-bound findings from cache-bound historic content.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: yes

## Pitfalls

Do not treat generated cache content as proof that the current product source is contaminated.
