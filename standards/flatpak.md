# Flatpak

Flatpak is part of the validation model, not only the packaging model.

Standards:

- use a GNOME SDK/runtime that matches the application's target;
- expose the Rust SDK extension explicitly when Rust tooling is needed;
- when Cargo runs inside a `flatpak run` GNOME SDK shell and needs registry/network access, explicitly share the network with `--share=network`;
- if Cargo repeatedly reports `Could not resolve host: index.crates.io` from that SDK shell, verify the Flatpak network share before debugging Cargo, crates.io, DNS, or project dependencies;
- verify the installed bundle, not only the build output;
- keep permissions minimal and justified;
- treat the installed Flatpak as the final proof of native integration;
- for localized applications using the standard `.Locale` extension model, verify the installed Locale extension and catalog resolution through `/app/share/locale`, not only generated `.po`/`.mo` files;
- prefer repository-based local localization QA when the goal is to validate the application and its generated extensions together;
- perform explicit `LANG`/`LANGUAGE` runtime smoke before changing gettext initialization that already passes build-time checks.

For example, a Clippy gate that may need crates.io access can be run from the matching GNOME SDK as:

```bash
flatpak run \
  --command=sh \
  --devel \
  --share=network \
  --filesystem="$PWD" \
  org.gnome.Sdk//50 \
  -c "export PATH=/usr/lib/sdk/rust-stable/bin:\$PATH; cd '$PWD'; cargo clippy --locked --all-targets --all-features -- -D warnings"
```

The same network rule applies to other Cargo commands executed through this SDK shell when they need to resolve or download registry content.

See `../learnings/gettext-runtime-locale-probe.md` for the evidence-backed localization delivery workflow.
