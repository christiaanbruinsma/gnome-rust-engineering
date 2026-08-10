# Tags should only be created after the final source gate

## Context / problem

A tag can be published before the source inventory is actually complete, which creates a release mismatch.

## What happened?

The Data Inspector release work showed why `COPYING` and final source verification had to be closed before the tag could represent the release accurately.

## Cause

The tag boundary was too early.

## Proven solution

Do the final source, license, metadata, and artifact gate first. Create or finalize the tag only after the release inventory is complete.

## Evidence

The Data Inspector v0.9.0 release path tied the final tag to the commit that added the GPL-3.0 license text and resolved the source inventory boundary.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: yes

## Pitfalls

Do not let an early tag become the release record before the source gate is actually done.
