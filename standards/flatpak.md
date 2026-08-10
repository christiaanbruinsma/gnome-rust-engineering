# Flatpak

Flatpak is part of the validation model, not only the packaging model.

Standards:

- use a GNOME SDK/runtime that matches the application's target;
- expose the Rust SDK extension explicitly when Rust tooling is needed;
- verify the installed bundle, not only the build output;
- keep permissions minimal and justified;
- treat the installed Flatpak as the final proof of native integration.
