# Standalone Flatpak releases need their own localization boundary

## Context / problem

Flatpak's default packaging model splits translations into a separate `.Locale` extension. That model is well suited to repository-based distribution, but it is a different delivery boundary from publishing one downloadable `.flatpak` file as the complete release artifact.

The engineering question is therefore not simply whether locale splitting is valid. It is whether the chosen packaging model matches the distribution contract that is actually being shipped and tested.

## What happened?

Image Bench release work and Delivery Hub v0.9.0 both exposed the standalone-bundle boundary.

For Delivery Hub v0.9.0, the release manifests used:

```yaml
separate-locales: false
```

After the localization layer was added, the Flatpak build contained all six non-source catalogs directly under the application payload:

```text
/app/share/locale/de/LC_MESSAGES/delivery-hub.mo
/app/share/locale/es/LC_MESSAGES/delivery-hub.mo
/app/share/locale/fr/LC_MESSAGES/delivery-hub.mo
/app/share/locale/it/LC_MESSAGES/delivery-hub.mo
/app/share/locale/nl/LC_MESSAGES/delivery-hub.mo
/app/share/locale/pt/LC_MESSAGES/delivery-hub.mo
```

The application was then installed from the local repository and runtime-smoked in Dutch, German, French, Spanish, Italian and Portuguese. All six passed.

A final standalone bundle was built, installed with `flatpak --user install --reinstall ./Delivery-Hub-v0.9.0.flatpak`, and the installed bundle passed normal runtime, application-icon, About/version and Dutch localization sanity checks. The published GitHub asset digest matched the locally generated SHA-256.

## Cause

The earlier repository guidance treated the default `.Locale` extension architecture and a standalone GitHub bundle as if they were the same release boundary. They are not.

`flatpak-builder` defaults `separate-locales` to `true`, which separates locale data into an extension. Setting it to `false` keeps locale files in the main application payload.

Neither model is universally superior. The correct choice depends on the distribution contract.

## Proven solution

For this suite's standalone GitHub `.flatpak` releases, where the downloadable file is intended to be the complete installable release artifact, embed the supported gettext catalogs in the application payload:

```yaml
separate-locales: false
```

Then validate the exact distributed artifact:

1. confirm all expected `.mo` files are present in the built application payload;
2. install from the local repository and run the complete supported locale matrix;
3. build the standalone `.flatpak` bundle;
4. install or reinstall that exact standalone file;
5. perform at least one explicit locale sanity check on the installed standalone artifact;
6. record and publish its SHA-256.

Repository-based or Flathub-style distribution may intentionally retain the default `.Locale` extension model. This learning does not require disabling locale splitting for every Flatpak.

## Evidence

- Flatpak Builder documentation: `separate-locales` defaults to `true`; `false` disables separation into a locale extension.
- Delivery Hub v0.9.0: six embedded catalogs present, six-language repository runtime smoke PASS, exact standalone bundle install PASS, Dutch standalone-bundle localization sanity PASS.
- Image Bench release work independently demonstrated the same standalone-localization concern.

Official upstream reference:

- https://docs.flatpak.org/en/latest/flatpak-builder-command-reference.html

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: yes, for the suite's standalone GitHub `.flatpak` distribution model

## Pitfalls

- Do not infer that `.Locale` extensions are wrong; they are Flatpak's default model.
- Do not infer that `separate-locales: false` is a universal Flatpak requirement.
- Do not call localization PASS merely because `.po` or `.mo` files exist.
- Do not validate only a local repository when the actual release contract is a standalone downloadable bundle.
- Do not validate only the standalone bundle if the product is actually distributed through a repository with extensions.
- Keep the packaging rule tied to the real distribution boundary.

## Provenance

- Originating projects: Image Bench release work; Delivery Hub v0.9.0
- Gate or phase: localization packaging and release QA
- Evidence type: manifest inspection, built payload inspection, installed Flatpak runtime smoke, published artifact checksum
- Validation result: PASS within the documented standalone GitHub bundle boundary
