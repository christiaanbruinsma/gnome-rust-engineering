# Migration parity is a contract, not an assumption

## Context / problem

When a project is migrated from one implementation to another, parity has to be defined and checked rather than assumed.

## What happened?

The Rust migration work across the GNOME apps repeatedly had to preserve app identity, user-facing behavior, metadata, and validation boundaries while changing internal implementation details.

## Cause

A migration can easily preserve the wrong things or lose the important ones if parity is not defined.

## Proven solution

Write down the parity contract explicitly. Include identity, visible behavior, metadata, release packaging, and any known edge-case behavior that the old implementation already proved.

## Evidence

The repository's evidence model and release baseline treat migration results as proven only when the actual behavior, metadata, and packaging gates are verified.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: yes

## Pitfalls

Do not assume the new implementation is “close enough” without spelling out what parity means.
