# renovate-repro-mise-cargo-pin

Minimal reproduction for the Renovate `mise` manager pinning `cargo:` backend tools with Cargo requirement syntax (`=x.y.z`), which mise cannot resolve.

## Current behavior

Renovate proposes this pin for `mise.toml`:

```diff
 [tools]
-"cargo:flip-link" = "0.1.12"
+"cargo:flip-link" = "=0.1.12"
```

The `mise` manager maps `cargo:` tools to the `crate` datasource, whose default versioning is `cargo`. The `cargo` versioning pins with Cargo dependency requirement syntax:

- `lib/modules/manager/mise/backends.ts` — `createCargoToolConfig()` returns `{ packageName, datasource: crate }` and sets no `versioning`
- `lib/modules/datasource/crate/index.ts` — `override defaultVersioning = cargoVersioning.id`
- `lib/modules/versioning/cargo/index.ts` — `getPinnedValue(newVersion)` returns `` `=${newVersion}` ``

The version field of a mise tool is not a Cargo dependency requirement, and mise does not support `=`-prefixed semver ranges there. Once the pin lands, mise can no longer resolve the tool (mise 2026.7.16):

```console
$ mise ls --current
mise WARN  semver range "=0.1.12" is not supported for cargo:flip-link (source: mise.toml); use a concrete version or version prefix
cargo:flip-link  =0.1.12 (missing)  mise.toml  =0.1.12

$ mise latest "cargo:flip-link@=0.1.12"
mise WARN  semver range "=0.1.12" is not supported for cargo:flip-link (source: command argument); use a concrete version or version prefix
```

The second command prints no version: `=0.1.12` is treated as a single opaque version string and matches nothing in the remote version list.

## Expected behavior

No pin PR is created, because `0.1.12` is already an exact version for mise. Later updates rewrite the plain version string:

```diff
 [tools]
-"cargo:flip-link" = "0.1.12"
+"cargo:flip-link" = "0.1.13"
```

## Link to the Renovate issue or Discussion

To be added.
