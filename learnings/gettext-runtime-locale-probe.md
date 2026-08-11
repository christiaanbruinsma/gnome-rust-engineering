# Gettext localization must be verified at the packaged runtime boundary

## Context / problem

A translated GNOME/Rust application can have complete `.po` files and correctly generated `.mo` catalogs while an installed Flatpak does not immediately show the expected language.

The important boundary is not only source translation completeness, but the complete chain:

`PO → MO generation → Flatpak Locale extension → runtime locale → gettext lookup → application UI`

## What happened?

Image Bench 0.9.0 was observed to remain in English after logging out of the desktop and starting the installed Flatpak while the desktop locale was Dutch. Other localized applications switched language correctly.

The investigation initially considered gettext path configuration, Flatpak Locale packaging, locale availability, and Rust `gettext-rs` initialization.

## Evidence-driven investigation

The following checks were performed against the installed Image Bench Flatpak:

1. Host environment exposed the expected locale:
   - `LANG=nl_NL.UTF-8`
   - `LANGUAGE=nl_NL:de_DE:en_US:en`

2. Meson correctly generated all six `.mo` catalogs:
   - `nl`
   - `de`
   - `fr`
   - `es`
   - `it`
   - `pt`

3. The installed Flatpak contained all six catalogs in its Locale extension, for example:
   `.../runtime/locale/nl/share/nl/LC_MESSAGES/image-bench.mo`

4. The compiled Image Bench binary contained `/app/share/locale`, confirming that the expected Flatpak locale directory was embedded in the binary.

5. The Flatpak runtime contained `nl_NL.utf8`, so the requested locale was available to glibc.

6. Direct gettext lookup inside the Flatpak worked when the locale directory was explicitly supplied:

   ```text
   TEXTDOMAINDIR=/app/share/locale gettext -d image-bench "Add Images"
   → Afbeeldingen toevoegen
   ```

7. No source-code i18n change was required.

## Resolution

A clean Flatpak rebuild and installation resolved the observed language-switching problem:

```bash
flatpak-builder --user --install --force-clean build-flatpak io.github.christiaanbruinsma.ImageBench.yml
```

After the clean rebuild/install and a fresh desktop login, Image Bench switched correctly between tested locales:

- English ✅
- Dutch ✅
- German ✅

The remaining supported locales (`fr`, `es`, `it`, `pt`) were present in the same generated and installed catalog set, but were not individually runtime-tested during this incident.

The evidence therefore supports this conclusion:

> The failure was associated with the previously installed/build state. A clean Flatpak rebuild and installation restored correct runtime localization without a source-code patch.

Do not claim that a stale installation is the universal cause of localization failures. The correct reusable lesson is that the packaged runtime must be rebuilt/reinstalled and tested before changing working i18n code.

## Reusable engineering rule

When a GNOME/Rust Flatpak appears stuck in English despite valid translations:

1. Confirm the active `LANG`/`LANGUAGE`.
2. Confirm `.mo` generation in the Meson build.
3. Confirm the catalogs exist in the installed Flatpak Locale extension.
4. Confirm the binary uses the intended Flatpak locale directory.
5. Confirm the runtime provides the requested locale.
6. Test gettext directly inside the packaged runtime.
7. Before changing source i18n code, perform a clean Flatpak rebuild/install and repeat the runtime test.

This keeps source-code changes evidence-driven and prevents fixing already-correct gettext configuration because of stale build/install state.

## Evidence level

- Project-specific observation: Image Bench 0.9.0
- Reusable lesson: yes
- Proven universal root cause for all localization failures: no

## Pitfalls

- Do not treat `.po` completeness as proof that the shipped package behaves correctly.
- Do not infer a Rust gettext bug from an old installed Flatpak.
- Do not use the host Meson to inspect or reconfigure a build directory created inside the GNOME SDK when the host Meson version differs.
- Do not patch working gettext initialization until the packaged runtime has been independently verified.