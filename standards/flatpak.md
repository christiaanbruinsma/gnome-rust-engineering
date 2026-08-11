# Flatpak

Flatpak is part of the validation model, not only the packaging model.

Standards:

- use a GNOME SDK/runtime that matches the application's target;
- expose the Rust SDK extension explicitly when Rust tooling is needed;
- verify the installed bundle, not only the build output;
- keep permissions minimal and justified;
- treat the installed Flatpak as the final proof of native integration;
- for localized applications using the standard `.Locale` extension model, verify the installed Locale extension and catalog resolution through `/app/share/locale`, not only generated `.po`/`.mo` files;
- prefer repository-based local localization QA when the goal is to validate the application and its generated extensions together;
- perform explicit `LANG`/`LANGUAGE` runtime smoke before changing gettext initialization that already passes build-time checks.

See `../learnings/gettext-runtime-locale-probe.md` for the evidence-backed localization delivery workflow.
