# Rust crate module boundaries must be explicit

## Context / problem

A library module can fail to resolve another module if the project's crate structure is not mirrored clearly.

## What happened?

A Rust GNOME app encountered a `crate::config` resolution issue when the library-side module boundary did not match the module layout exposed by `main.rs`.

## Cause

The module existed in the binary side but was not available through the library boundary as referenced.

## Proven solution

Make the crate/module boundary explicit and keep the library module layout aligned with the code that imports it.

## Evidence

The project notes record the `E0432 crate::config` case and its module-boundary fix.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: yes

## Pitfalls

Do not assume a module in `main.rs` is automatically visible everywhere else in the crate.
