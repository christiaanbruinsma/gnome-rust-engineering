# Backend availability is not backend activation

## Context / problem

A system can report that a backend or device is present without proving that the application actually used it on the critical path.

## What happened?

In Image Bench, the environment reported Vulkan support and a discrete GPU, but the measured resize path still ran on the CPU. The first visible bottleneck was the resize stage, not decoding or encoding.

## Cause

Backend availability was observed, but the actual resize implementation had not yet switched to the GPU path.

## Proven solution

Do not assume that detected hardware, a loaded library, or a runtime capability report means the application is using that path. Measure the actual work path and confirm where the time is spent.

## Evidence

The performance logs showed `resize_backend=Cpu` while the GPU itself was available.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: no

## Pitfalls

Avoid documenting a backend as active just because the environment can see it.
