# Gettext localization must be verified at the packaged runtime boundary

## Context / problem

A translated GNOME/Rust application can have complete `.po` files and correctly generated `.mo` catalogs while an installed Flatpak does not immediately show the expected language.

The important boundary is not only source translation completeness, but the complete chain:

`PO → MO generation → Flatpak Locale extension → installed locale subpaths → runtime locale → gettext lookup → application UI`

## What happened?

Image Bench 0.9.0 was observed to remain in English after logging out of the desktop and starting the installed Flatpak while the desktop locale was Dutch. Other localized applications switched language correctly.

A later Snippet Manager 0.9.0 investigation reproduced a related packaging/runtime boundary: all six `.mo` catalogs were generated and registered by Meson, and the Flatpak `.Locale` extension existed in the OSTree repository, but the initial user installation contained only a subset of locale subpaths. Languages not present in the installed extension could not be selected successfully with `LANGUAGE=<lang>`.

The investigation therefore distinguished three separate states that are easy to conflate:

1. the Locale extension exists in the repository;
2. the Locale extension is installed on the host;
3. the requested language subpath exists inside the installed Locale extension.

## Evidence-driven investigation

The following checks were performed across localized GNOME/Rust Flatpak projects:

1. Host environment exposed the expected locale, for example:
   - `LANG=nl_NL.UTF-8`
   - `LANGUAGE=nl_NL:de_DE:en_US:en`

2. Meson correctly generated all six `.mo` catalogs:
   - `nl`
   - `de`
   - `fr`
   - `es`
   - `it`
   - `pt`

3. `meson introspect <builddir> --installed` showed the catalogs registered for installation under `/app/share/locale/<lang>/LC_MESSAGES/`.

4. `flatpak-builder` produced a separate `.Locale` extension in the local OSTree repository. An `ostree --repo=<repo> refs` inspection exposed the Locale runtime ref.

5. A user-level app installation did not necessarily install every supported Locale subpath. Direct `flatpak info` inspection of the Locale extension showed the actually installed language directories.

6. `flatpak update --user --subpath= <APP_ID>.Locale` expanded the installed Locale extension from a subset of languages to the complete supported catalog set.

7. After the full Locale extension was installed, explicit runtime smoke using `LANG` and `LANGUAGE` succeeded for Dutch, German, French, Spanish, Italian, and Portuguese without changing application source code.

8. The same `.Locale` architecture is used by the suite's Data Inspector and Image Bench releases. The release model therefore does not require disabling locale splitting merely to make a standalone `.flatpak` appear self-contained.

## Resolution

For the observed Image Bench issue, a clean Flatpak rebuild and installation resolved the language-switching problem without source changes:

```bash
flatpak-builder --user --install --force-clean build-flatpak io.github.christiaanbruinsma.ImageBench.yml
```

For the Snippet Manager investigation, the source/build/install chain was already correct. The missing languages were caused by the installed Locale extension containing only selected language subpaths. Updating the user-level Locale extension restored the complete matrix:

```bash
flatpak update --user --subpath= io.github.christiaanbruinsma.SnippetManager.Locale
```

After that update, runtime smoke passed for:

- English fallback ✅
- Dutch ✅
- German ✅
- French ✅
- Spanish ✅
- Italian ✅
- Portuguese ✅

The evidence supports a packaged-runtime/Locale-subpath explanation for these observations. It does **not** establish that stale installation state or partial Locale installation is the universal cause of all localization failures.

## Reusable engineering rule

When a GNOME/Rust Flatpak appears stuck in English despite valid translations:

1. Confirm the active `LANG`/`LANGUAGE`.
2. Confirm `.mo` generation in the Meson build.
3. Confirm Meson registers the catalogs for installation.
4. Confirm the Flatpak `.Locale` extension exists in the repository.
5. Confirm the `.Locale` extension is actually installed for the intended user/system scope.
6. Inspect which language subpaths are installed inside the Locale extension.
7. If a supported language is absent, install/update the Locale extension's relevant subpath content through the Flatpak workflow.
8. Confirm the runtime provides the requested locale.
9. Test gettext directly inside the packaged runtime when necessary.
10. Only after the packaged boundary is verified should application gettext initialization be changed.

For the suite's normal local workflow, user-level Flatpak installation is preferred unless a specific system-wide test is intentional.

## Important distinction: bundle vs repository

A single-file `.flatpak` application bundle and a repository containing the application plus its extensions are not the same validation boundary.

The presence of a `.Locale` OSTree ref proves that the extension was created/exported. It does not prove that every supported language subpath is installed on the target host.

Therefore, release QA for applications using the standard `.Locale` model must validate the **installed** Locale extension, not only the application bundle or source tree.

## Evidence level

- Project-specific observations: Image Bench 0.9.0 and Snippet Manager 0.9.0
- Reusable lesson: yes
- Golden Standard packaging rule: recorded separately in `GOLDEN-STANDARD.md` and `PROJECT-CONFIGURATION.md`
- Proven universal root cause for all localization failures: no

## Pitfalls

- Do not treat `.po` completeness as proof that the shipped package behaves correctly.
- Do not infer a Rust gettext bug from an old or partial installed Flatpak state.
- Do not assume that a `.Locale` OSTree ref means every language is installed.
- Do not assume `flatpak --user config --get languages` alone explains every explicit `LANGUAGE=<lang>` test; inspect the installed extension when behavior differs.
- Do not disable standard Flatpak locale splitting before tracing the packaged boundary.
- Do not use the host Meson to inspect or reconfigure a build directory created inside the GNOME SDK when the host Meson version differs.
- Do not patch working gettext initialization until the packaged runtime has been independently verified.
