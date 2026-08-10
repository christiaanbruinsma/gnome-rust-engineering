# Build normally before reaching for Clean/Rebuild

## Context / problem

A clean rebuild can hide the fact that a normal incremental build would have been enough, and it can erase useful evidence from the current build state.

## What happened?

Across the GNOME Rust work, the documented baseline settled on normal Build as the default and Clean/Rebuild only when a toolchain or configuration change or stale artifact had actually been demonstrated.

## Cause

Clean/Rebuild is not a diagnostic proof; it is a state reset.

## Proven solution

Use normal Build first. Only use Clean/Rebuild when the evidence shows that the current artifacts or configuration are the problem.

## Evidence

The repository baseline and the Data Inspector Golden Standard both state normal Build as the default and Clean/Rebuild only for demonstrated need.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: yes

## Pitfalls

Do not turn Clean/Rebuild into a reflex. It can erase evidence and create unnecessary churn.
