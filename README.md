# ikenga-registry

[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)

> The static JSON registry of published Ikenga packages — served via GitHub Pages,
> consumed by the shell and the `ikenga` CLI for pkg discovery, install, and update.

## What it is

A machine-generated catalog of every published [`@ikenga/pkg-*`](https://www.npmjs.com/org/ikenga)
package: names, versions, manifests, tarball URLs, integrity hashes — plus a minisign
signature over the index.

**Endpoint:** https://royalti-io.github.io/ikenga-registry/index.json

## Layout

```
index.json              — root catalog: pkg names + latest version + detail pointer
index.json.minisig      — minisign signature over index.json
pkgs/<short-name>.json  — per-pkg detail: every published version, manifest, tarball URL, integrity
schema/                 — JSON Schemas (mirrors of @ikenga/contract Zod types)
REGISTRY_PUBLIC_KEY.txt — minisign public key for verifying signatures
REGISTRY_PUBLIC_KEY.md  — human-readable verify instructions
```

## Updating

This repo is **not edited by hand**. The release workflow in
[`Royalti-io/ikenga-pkgs`](https://github.com/Royalti-io/ikenga-pkgs)
runs `scripts/update-registry-index.mjs` after every successful Changesets
publish; that script writes/updates `index.json` and `pkgs/<name>.json`,
re-signs `index.json` with minisign, and pushes the changes.

The only manual edits accepted here are:

- README/governance updates
- Schema bumps (after the matching `@ikenga/contract` release)
- Rotating the signing key (requires shell-side public-key update)

## Signature

`index.json.minisig` is a [minisign](https://jedisct1.github.io/minisign/) signature
over `index.json`. The shell embeds the public key and refuses to trust an unsigned
or mismatched index.

Verify locally:

```bash
minisign -Vm index.json -P RWRTqugAYXnZRgZPMyuqRNB3G41wg+AhSU2yT8nmDNNQlWQPeCfRXAvI
```

The current public key also lives at [`REGISTRY_PUBLIC_KEY.txt`](REGISTRY_PUBLIC_KEY.txt).

## Package kinds

`kind` is a **descriptive hint for catalog and discovery** — it is deliberately an
open string, and the schema does not constrain it. The kernel never dispatches on
it: at load time it dispatches on which manifest blocks are present, and the CLI
does not read it at all. Adding a new kind therefore needs no schema change.

Values in use today:

| `kind` | What it describes |
|---|---|
| `bundle` | A composite package grouping several components or sub-packages |
| `embedded` | A package whose UI embeds in the shell (the common case for apps) |
| `engine` | An engine adapter — the agent runtime a pkg drives |
| `skill` | A capability or tool surfaced to the agent |
| `webview` | A package whose surface is an embedded third-party site |

**`kind` is not what makes a package mount as a webview.** That is decided by
`ui.routes[].kind = "webview"` plus a `capabilities.webview` block declaring
`child_webviews: true` — a package can carry `kind: "embedded"` and still mount a
webview route, which is exactly what the Spotify / Sentry / Notion packages do.
Constraining this field to an enum would break that, and would break the next kind
someone adds; keep it open.

## Schema

The shape of `index.json` and `pkgs/<name>.json` is defined by Zod schemas
in [`@ikenga/contract`](https://github.com/Royalti-io/ikenga-contract) (`./registry`
export). JSON Schemas are mirrored under [`schema/`](schema/) for non-TypeScript
consumers and editor autocomplete.

## Links

- [`ikenga-pkgs`](https://github.com/Royalti-io/ikenga-pkgs) — the monorepo whose release workflow regenerates this index
- [`ikenga`](https://github.com/Royalti-io/ikenga) — the desktop shell that consumes it

## License

Apache-2.0 — see [`LICENSE`](LICENSE).
