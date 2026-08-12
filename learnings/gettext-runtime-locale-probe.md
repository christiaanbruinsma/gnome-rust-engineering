# Gettext localization must be verified at the packaged runtime boundary

## Context / problem

A translated GNOME/Rust application can have complete `.po` files and correctly generated `.mo` catalogs while the installed release still does not show the expected language.

The important boundary is not only source translation completeness. It is the complete distribution chain:

```text
PO → MO generation → Meson install registration
   → Flatpak packaging model → installed locale data
   → runtime locale → gettext lookup → application UI
```

Two Flatpak distribution models must now be treated separately:

1. repository-based distribution using Flatpak's default `.Locale` extension model;
2. standalone `.flatpak` distribution where the downloaded bundle is intended to be the complete release artifact.

This document originally focused only on the first model. Delivery Hub v0.9.0 and later standalone release evidence supersede that limitation.

## Repository / `.Locale` evidence

Image Bench 0.9.0, Snippet Manager 0.9.0 and Signature Designer 0.9.0 demonstrated the repository-based boundary.

The investigation distinguished three states that are easy to conflate:

1. the `.Locale` extension exists in the repository;
2. the `.Locale` extension is installed on the host;
3. the requested language subpath exists inside the installed extension and resolves through `/app/share/locale`.

A local OSTree repository can contain a complete `.Locale` ref while a user installation contains only a subset of language subpaths. `find -L /app/share/locale` is important because the normal locale directories can be symlinks into the mounted extension.

When this model is used, validate the installed extension rather than only `.po`, `.mo`, Meson or repository output.

A proven local repository workflow is:

```bash
rm -rf build-dir repo

flatpak-builder \
  --force-clean \
  --repo=repo \
  build-dir \
  <MANIFEST>.yml

flatpak --user remote-add \
  --if-not-exists \
  --no-gpg-verify \
  <LOCAL-REMOTE> \
  "file://$PWD/repo"

flatpak --user install \
  --reinstall \
  <LOCAL-REMOTE> \
  <APP_ID>
```

Then inspect catalog resolution through the normal gettext path:

```bash
flatpak --user run \
  --command=sh \
  <APP_ID> \
  -c 'find -L /app/share/locale -maxdepth 4 -type f -name "*.mo" -print 2>/dev/null | sort'
```

If a supported language is absent from an installed Locale extension, inspect/update the relevant extension subpaths before changing gettext source code.

## Standalone bundle evidence

Flatpak Builder defaults `separate-locales` to `true`, which creates a separate locale extension. For the suite's standalone GitHub `.flatpak` releases, newer evidence proved a different requirement: the one downloadable bundle is intended to contain the complete supported localization payload.

For that distribution contract, use:

```yaml
separate-locales: false
```

Delivery Hub v0.9.0 provided a complete evidence chain:

1. gettext integration generated Dutch, German, French, Spanish, Italian and Portuguese catalogs;
2. the Flatpak application payload contained all six catalogs directly under `/app/share/locale/<lang>/LC_MESSAGES/delivery-hub.mo`;
3. a user-level installation from the local repository passed runtime smoke in all six non-source languages;
4. the final standalone `Delivery-Hub-v0.9.0.flatpak` was built;
5. that exact file was installed with `flatpak --user install --reinstall`;
6. the installed standalone bundle passed normal runtime, app-icon, About/version and Dutch localization sanity checks;
7. the published GitHub asset digest matched the locally generated SHA-256.

This proves the standalone release boundary for that suite distribution model.

It does **not** prove that `.Locale` extensions are wrong, that `separate-locales: false` should be used for every Flatpak, or that repository distribution should abandon locale splitting.

## Explicit runtime smoke

Always close the application before switching language so a single existing process does not reuse the previous environment.

Example Dutch smoke:

```bash
flatpak run \
  --env=LANG=nl_NL.UTF-8 \
  --env=LANGUAGE=nl \
  <APP_ID>
```

Suite localization matrix:

```text
nl_NL.UTF-8 / nl
de_DE.UTF-8 / de
fr_FR.UTF-8 / fr
es_ES.UTF-8 / es
it_IT.UTF-8 / it
pt_PT.UTF-8 / pt
```

English remains the source/fallback language.

## PASS boundary

For repository-based locale splitting, localization delivery can be recorded as PASS when the applicable evidence shows:

1. `.mo` generation succeeds;
2. Meson registers catalogs for installation;
3. the `.Locale` extension is exported and installed;
4. every required language resolves through `/app/share/locale`;
5. explicit runtime smoke passes for the supported matrix;
6. user/source data remains untranslated.

For a standalone bundle with embedded locales, PASS requires:

1. every supported `.mo` catalog is present in the application payload;
2. packaged runtime locale smoke passes for the supported matrix;
3. the final standalone `.flatpak` is built;
4. that exact artifact is installed;
5. at least one explicit locale sanity check passes on the installed standalone artifact;
6. the final artifact checksum is recorded.

## Reusable engineering rule

Choose the localization QA path from the actual distribution model.

Do not conflate:

```text
source translation complete
≠ catalogs generated
≠ catalogs installed
≠ repository extension present
≠ language subpath installed
≠ standalone bundle contains locales
≠ installed application displays translation
```

Trace the complete boundary before changing gettext initialization.

## Important architecture boundary

Localization belongs in presentation. Machine-facing identifiers, enum values and domain contracts must not depend on translated strings.

When a legacy application already branches on stable human-readable service values, keep those values stable and localize them at the UI boundary until the domain contract can be converted safely to typed state.

## Evidence level

- Repository / `.Locale` evidence: Image Bench 0.9.0, Snippet Manager 0.9.0, Signature Designer 0.9.0
- Standalone embedded-locale evidence: Image Bench release work and Delivery Hub v0.9.0
- Reusable lesson: yes
- Universal Flatpak packaging rule: no

## Pitfalls

- Do not treat `.po` completeness as runtime proof.
- Do not infer a gettext code bug from incomplete installed locale data.
- Do not assume an exported `.Locale` ref means every language is installed.
- Do not assume a local repository PASS proves a downloadable standalone bundle.
- Do not assume a standalone embedded-locale PASS means repository distribution should disable locale splitting.
- Do not patch working gettext initialization until the packaged runtime boundary has been verified.
- Do not translate machine-facing state or use translated strings as behavior keys.

## Provenance

- Originating projects: Image Bench, Snippet Manager, Signature Designer, Delivery Hub
- Gate or phase: localization packaging and installed Flatpak runtime QA
- Evidence type: source/config inspection, generated catalog inspection, installed extension inspection, runtime smoke, standalone bundle installation
- Validation result: both documented distribution models have project evidence within their stated boundaries
