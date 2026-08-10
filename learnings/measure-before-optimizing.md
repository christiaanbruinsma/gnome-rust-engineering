# Measure before optimizing

## Context / problem

Performance work is easy to misdirect if the actual bottleneck is not measured first.

## What happened?

Image Bench showed a clear timing breakdown where resize dominated the path, while decode and encode were much smaller.

## Cause

A feature or backend concern can seem important while the real cost sits in another stage.

## Proven solution

Measure each major stage before proposing a rewrite or acceleration strategy.

## Evidence

The Image Bench runtime logs recorded `decode_ms=0.00`, `resize_ms=3556.62`, `encode_ms=91.49`, `total_ms=3648.23`, and `resize_backend=Cpu`, which pointed directly at resizing as the bottleneck.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: yes

## Pitfalls

Do not optimize the wrong stage because a different part of the stack looks more exciting.
