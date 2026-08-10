# `spawn_blocking` boundaries must handle nested results and panic payloads explicitly

## Context / problem

Background work that uses `gio::spawn_blocking` can produce layered results and can fail with panic-payload types that are not immediately obvious from the outer return signature.

## What happened?

Git Bench's compiler output showed an `E0277` around `Box<dyn Any + Send>`, and the inner callback result was `Result<CommandOutput, io::Error>` rather than the simpler type the code expected.

## Cause

The function boundary returned a panic payload or nested `Result` shape that had not been handled explicitly.

## Proven solution

Treat `spawn_blocking` results as explicit boundary objects. Handle panic payloads and nested results deliberately instead of assuming a flat success path.

## Evidence

The recorded build output from Git Bench showed the exact `E0277` and inner `Result<CommandOutput, io::Error>` mismatch.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: yes

## Pitfalls

Do not hand-wave the blocking boundary just because the blocking code is short.
