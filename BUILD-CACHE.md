# `build-cache` branch

This branch carries **only build artifacts** — the compiled `.olean` files of
this library together with the `.trace` / `.hash` sidecars Lake uses for its
up-to-date checks. Neither the sources nor the history of `main` are on this
branch, and nothing here should ever be merged into it.

## Why

A from-scratch `lake build` of this library takes tens of minutes. Checking
this branch out over a fresh clone lets Lake reuse the oleans instead.

## Use

```sh
git pull
git checkout origin/build-cache -- .lake
```

Then `lake build` should report `All targets up-to-date`.

Requirements:

* `lean-toolchain` must be the same (`leanprover/lean4:v4.34.0`);
* the mathlib pin in `lake-manifest.json` must resolve to the same revision —
  mathlib tag `v4.34.0`, rev `5ed2965` — the oleans record the content hashes of
  their dependencies;
* `.lake/packages/*` must be wired to that mathlib (the usual junction /
  symlink setup). This branch deliberately does **not** carry `.lake/packages`.

## What is not here

* `.lake/packages/**` — junctions into the shared mathlib checkout.
* `.lake/build/bin/**` — compiled executables (machine-specific, ~100 MB each).
* `ir/**/*.c` — the native-codegen C files are replaced by a one-line comment.
  Lake only requires every output recorded in the module's `.trace` to *exist*
  (it compares the sibling `.c.hash`, which is left intact), and importing
  oleans never reads the C. Building an `lean_exe` target regenerates them.
* `ir/**/*.setup.json` — codegen setup files, ~0.5 MB per module. They are not
  among the outputs recorded in any `.trace`, and removing one leaves
  `lake build --no-build` reporting up-to-date.

Everything on this branch comes from a plain `lake build` against mathlib tag
`v4.34.0` plus the stub pass over the `ir/**/*.c` files; nothing is hand-written.
`.gitattributes` pins `.lake/**` to `-text`, so the artifacts are stored
byte-for-byte whatever a checkout's `core.autocrlf` says.
