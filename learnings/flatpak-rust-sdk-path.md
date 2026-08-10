# Flatpak Rust SDK path must be exposed explicitly

## Context / problem

Installing the Rust SDK extension is not the same as making Cargo available inside the build environment.

## What happened?

A GNOME Builder/Flatpak project could still miss the Rust toolchain path unless the manifest or build options explicitly exposed the SDK bin directory.

## Cause

The extension was present, but the build environment did not automatically know to use it.

## Proven solution

Declare the Rust SDK extension and append `/usr/lib/sdk/rust-stable/bin` to the build path in the manifest when using the rust-stable Flatpak extension.

## Evidence

The manifest baseline records both the SDK extension and the appended path.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: no

## Pitfalls

Do not confuse “extension installed” with “extension in PATH”.
