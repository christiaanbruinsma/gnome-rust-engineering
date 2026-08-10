# Stale source-contract tests must not override validated runtime behavior

## Context / problem

A test can remain aligned to an old source string or an old UI representation even when the runtime behavior has already moved on.

## What happened?

In the Rust GNOME work, a test continued to assert an older API or text shape while the real behavior under the current app had already changed.

## Cause

The test itself was stale.

## Proven solution

Update or retire the test only after checking whether it is still asserting the correct contract. Do not let a stale test force product code back to an old implementation shape.

## Evidence

The project notes explicitly describe a stale source-contract test and warn that authored tests are not executed proof of current runtime truth.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: yes

## Pitfalls

Do not change working behavior just to satisfy a test that is no longer describing the correct contract.
