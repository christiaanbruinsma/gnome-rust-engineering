# Meson and Cargo have different jobs

## Context / problem

Rust GNOME projects can become confusing if Meson and Cargo are treated as interchangeable build systems.

## What happened?

The project baseline explicitly separates Meson's role as the GNOME build/install layer from Cargo's role as the Rust compilation and test layer.

## Cause

Each tool controls a different part of the stack.

## Proven solution

Keep the boundary explicit: Cargo owns Rust dependencies, compilation, tests, Clippy and formatting; Meson owns project configuration, build identity, generated desktop/AppStream files, resource installation, gettext integration and installation layout.

## Evidence

The repository's project configuration and Golden Standard both define this boundary.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: yes

## Pitfalls

Do not replace Meson with Cargo just because the project is Rust.
