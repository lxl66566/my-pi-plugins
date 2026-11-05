# my-pi-plugins

My [pi](https://github.com/earendil-works/pi) extensions in one monorepo. Repo-level infrastructure (toolchain configs, CI, git hooks) is declared once at the root; each package publishes to npm separately.

| Package                                             | npm                                                                      | Description                                                                                                       |
| --------------------------------------------------- | ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------- |
| [pi-replace-function](packages/pi-replace-function) | [pi-replace-function](https://www.npmjs.com/package/pi-replace-function) | `replace_function` tool: locate a declaration on the syntax tree and swap it in place from a full new declaration |
| [pi-reuse-model](packages/pi-reuse-model)           | [pi-reuse-model](https://www.npmjs.com/package/pi-reuse-model)           | Start every fresh session on the model and thinking level last used in this workspace                             |

## Install

```sh
pi install npm:pi-replace-function
pi install npm:pi-reuse-model
```

Grammar packages ship prebuilt binaries, so no compiler toolchain is needed. If you manage pi extensions with pnpm, run `pnpm approve-builds` in `~/.pi/agent/npm` once: pnpm ≥ 10 blocks the tree-sitter grammar packages' build scripts by default.

## Development

```sh
pnpm install
pnpm verify   # fmt:check + lint + tsc + tests (all packages)
pnpm test     # vitest
```

Shared tooling and host-provided type dependencies (`@earendil-works/pi-coding-agent`, `typebox`) are devDependencies of the root workspace; packages declare them only as `peerDependencies` per pi's guidance. Releases go through the Release workflow (`workflow_dispatch`), which verifies and publishes both packages.
