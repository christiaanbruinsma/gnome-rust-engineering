# GTK callbacks should keep ownership boundaries explicit

## Context / problem

A GTK closure often needs owned data to survive beyond the current stack frame.

## What happened?

In a Rust GTK flow, using a moved value in a closure could trigger ownership errors such as `E0382` when the same data was still needed elsewhere.

## Cause

The closure captured ownership too aggressively instead of cloning or otherwise structuring the owned data boundary correctly.

## Proven solution

Clone the data intentionally when the callback needs its own owned copy, and keep the overall ownership boundaries explicit.

## Evidence

The Git Bench compiler output showed the `E0382` ownership boundary error and the later stable callback implementation used explicit ownership handling.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: no

## Pitfalls

Do not treat callback ownership errors as a sign to suppress borrow checking; they are often a sign to make the data flow clearer.
