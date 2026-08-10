# Large-file hashing should stream instead of loading everything at once

## Context / problem

Large file inspection, checksum work, or export validation can become memory-heavy if the whole file is loaded before hashing.

## What happened?

The Rust migration work introduced checksum-style verification patterns and emphasized processing files in a streaming fashion.

## Cause

Loading everything into memory is unnecessary for many hash and validation tasks.

## Proven solution

Use streaming file processing for large-file hashing and validation rather than buffering the entire file at once.

## Evidence

The GNOME Rust engineering notes and file-validation work repeatedly treat streaming verification as the responsible default for large inputs.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: no

## Pitfalls

Do not assume the in-memory approach is acceptable just because a small sample file passes.
