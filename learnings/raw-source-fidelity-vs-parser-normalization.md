# Raw source fidelity is separate from parser normalization

## Context / problem

A structured view and the original source are not the same product requirement.

## What happened?

Data Inspector deliberately kept a Raw view and a structured view separate so that the original input could remain visible while parser-normalized data was still available.

## Cause

Parsing often changes representation in ways that are useful for inspection but not identical to the source.

## Proven solution

Keep raw source fidelity and parser normalization as separate responsibilities.

## Evidence

The Data Inspector README and changelog describe Structured and Raw views and the preservation of decoded Raw source fidelity.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: yes

## Pitfalls

Do not rewrite the raw source just to make the parser happy.
