# Gettext localization must be verified at the packaged runtime boundary

## Context / problem

A translated GNOME/Rust application can have complete `.po` files and correctly generated `.mo` catalogs while an installed Flatpak does not immediately show the expected language.

The important boundary is not only source translation completeness, but the complete chain:

`PO → MO generation → Flatpak Locale extension → installed locale subpaths → runtime locale → gettext lookup → application UI`

## What happened?

Image Bench 0.9.0 was observed to remain in English after logging out of the desktop and starting the installed Flatpak while the desktop locale was Dutch. Other localized applications switched language correctly.

A later Snippet Manager 0.9.0 investigation reproduced a related packaging/runtime boundary: all six `.mo` catalogs were generated and registered by Meson, and the Flatpak `.Locale` extension existed in the OSTree repository, but the initial user installation contained only a subset of locale subpaths. Languages not present in the installed extension could not be selected successfully with `LANGUAGE=<lang>`.

Signature Designer 0.9.0 added a third evidence case with a clean local repository workflow. `flatpak-builder --repo=repo` exported both the application and its `.Locale` extension, a user-level install from that repository installed the Locale extension, all six supported catalogs resolved through `/app/share/locale`, and explicit runtime language smoke succeeded without changing gettext source code after packaging.

The investigation therefore distinguishes three separate states that are easy to conflate:

1. the Locale extension exists in the repository;
2. the Locale extension is installed on the host;
3. the requested language subpath exists inside the installed Locale extension and resolves through the normal gettext path.

## Evidence-driven investigation

The following checks were performed across localized GNOME/Rust Flatpak projects:

1. Host environment exposed the expected locale, for example:
   - `LANG=nl_NL.UTF-8`
   - `LANGUAGE=nl_NL:de_DE:en_US:en`

2. Meson correctly generated the supported `.mo` catalogs, including the suite baseline:
   - `nl`
   - `de`
   - `fr`
   - `es`
   - `it`
   - `pt`

3. `meson introspect <builddir> --installed` showed the catalogs registered for installation under `/app/share/locale/<lang>/LC_MESSAGES/`.

4. `flatpak-builder` produced a separate `.Locale` extension in the local OSTree repository. An `ostree --repo=<repo> refs` inspection exposed the Locale runtime ref.

5. A user-level app installation did not necessarily install every supported Locale subpath. Direct `flatpak info` inspection of the Locale extension showed the actually installed language directories.

6. `flatpak update --user --subpath= <APP_ID>.Locale` expanded an installed Locale extension from a subset of languages to the complete supported catalog set when needed.

7. Signature Designer 0.9.0 showed the standard Flatpak mount structure directly. The physical catalog files lived under `/app/share/runtime/locale/...`, while `/app/share/locale/<lang>` exposed symlinks into those mounted extension directories.

8. `find -L /app/share/locale -type f -name "*.mo"` followed those symlinks and confirmed that all six Signature Designer catalogs resolved through the normal gettext directory:

   ```text
   /app/share/locale/de/LC_MESSAGES/signature-designer.mo
   /app/share/locale/es/LC_MESSAGES/signature-designer.mo
   /app/share/locale/fr/LC_MESSAGES/signature-designer.mo
   /app/share/locale/it/LC_MESSAGES/signature-designer.mo
   /app/share/locale/nl/LC_MESSAGES/signature-designer.mo
   /app/share/locale/pt/LC_MESSAGES/signature-designer.mo
   ```

9. After the installed Locale boundary was proven, explicit runtime smoke using `LANG` and `LANGUAGE` switched the localized application correctly without a gettext code patch.

10. The same `.Locale` architecture is used by multiple suite applications. The release model therefore does not require disabling locale splitting merely to make a standalone `.flatpak` appear self-contained.

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

For Signature Designer, building and installing through a local repository made the application-plus-extension boundary explicit and reproducible. No source localization fix was required after the packaged runtime checks passed.

The evidence supports a packaged-runtime/Locale-subpath explanation for these observations. It does **not** establish that stale installation state, partial Locale installation, or repository installation is the universal cause or universal remedy for all localization failures.

## Proven local repository localization QA workflow

The following application-neutral workflow was proven with Signature Designer 0.9.0 on 2026-08-11. Replace the placeholders with the target project's values.

### 1. Build and export to a local repository

From the project root:

```bash
rm -rf build-dir repo

flatpak-builder \
  --force-clean \
  --repo=repo \
  build-dir \
  <MANIFEST>.yml
```

This validates the build and exports the application plus generated extensions into `repo/`.

### 2. Register the local repository

```bash
flatpak --user remote-delete <LOCAL-REMOTE> 2>/dev/null || true

flatpak --user remote-add \
  --no-gpg-verify \
  <LOCAL-REMOTE> \
  "$PWD/repo"
```

### 3. Install or reinstall from that repository

```bash
flatpak --user install \
  --reinstall \
  <LOCAL-REMOTE> \
  <APP_ID>
```

For a correctly localized Flatpak, installation should include the application and its `.Locale` extension when locale splitting applies.

### 4. Verify catalog resolution through the normal gettext path

Use `find -L` because `/app/share/locale/<lang>` can be a symlink into the mounted `.Locale` extension:

```bash
flatpak --user run \
  --command=sh \
  <APP_ID> \
  -c 'find -L /app/share/locale -maxdepth 4 -type f -name "*.mo" -print 2>/dev/null | sort; echo "---"; ls -la /app/share/locale 2>/dev/null'
```

This check is stronger than inspecting only `/app/share/runtime/locale`: it proves that the installed catalogs resolve through the directory gettext is expected to use.

### 5. Run explicit locale smoke tests

Always stop the application before switching language:

```bash
flatpak kill <APP_ID> 2>/dev/null || true
```

Then launch with explicit locale variables, for example Dutch:

```bash
flatpak run \
  --env=LANG=nl_NL.UTF-8 \
  --env=LANGUAGE=nl \
  <APP_ID>
```

Repeat for the supported language matrix, for example:

```text
nl_NL.UTF-8 / nl
de_DE.UTF-8 / de
fr_FR.UTF-8 / fr
es_ES.UTF-8 / es
it_IT.UTF-8 / it
pt_PT.UTF-8 / pt
```

English remains the source/fallback smoke:

```bash
flatpak kill <APP_ID> 2>/dev/null || true
flatpak run --env=LANG=en_US.UTF-8 --env=LANGUAGE=en <APP_ID>
```

### 6. Smoke-test scope

This is a localization delivery smoke, not a full application regression pass. Verify that:

- the application launches;
- primary navigation, menus, dialogs, common buttons, tooltips, and status messages use the requested language;
- no obvious English-only UI section remains unexpectedly;
- user/project data is not translated;
- identifiers, filenames, paths, MIME types, URLs, and application IDs remain unchanged;
- the translated UI does not introduce obvious broken layouts or missing glyphs.

### 7. PASS boundary

Localization delivery can be recorded as PASS when the applicable evidence shows:

1. the local Flatpak repository build succeeds;
2. the `.Locale` extension is exported/installed when applicable;
3. every supported `.mo` catalog resolves through `/app/share/locale`;
4. explicit runtime language smoke passes for the supported matrix;
5. English fallback works;
6. user/source data remains untranslated.

## Reusable engineering rule

When a GNOME/Rust Flatpak appears stuck in English despite valid translations:

1. Confirm the active `LANG`/`LANGUAGE`.
2. Confirm `.mo` generation in the Meson build.
3. Confirm Meson registers the catalogs for installation.
4. Confirm the Flatpak `.Locale` extension exists in the repository.
5. Confirm the `.Locale` extension is actually installed for the intended user/system scope.
6. Inspect which language subpaths are installed inside the Locale extension.
7. Follow `/app/share/locale` symlinks and confirm the requested catalog resolves through the normal gettext path.
8. If a supported language is absent, install/update the Locale extension's relevant subpath content through the Flatpak workflow.
9. Confirm the runtime provides the requested locale.
10. Perform explicit `LANG`/`LANGUAGE` runtime smoke.
11. Test gettext directly inside the packaged runtime when necessary.
12. Only after the packaged boundary is verified should application gettext initialization be changed.

For the suite's normal local workflow, user-level Flatpak installation is preferred unless a specific system-wide test is intentional.

## Important distinction: bundle vs repository

A single-file `.flatpak` application bundle and a repository containing the application plus its extensions are not the same validation boundary.

The presence of a `.Locale` OSTree ref proves that the extension was created/exported. It does not prove that every supported language subpath is installed on the target host or that the catalogs resolve through the normal gettext path.

Therefore, release QA for applications using the standard `.Locale` model must validate the **installed** Locale extension, not only the application bundle or source tree.

## Evidence level

- Project-specific observations: Image Bench 0.9.0, Snippet Manager 0.9.0, and Signature Designer 0.9.0
- Reusable lesson: yes
- Proven local repository workflow: yes, within the documented GNOME/Rust/Flatpak suite boundary
- Golden Standard packaging rule: recorded separately in `GOLDEN-STANDARD.md` and `PROJECT-CONFIGURATION.md`
- Proven universal root cause for all localization failures: no

## Pitfalls

- Do not treat `.po` completeness as proof that the shipped package behaves correctly.
- Do not infer a Rust gettext bug from an old or partial installed Flatpak state.
- Do not assume that a `.Locale` OSTree ref means every language is installed.
- Do not inspect only `/app/share/runtime/locale` and assume that proves gettext lookup; verify `/app/share/locale` resolution as well.
- Do not forget `find -L` when the normal locale directories are symlinks into the mounted extension.
- Do not assume `flatpak --user config --get languages` alone explains every explicit `LANGUAGE=<lang>` test; inspect the installed extension when behavior differs.
- Do not disable standard Flatpak locale splitting before tracing the packaged boundary.
- Do not use the host Meson to inspect or reconfigure a build directory created inside the GNOME SDK when the host Meson version differs.
- Do not patch working gettext initialization until the packaged runtime has been independently verified.
