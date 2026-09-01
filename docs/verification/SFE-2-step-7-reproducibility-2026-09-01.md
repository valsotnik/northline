# SFE-2 Step 7 reproducibility evidence

- Date: 2026-09-01
- Branch: `codex/SFE-2-engineering-foundation`
- Host: macOS on `darwin arm64`
- Scope: a clean checkout of the committed SFE-2 branch

## Toolchain

The following versions were captured before and after the clean dependency install:

| Tool        | Version                            |
| ----------- | ---------------------------------- |
| Node.js     | `24.18.0`                          |
| Corepack    | `0.35.0`                           |
| pnpm        | `10.33.0`                          |
| Git         | `2.50.1 (Apple Git-155)`           |
| Nx          | `23.1.2` local; no global Nx found |
| Angular CLI | `22.0.9`                           |
| Angular     | `22.0.8`                           |
| TypeScript  | `6.0.3`                            |
| WebStorm    | `2026.2.1` (`WS-262.9437.27`)      |

Commands:

```bash
git branch --show-current
node --version
corepack --version
pnpm --version
git --version
pnpm exec nx --version
pnpm exec ng version
pnpm exec tsc --version
```

## Frozen dependency installation

The SHA-256 checksum captured before the install was:

```text
d5e028bba4225f0cabac903e555a48bc7d33f2da53021029af339f266d601619  pnpm-lock.yaml
```

The committed branch was cloned with `--no-hardlinks` into a newly created temporary directory.
The checkout contained no `node_modules`, `.nx`, or `dist` directory when this command started:

```bash
pnpm install --frozen-lockfile
```

Result: passed. pnpm reported that the lockfile was up to date, skipped dependency resolution,
and reconstructed the dependency tree from the lockfile. The local `nx` and `ng` executables were
present after installation.

The post-install checksum was identical:

```text
d5e028bba4225f0cabac903e555a48bc7d33f2da53021029af339f266d601619  pnpm-lock.yaml
```

The lockfile checksum remained unchanged throughout the install and complete quality gate. The
temporary clone was retained only for the duration of verification.

## Uncached command contract

Nx state was reset first, and all non-continuous repository checks were run with local cache use
disabled:

```bash
pnpm exec nx reset
NX_SKIP_NX_CACHE=true NX_DAEMON=false pnpm check
```

Result: passed.

- Prettier: all matched files use Prettier formatting.
- ESLint: `web:lint` passed; Nx reported `Cache: Skipped (--skip-nx-cache)`.
- Type checking: Angular application/template compilation and test TypeScript passed; Nx reported
  `Cache: Skipped (--skip-nx-cache)`.
- Tests: 1 test file and 1 test passed; Nx reported `Cache: Skipped (--skip-nx-cache)`.
- Production build: passed; Nx reported `Cache: Skipped (--skip-nx-cache)`.
- Production output: `187.54 kB` raw initial JavaScript and `51.13 kB` estimated transfer size.

## Development server

The continuous command was checked separately on a non-default localhost port:

```bash
pnpm serve -- --host=127.0.0.1 --port=4207
curl --fail --silent --show-error --output /dev/null \
  --write-out 'http_status=%{http_code}\ncontent_type=%{content_type}\n' \
  http://127.0.0.1:4207/
curl --fail --silent --show-error http://127.0.0.1:4207/
```

The application compiled successfully. The HTTP request returned status `200`, content type
`text/html`, `<title>Northline</title>`, and `<app-root></app-root>`. The server was stopped with
`Ctrl-C`; a subsequent request could not connect to port 4207, confirming cleanup.

## Repository invariants

```bash
pnpm exec nx show projects
git diff --check
git ls-files -- node_modules .nx dist
git status --short --ignored -- node_modules .nx dist
```

Results:

- Nx listed exactly `["web"]`.
- `git diff --check` exited successfully with no output.
- `git ls-files` returned no files for `node_modules`, `.nx`, or `dist`.
- Git reported `node_modules/`, `.nx/`, and `dist/` as ignored generated state.

## Environment caveats

- The final verification used a separate clone of the repository's committed SFE-2 branch. The clone
  began without `node_modules`, `.nx`, or `dist`, so it did not depend on generated state from the
  development worktree.
- The first sandboxed install attempt could not resolve `registry.npmjs.org` (`ENOTFOUND`). It was
  stopped, its partial dependency directory was removed, and the original backup was restored
  before repeating the clean-install procedure with approved network access.
- pnpm used the host's content-addressable store during the successful clean install. No packages
  required downloading, but all dependency links were reconstructed from the frozen lockfile.
- pnpm warned that lifecycle scripts for `@parcel/watcher`, `lmdb`, and `msgpackr-extract` were
  ignored. This repository intentionally permits only `esbuild` and `nx` through
  `onlyBuiltDependencies`; all repository checks still passed.
- Angular's production build failed without diagnostics inside the filesystem/process sandbox, and
  the sandbox blocked the development server bind with `EPERM`. The complete uncached check and
  localhost serve verification passed outside those restrictions. Nx labelled the later successful
  build as flaky because it observed the preceding sandbox-induced failure.
