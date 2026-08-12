# A built Flatpak dependency must also be discoverable by later build steps

## Context / problem

A native dependency can be successfully built inside a Flatpak module while a later Rust/Cargo build still reports that the dependency cannot be found through `pkg-config`.

The relevant engineering boundary is therefore not only whether the dependency exists somewhere in the build environment. Its development metadata and libraries must be installed into the prefix and search layout that later build steps actually consume.

## What happened?

During PDF Workbench Rust/Flatpak work, the Rust Poppler binding failed because `pkg-config` could not find `poppler-glib.pc`.

The investigation established that Poppler itself was present in the Flatpak build context, but its discoverability depended on the installation layout used by the Poppler/CMake module.

The release manifest was aligned to install libraries under `lib` using:

```text
-DCMAKE_INSTALL_LIBDIR=lib
```

A later runtime/build inspection proved:

```text
/app/lib/pkgconfig/poppler-glib.pc
```

was present, reported Poppler GLib version `26.08.0`, and exposed its expected include flags through `pkg-config`.

This distinguished "Poppler was built" from "the later Rust build can discover Poppler through the expected pkg-config path."

## Cause

Within the observed project, dependency availability and dependency discoverability were separate boundaries. A non-matching install layout could leave the dependency outside the pkg-config search layout expected by the later build step.

The evidence does not establish that `lib` is universally correct for every Flatpak dependency or architecture. It establishes that the producer module and consumer build must agree on the installation/search contract.

## Proven solution

When a later Flatpak module reports a missing native dependency that an earlier module supposedly built:

1. inspect the actual installed `.pc` file location inside the build/runtime prefix;
2. inspect `pkg-config --modversion`, `--cflags` and search behavior in the same Flatpak boundary;
3. compare the producer module's install directory with the consumer's expected pkg-config paths;
4. align the install layout deliberately, for example through the dependency's supported CMake/Meson install-dir option;
5. rerun the actual Flatpak build before changing Rust binding code.

For the observed PDF Workbench/Poppler case, `-DCMAKE_INSTALL_LIBDIR=lib` produced the expected `/app/lib/pkgconfig/poppler-glib.pc` boundary.

## Evidence

- PDF Workbench Rust build initially failed because `poppler-glib.pc` was not discoverable.
- Poppler was built as part of the Flatpak dependency chain.
- Manifest/build configuration used `-DCMAKE_INSTALL_LIBDIR=lib`.
- `/app/lib/pkgconfig/poppler-glib.pc` was later directly verified.
- `pkg-config` reported Poppler GLib `26.08.0` and valid include flags from that installed metadata.

## Reusability

- Project-specific: no
- Possibly reusable: yes
- Proven suite-wide: reusable diagnostic boundary; exact install directories remain dependency/platform-specific

## Pitfalls

- "Dependency built" does not prove "dependency discoverable by later modules."
- Do not inspect only the host filesystem when the consumer builds inside Flatpak.
- Do not patch Rust FFI or binding code before proving the native pkg-config boundary.
- Do not hardcode `lib` or `lib64` as a universal rule; inspect the target SDK and producer/consumer contract.
- Keep build-time development metadata distinct from final runtime-library presence.

## Provenance

- Originating project: PDF Workbench
- Gate or phase: Rust migration / Flatpak native dependency integration
- Evidence type: build error, manifest configuration, installed `.pc` inspection, pkg-config output
- Validation result: native Poppler pkg-config metadata proven under `/app/lib/pkgconfig`
