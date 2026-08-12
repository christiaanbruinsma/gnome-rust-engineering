# Manual SDK linker failures are not automatically product build failures

## Context / problem

Rust GNOME applications are often validated inside the target Flatpak SDK. A manually entered SDK shell can still differ from the actual `flatpak-builder` or GNOME Builder build pipeline, including linker selection and linker search behavior.

A link failure in that diagnostic shell is therefore evidence about that invocation first. It is not automatically evidence that the application, SDK, native dependencies, or release manifest are broken.

## What happened?

During Delivery Hub v0.9.0 release QA, `cargo test` was run manually through `flatpak run org.gnome.Sdk//50` with the Rust stable SDK extension on `PATH`.

Rust linking failed through `rust-lld` and reported that many GNOME libraries could not be found, including GTK4, libadwaita, GLib, GIO, Pango and Cairo libraries.

The native dependency boundary was then tested independently inside the same GNOME SDK:

```text
pkg-config --modversion gtk4 libadwaita-1 glib-2.0
```

returned valid versions, and `ldconfig -p` showed the relevant shared libraries under `/usr/lib/x86_64-linux-gnu/`.

That disproved the hypothesis that the GNOME SDK itself lacked the required native libraries.

A targeted linker falsification test was then run:

```bash
RUSTFLAGS="-C linker-features=-lld" cargo test
```

The full Rust test suite passed.

Finally, the actual release build was executed normally through `flatpak-builder` without the `RUSTFLAGS` workaround. It also passed.

## Cause

Within the observed environment, the failure was specific to the linker behavior selected by the manual Rust SDK invocation. The actual Flatpak build pipeline did not reproduce it.

The evidence does not establish a universal Rust, LLD, GNOME SDK or Flatpak defect.

## Proven solution

When a manual target-SDK Cargo invocation fails at the linker boundary:

1. verify the native dependencies independently with `pkg-config` and the SDK library inventory;
2. inspect which linker Rust is invoking;
3. use a targeted linker-feature override only as a falsification test when appropriate;
4. run the actual GNOME Builder or `flatpak-builder` pipeline before changing project configuration;
5. only make a persistent build change if the real product pipeline reproduces the problem.

On supported Rust targets, the following can be used as a diagnostic opt-out from Rust's LLD linker feature:

```bash
RUSTFLAGS="-C linker-features=-lld" cargo test
```

Do not persist that flag merely because it makes a diagnostic shell pass.

## Evidence

- Delivery Hub v0.9.0 manual GNOME SDK test invocation: LLD link failure.
- Same SDK: GTK4, libadwaita and GLib visible through `pkg-config` and `ldconfig`.
- `RUSTFLAGS="-C linker-features=-lld" cargo test`: PASS.
- Normal `flatpak-builder` release build without linker override: PASS.
- Rust compiler documentation defines `linker-features` and the `lld` feature opt-out syntax.

Official upstream reference:

- https://doc.rust-lang.org/rustc/codegen-options/index.html#linker-features

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: reusable diagnostic rule; the exact LLD behavior is environment-specific

## Pitfalls

- Do not interpret a manual SDK-shell failure as a product failure without reproducing it in the actual build pipeline.
- Do not interpret successful `pkg-config` alone as complete link proof.
- Do not permanently add `-lld` or another linker workaround without a product-pipeline failure that justifies it.
- Keep diagnostic environment, build pipeline and installed runtime as separate evidence boundaries.

## Provenance

- Originating project: Delivery Hub v0.9.0
- Gate or phase: Rust release QA after localization integration
- Evidence type: linker output, SDK dependency probes, executed Rust tests, successful Flatpak release build
- Validation result: root boundary isolated; product build PASS
