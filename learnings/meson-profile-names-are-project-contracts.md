# Meson profile names are project contracts, not conventions

## Context / problem

Release handovers and copied build commands can casually refer to a `release` profile even when a project's Meson configuration uses a different vocabulary.

The profile name accepted by a project is defined by that project's `meson.options`; it must not be inferred from common words such as release, production, default or devel.

## What happened?

During Delivery Hub v0.9.0 release preparation, a handover instructed the release build to use:

```text
-Dprofile=release
```

Inspection of `meson.options` proved that the project only accepted:

```meson
choices: ['default', 'development']
value: 'default'
```

The correct release build therefore used:

```text
-Dprofile=default
```

Meson setup and compile then passed with that actual project contract.

## Cause

A release concept was mistaken for a literal Meson option value. The handover carried an assumption that was not validated against the source-of-truth configuration.

## Proven solution

Before issuing or reusing any profile-specific Meson command:

1. inspect `meson.options`;
2. identify the exact allowed values and default;
3. map semantic intent such as release/development onto those literal values;
4. use the project's actual option value in Builder and Flatpak manifests;
5. reject handover wording when it conflicts with source configuration.

## Evidence

Delivery Hub v0.9.0 `meson.options` exposed exactly two choices: `default` and `development`, with `default` as the release baseline. Release Meson setup and compile passed with `-Dprofile=default`.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: reusable configuration rule

## Pitfalls

- Do not assume every project has a `release` profile.
- Do not infer Meson option values from a `.Devel` application ID or from another project's manifest.
- Do not let a handover override current source configuration.
- Treat `meson.options` as the source of truth for allowed profile values.

## Provenance

- Originating project: Delivery Hub v0.9.0
- Gate or phase: release Meson setup
- Evidence type: source inspection and successful Meson release build
- Validation result: PASS with `-Dprofile=default`
