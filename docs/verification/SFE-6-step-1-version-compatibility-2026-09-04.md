# SFE-6 Step 1 version compatibility

- **Ticket:** SFE-6
- **Branch:** `codex/SFE-6-portfolio-api-boundary`
- **Date:** 2026-09-04
- **Scope:** Reconfirm the supported Node, Nx, `@nx/nest`, NestJS, TypeScript, and OpenAPI type-generation version matrix before creating an API proof.

## Local baseline

`pnpm exec nx report` returned:

| Tool or package                           | Local version                                                                         |
| ----------------------------------------- | ------------------------------------------------------------------------------------- |
| Node                                      | `24.18.0`                                                                             |
| pnpm                                      | `10.33.0`                                                                             |
| Nx                                        | `23.1.2`                                                                              |
| `@nx/angular`                             | `23.1.2`                                                                              |
| `@nx/js`                                  | `23.1.2`                                                                              |
| TypeScript                                | `6.0.3`                                                                               |
| Angular packages resolved by the lockfile | `@angular/core` `22.0.8`, `@angular/compiler-cli` `22.0.8`, `@angular/build` `22.0.9` |
| RxJS                                      | `7.8.2`                                                                               |

`pnpm exec nx show projects` returned only `web`, which matches ADR 0001 before this spike proves a concrete API project.

## Compatibility matrix

### Angular 22 runtime

Angular's version table lists Angular `22.0.x` with Node
`^22.22.3 || ^24.15.0 || ^26.0.0`, TypeScript `>=6.0.0 <6.1.0`, and RxJS
`^6.5.3 || ^7.4.0`.

Conclusion: local Node `24.18.0`, TypeScript `6.0.3`, and RxJS `7.8.2` satisfy
that row. The current Angular line is internally compatible for this ticket.

### Nx 23 runtime

Nx 23 release notes state that Node 20 was dropped and the minimum Node version
is Node 22.

Conclusion: local Node `24.18.0` is compatible with Nx 23.

### Nx plugin alignment

The repository uses Nx `23.1.2` packages. Registry metadata for
`@nx/nest@23.1.2` depends on `@nx/js`, `@nx/node`, `@nx/devkit`, and `@nx/eslint`
at `23.1.2`.

Conclusion: use `@nx/nest@23.1.2` for the proof so the new plugin stays aligned
with the workspace.

### NestJS line for `@nx/nest`

Registry metadata for `@nx/nest@23.1.2` peers with `@nestjs/core` and
`@nestjs/common` `>=10.0.0 <12.0.0`. Registry metadata for
`@nestjs/core@11.2.3`, `@nestjs/common@11.2.3`, and
`@nestjs/platform-express@11.2.3` all remain on the Nest 11 peer line, and
`@nestjs/core@11.2.3` declares Node `>=20`.

Conclusion: prove the API with NestJS 11, not NestJS 12.

### NestJS 12 comparison

Registry metadata for `@nestjs/core@12.0.1` peers with Nest 12 packages, while
`@nx/nest@23.1.2` excludes Nest 12 through `<12.0.0`.

Conclusion: defer NestJS 12 until the Nx plugin line supports it or a separate
evidence-backed override is approved.

### Nest OpenAPI generation

The current `@nestjs/swagger` package is on a Nest 12 peer line. The
`@nestjs/swagger@11.4.7` package peers with `@nestjs/core` and `@nestjs/common`
`^11.0.1`.

Conclusion: if Step 2 uses Nest's Swagger document generation, pin
`@nestjs/swagger` to the 11.x line.

### Frontend OpenAPI type generation

`openapi-typescript@7.13.0` declares a TypeScript peer of `^5.x`, which does not
match this repository's TypeScript `6.0.3`. `@hey-api/openapi-ts@0.99.0`
declares TypeScript `>=5.5.3 || >=6.0.0 || 6.0.1-rc` and Node `>=22.18.0`,
which the local runtime satisfies.

Conclusion: Step 2 should prove type generation before selecting the generator.
Prefer a TypeScript 6-compatible generator if `openapi-typescript` produces peer
friction under pnpm.

## Recommendation for Step 2

Create the smallest API proof with an Nx-aligned Nest project candidate:

- `@nx/nest@23.1.2`
- NestJS 11 packages, starting from the current 11.x line rather than NestJS 12
- `@nestjs/swagger@11.x` only if the proof generates the OpenAPI document from Nest metadata
- A TypeScript 6-compatible frontend type generator, with `@hey-api/openapi-ts` as the current lower-friction candidate and `openapi-typescript` treated as a risk to verify

This recommendation does not yet approve a production API platform. Step 2 still needs to prove build and test integration through repository-owned commands and the Nx project graph.

## Commands run

```bash
node -v
corepack pnpm -v
pnpm exec nx report
pnpm exec nx show projects
pnpm view @nx/nest@23.1.2 version peerDependencies dependencies engines --json
pnpm view @nestjs/core@11.2.3 version peerDependencies engines --json
pnpm view @nestjs/common@11.2.3 version peerDependencies engines --json
pnpm view @nestjs/platform-express@11.2.3 version peerDependencies engines --json
pnpm view @nestjs/core@12 version peerDependencies dependencies engines --json
pnpm view @nestjs/swagger@11.4.7 version peerDependencies dependencies engines --json
pnpm view openapi-typescript version peerDependencies dependencies engines --json
pnpm view @hey-api/openapi-ts version peerDependencies dependencies engines --json
```

The npm registry metadata commands required network access and were rerun outside the filesystem sandbox with a narrow `pnpm view` approval after sandboxed calls hung without output.

## Sources

- Angular version compatibility: https://angular.dev/reference/versions
- Nx 23 release notes: https://nx.dev/blog/nx-23-release
- Nx Nest plugin metadata: https://registry.npmjs.org/@nx%2Fnest/23.1.2
- NestJS core metadata: https://registry.npmjs.org/@nestjs%2Fcore/11.2.3
- NestJS Swagger metadata: https://registry.npmjs.org/@nestjs%2Fswagger/11.4.7
- `openapi-typescript` metadata: https://registry.npmjs.org/openapi-typescript/latest
- `@hey-api/openapi-ts` metadata: https://registry.npmjs.org/@hey-api%2Fopenapi-ts/latest
