# GNOME Builder + Rust Project Configuration

This is the recovery guide for starting, rebuilding, or auditing a native GNOME application when no prior project knowledge can be assumed.

The goal is reproducibility: a developer or assistant should be able to reconstruct the expected Rust, GTK4, libadwaita, Meson, Flatpak, GNOME Builder, and QA setup from this document without copying a specific application's identity.

Data Inspector v0.9.0 is the approved reference baseline for this configuration. Application-specific names, IDs, permissions, crates, and feature code are placeholders, not suite-wide constants.

## 1. Technology baseline

Use this baseline for standalone native GNOME applications unless the project has a documented reason to differ:

- Rust (current stable toolchain supported by the target SDK)
- Rust 2024 edition for new projects
- GTK4
- libadwaita
- Meson as the GNOME project/build/install layer
- Cargo as the Rust dependency, compile, test, Clippy, and formatting layer
- Flatpak with the target GNOME Platform and SDK
- `org.freedesktop.Sdk.Extension.rust-stable`
- GNU gettext / Meson i18n when localization is applicable

GNOME Shell extensions remain a separate GJS track.

## 2. Identity model

Define one base application ID and derive the development identity from it.

Conceptually:

```text
base ID:        <APP_ID>
development ID: <APP_ID>.Devel
```

The active identity must consistently drive:

- GTK/GApplication application ID
- desktop filename and desktop ID
- AppStream/MetaInfo component ID
- AppStream launchable desktop ID
- application icon installation name
- symbolic application icon installation name
- About/application identity
- Flatpak manifest identity

Do not duplicate the UI or resources merely to create a `.Devel` build.

## 3. Meson profile model

Keep release and development builds in one source tree.

A proven pattern is:

```meson
base_id = '<APP_ID>'
is_devel = get_option('profile') == 'development'
application_id = is_devel ? base_id + '.Devel' : base_id
```

Use Meson configuration/template substitution so generated desktop and MetaInfo files receive the active identity.

The normal GNOME project hierarchy should keep platform concerns explicit, for example:

```text
project/
├── Cargo.toml
├── Cargo.lock
├── meson.build
├── meson.options
├── <APP_ID>.yml
├── <APP_ID>.Devel.yml
├── data/
├── po/
├── src/
├── tests/              # when applicable
└── docs/
```

Exact module layout is application-specific.

### Profile values are project-defined

Do not assume that a release build literally uses a profile named `release`.

Before issuing or reusing a profile-specific command, inspect `meson.options` and use its exact allowed values. Delivery Hub v0.9.0, for example, defined only:

```meson
choices: ['default', 'development']
value: 'default'
```

Its release build therefore correctly used:

```text
-Dprofile=default
```

Treat `meson.options` as the source of truth, including when a handover or command copied from another project says otherwise.

## 4. Flatpak Rust toolchain

Both release and development manifests should declare the Rust SDK extension:

```yaml
sdk-extensions:
  - org.freedesktop.Sdk.Extension.rust-stable

build-options:
  append-path: /usr/lib/sdk/rust-stable/bin
```

This is not cosmetic. The extension can be installed while Cargo/Rust is still unavailable to the Meson build unless the SDK path is exposed correctly.

Use the target GNOME Platform/SDK version selected for the project. Do not blindly copy a historical runtime version from another app.

Runtime permissions must be minimal and justified by actual functionality.

## 5. GNOME Builder workflow

Open the development Flatpak manifest in GNOME Builder for normal development.

Default workflow:

1. **Build** using GNOME Builder.
2. Run the app in the Builder/Flatpak environment.
3. Inspect runtime output when behavior is in question.
4. Apply the smallest justified patch.
5. Build/run again.
6. Run the applicable QA commands before declaring the change stable.

Use normal **Build** by default.

Use **Clean/Rebuild** only when:

- toolchain configuration changed;
- Meson/Flatpak configuration changed in a way that invalidates artifacts;
- feature/configuration boundaries changed and a clean state is required; or
- stale build artifacts have been demonstrated.

Do not use Clean/Rebuild as a generic debugging ritual.

## 6. GNOME Builder custom commands

Create these project commands in GNOME Builder. The important part is not only the shell command: **Working Directory, Locality, and Use Subshell are part of the configuration contract.**

### Clippy (strict)

```text
Name:              Clippy (strict)
Shell command:     cargo clippy --all-targets --all-features -- -D warnings
Working directory: $SRCDIR/
Locality:          Build Pipeline
Use Subshell:      Off
```

For release gates with an existing reviewed `Cargo.lock`, prefer the locked form:

```text
cargo clippy --locked --all-targets --all-features -- -D warnings
```

### Cargo Tests

```text
Name:              Cargo Tests
Shell command:     cargo test --all-targets --all-features
Working directory: $SRCDIR/
Locality:          Build Pipeline
Use Subshell:      Off
```

For the release gate, use the project's proven locked test command when `Cargo.lock` is established, for example:

```text
cargo test --locked
```

Do not invent flags merely for uniformity; the test command must match the project's actual targets/features and release gate.

### AppStream Validate

```text
Name:              AppStream Validate
Shell command:     appstreamcli validate --explain data/<GENERATED_DEVEL_METAINFO>.metainfo.xml
Working directory: $BUILDDIR/
Locality:          Build Pipeline
Use Subshell:      Off
```

The filename must be the **generated** MetaInfo file for the active build identity. Do not copy Data Inspector's literal ID into another project.

### Formatting

`cargo fmt --check` does not require GTK system libraries and can be run from the source directory. It may be added as a Builder command if desired, but the required release gate is the result, not a specific UI entry.

## 7. Why Build Pipeline locality matters

Rust GNOME projects depend on native GTK/libadwaita/GIO libraries supplied by the target SDK.

A host-shell Cargo invocation can fail because host `pkg-config` cannot find those libraries even though the Builder project itself is valid. Conversely, a host toolchain can accidentally validate against a different environment than the shipped application.

Therefore:

- GNOME/native dependency checks should run in the target Builder/Flatpak SDK context.
- Builder custom commands that depend on that environment use `Locality: Build Pipeline`.
- Host failures are evidence about the host environment, not automatically evidence that product code is broken.
- SDK success is not automatically a complete runtime or installed-Flatpak PASS.

A manually entered target-SDK shell is also not identical to the actual Builder or `flatpak-builder` pipeline. If a manual Cargo invocation fails at the linker boundary while SDK dependency probes succeed, isolate the linker behavior and reproduce the issue in the real product pipeline before changing persistent project configuration.

## 8. Cargo and Meson responsibilities

Keep the boundary explicit.

**Cargo owns:**

- Rust dependencies
- Rust compilation
- `cargo test`
- Clippy
- rustfmt / `cargo fmt`
- `Cargo.lock`

**Meson owns:**

- GNOME project configuration
- profiles/build identity
- generated desktop/AppStream files
- resource/data installation
- gettext integration
- installation layout
- invoking/integrating the Rust build in the GNOME project

Cargo does not replace Meson in this project model.

## 9. Desktop and AppStream generation

Generate desktop and MetaInfo files from the active application identity rather than maintaining mismatched copies.

The installed release must agree on:

```text
application ID
desktop ID
MetaInfo component ID
launchable desktop ID
main app icon name
symbolic app icon name
```

The generated MetaInfo should install to:

```text
/app/share/metainfo/<APP_ID>.metainfo.xml
```

Validate the generated file, not only the `.in` template.

## 10. Icon configuration

Normal UI actions use semantic GTK/Freedesktop/GNOME `icon-name` values.

Rules:

- verify the semantic icon name; never assume it from memory;
- do not hardcode the user's icon theme;
- do not ship ordinary action icons as arbitrary SVG/PNG replacements;
- use `hicolor` fallbacks only for required semantic icons that are not reliably available;
- test resolution in the application runtime and again in the installed Flatpak.

Suite sidebar toggles:

```text
left sidebar:              panel-left-symbolic
right sidebar / Inspector: panel-right-symbolic
```

Application identity icons are separate assets and are installed under the active application ID.

## 11. Localization configuration

English is the source/fallback language.

Suite order:

1. English
2. Dutch
3. German
4. French
5. Spanish
6. Italian
7. Portuguese

Keep gettext initialization behind a small i18n boundary. Localize runtime UI and desktop/AppStream metadata where applicable. Never translate inspected user data, file contents, identifiers, or other source values.

Machine-facing behavior MUST NOT depend on translated strings. Prefer typed domain state and localize only at the UI/presentation boundary. When legacy logic already depends on stable human-readable service values, preserve those values during localization and map them to translated presentation separately.

A compile-time translation catalog is not enough: perform packaged/runtime locale smoke.

### 11.1 Choose locale packaging from the distribution model

Flatpak Builder defaults to splitting locale data into a `.Locale` extension.

That model SHOULD remain in place for repository-based distribution such as Flathub or another OSTree repository where extensions are part of the installation contract.

For this suite's standalone GitHub `.flatpak` releases, where a single downloaded file is intended to be the complete installable release artifact, use:

```yaml
separate-locales: false
```

This embeds the supported catalogs in the application payload instead of relying on a separate Locale extension.

Do not treat either model as universal. The manifest must match the actual distribution contract.

### 11.2 Repository / `.Locale` validation

When locale splitting is enabled, validate the full chain:

```text
PO → MO generation → Meson install registration
  → Flatpak Locale extension → installed locale subpaths
  → runtime locale → gettext lookup → translated UI
```

A `.Locale` extension can exist in a local OSTree repository while only some language subpaths are installed on the host. Therefore:

- a generated `.mo` file proves only MO generation;
- a Meson installed-file entry proves only install registration;
- an OSTree `.Locale` ref proves only extension creation/export;
- installed extension inspection is required to prove which languages are available;
- runtime smoke must use the installed extension, not merely the source tree or build directory.

When needed, follow `/app/share/locale` symlinks with `find -L` to prove normal gettext resolution.

### 11.3 Standalone-bundle validation

When `separate-locales: false` is used, verify all expected catalogs in the built application payload, for example:

```text
/app/share/locale/<lang>/LC_MESSAGES/<GETTEXT_DOMAIN>.mo
```

Then:

1. install from the local repository and run the supported locale matrix;
2. build the final standalone `.flatpak`;
3. install or reinstall that exact standalone file;
4. perform at least one explicit locale sanity check on the installed release artifact;
5. record and publish the final SHA-256.

Delivery Hub v0.9.0 proved this standalone release path with embedded Dutch, German, French, Spanish, Italian and Portuguese catalogs, six-language packaged runtime smoke, and a final installed standalone-bundle locale sanity check.

### 11.4 Runtime locale smoke

Always stop the application before changing language so an existing process does not reuse the previous environment.

Example:

```bash
flatpak run \
  --env=LANG=nl_NL.UTF-8 \
  --env=LANGUAGE=nl \
  <APP_ID>
```

Repeat for the supported language matrix:

```text
nl_NL.UTF-8 / nl
de_DE.UTF-8 / de
fr_FR.UTF-8 / fr
es_ES.UTF-8 / es
it_IT.UTF-8 / it
pt_PT.UTF-8 / pt
```

English remains the source/fallback smoke.

## 12. Native build dependency discoverability

A native dependency built by an earlier Flatpak module is not automatically discoverable by a later Rust/Cargo module.

When `pkg-config` reports a missing library that should already have been built:

1. inspect the actual installed `.pc` file location inside the same Flatpak prefix;
2. run `pkg-config` in that same build boundary;
3. compare the producer module's install directory with the consumer's search paths;
4. align install layout through the dependency's supported Meson/CMake option when required;
5. rebuild the real Flatpak pipeline before changing Rust bindings.

Do not hardcode `lib` or `lib64` as a universal solution. PDF Workbench proved only that its Poppler producer/consumer contract needed `poppler-glib.pc` discoverable under the path used by the target build, ultimately verified at `/app/lib/pkgconfig/poppler-glib.pc`.

## 13. Required validation ladder

Do not collapse these into one claim:

```text
source/static PASS
    ↓
build PASS
    ↓
tests PASS
    ↓
runtime smoke PASS
    ↓
generated metadata PASS
    ↓
exported package PASS
    ↓
installed Flatpak PASS
```

A lower gate never proves a higher gate.

Minimum applicable release gates:

1. source hygiene / legacy-name scan scoped to the source tree;
2. normal GNOME Builder Build;
3. `cargo fmt --check`;
4. strict Clippy;
5. automated tests;
6. runtime smoke of core workflows;
7. semantic icon audit;
8. localization smoke;
9. generated AppStream validation;
10. export stable-ID Flatpak;
11. install that exported Flatpak;
12. launch and verify host integration;
13. feature-specific robustness/edge-case tests;
14. verify release metadata, license, source inventory, version, and assets;
15. only then create/finalize the release tag.

When tests target an ordered validation pipeline, design fixtures so earlier guards remain satisfied unless guard precedence itself is the behavior under test. A test name or downstream assertion does not prove that the intended branch executed.

## 14. Evidence language

When recording results:

- **FACT** means demonstrated by code, output, logs, tests, official validation tooling, or installed package behavior.
- **UNKNOWN** means not yet demonstrated.
- **HYPOTHESIS** means a proposed explanation with a falsifiable test.
- **PASS** means the named gate was actually executed and passed.
- **NOT TESTED** means applicable but not executed.
- **N/A** means genuinely not applicable.

Remember:

```text
authored tests ≠ executed tests
compile PASS ≠ runtime PASS
runtime PASS ≠ installed Flatpak PASS
reference implementation ≠ every implementation detail is a standard
```

## 15. Recovery checklist for a forgotten/new project

If a developer or assistant inherits a project with no context, inspect in this order:

1. Read `README.md`, project handover/`docs/`, and `Cargo.toml`.
2. Identify the base app ID and development/release profiles.
3. Inspect `meson.build`, `meson.options`, and `data/meson.build`.
4. Inspect both Flatpak manifests.
5. Confirm Rust SDK extension and `append-path`.
6. Confirm GTK4/libadwaita crate versions are compatible.
7. Confirm desktop/AppStream/icon identities are generated consistently.
8. Open the development manifest in GNOME Builder.
9. Confirm/create the Builder custom commands from section 6.
10. Run normal Build before changing code.
11. Run the app and capture runtime output.
12. Run formatting, strict Clippy, tests, and AppStream validation.
13. Inspect semantic icon resolution and localization.
14. Export/install the stable-ID Flatpak before release approval.
15. Record new project-specific lessons in `project/docs/LEARNINGS.md`.
16. Promote a lesson centrally only when its evidence supports reuse.

If a step cannot be verified, mark it UNKNOWN or NOT TESTED rather than guessing.

## 16. Reference baseline

Data Inspector v0.9.0 is the approved Golden Standard reference baseline as of 2026-08-10.

Use its public engineering documentation as the canonical concrete example, while keeping this guide application-neutral. Newer evidence may supersede individual details; update this repository when that happens.
