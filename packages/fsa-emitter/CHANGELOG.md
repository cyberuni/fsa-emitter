# fsa-emitter

## 3.0.0

### Major Changes

- 2c8179d: Part of an estate-wide sweep pinning `type-plus` to an exact `8.0.0-beta.10`. `fsa-emitter` is a wave-1 leaf — nothing else in the estate depends on it — so it goes first.
  
  ## Breaking: the TypeScript floor moves to 5.6
  
  `type-plus` **5, 6 and 7 declare no `typescript` peer at all.** 8 declares `>= 5.6.0`. That reaches consumers here, because `AnyFunction` is in the published declarations:
  
  ```ts
  import { AnyFunction } from "type-plus";
  ```
  
  Anyone type-checking against `fsa-emitter` resolves type-plus's own `.d.ts` and inherits the peer. Hence `major`.
  
  The last published release is **2.0.2**, which already declares `type-plus: ^7.0.0` — the same as `main`. So the jump consumers actually see is **7 → 8**.
  
  ## Why an exact pin
  
  `^8.0.0-beta.10` resolves to `>=8.0.0-beta.10 <9.0.0-0`, which admits every later 8.0.0 prerelease as well as `8.0.0` and `8.1.0`. type-plus 8 is a prerelease line where breaking changes land between betas: beta.10 -> beta.11 changed `Equal`'s signature and removed `isType.f`. An exact version makes each bump a reviewable PR instead of something a lockfile refresh can do silently. Move back to a caret when 8.0.0 is stable.
  
  ## No source change
  
  `AnyFunction` is unchanged in 8 — this package's single type-plus import needed no edit. `pnpm verify` passes: 4/4 tasks, 51 tests.
  
  ## Second commit: first-party packages are exempt from the release-age soak
  
  `assertron@11.6.0` already declares `type-plus: ^8.0.0-beta.10` and `tersify: ^4.0.6`, but was younger than the 24h soak, so pnpm would otherwise silently resolve `11.5.2` instead and drag in `type-plus@7.6.2` and `tersify@3.12.1` alongside the 8.x this package pins. `pnpm-workspace.yaml` now exempts first-party names and scopes. Third-party packages are unaffected: `minimumReleaseAge: 1440` and `minimumReleaseAgeStrict: true` both stay.
  
  ```
  pnpm why type-plus -r  ->  Found 1 version of type-plus   (8.0.0-beta.10)
  pnpm why tersify -r    ->  Found 1 version of tersify     (4.0.6)
  ```
  
  ## Node floor
  
  type-plus 8 pulls `unpartial@^1.0.7`, which declares `engines: node >= 20`. This package already declares `^20.19.0 || ^22.13.0 || >=24`, so nothing moves here.
- 8fd6935: Ship ESM only. The CommonJS build is gone.
  
  **What was removed**
  
  - The `cjs/` output is no longer built or published, and with it `main` and `typings`.
  - The published tarball now contains `esm/` only. The `ts/` sources are no longer shipped; the sourcemaps carry `sourcesContent`, so stepping into the library still shows the original TypeScript.
  
  **What replaces it**
  
  - `"type": "module"` plus an `exports` map with no `require` condition:
  
    ```json
    {
      ".": {
        "types": "./esm/index.d.ts",
        "default": "./esm/index.js"
      },
      "./package.json": "./package.json"
    }
    ```
  
  - Declarations now sit beside the JavaScript in `esm/`, where they never shipped before. The emitted file names in `esm/` are unchanged.
  
  **What consumers must do**
  
  - Import the package, do not `require` it. On Node 22.12 and later `require('fsa-emitter')` still works through Node's built-in support for requiring ESM; on older runtimes it throws `ERR_REQUIRE_ESM`. Move to `import` or dynamic `import()`.
  - Import from the package root. Deep imports such as `fsa-emitter/esm/Emitter.js` and `fsa-emitter/ts/Emitter` no longer resolve, because the `exports` map does not expose subpaths. Everything the package exported is re-exported from the root.
  - TypeScript consumers need a module resolution that reads `exports` — `node16`, `nodenext`, or `bundler`. The legacy `node` resolution finds no entry point, since `main` and `typings` are gone.

### Patch Changes

- 9c64c64: Declare a supported Node range: `^20.19.0 || ^22.13.0 || >=24`.
  
  Every version in that range has unflagged `require(esm)`, so a CommonJS consumer's
  `require()` of this now-ESM-only package resolves rather than throwing `ERR_REQUIRE_ESM`.
  Node 18 (EOL April 2025) and Node 20.0–20.18 are excluded because `require()` hard-fails there.

## 2.0.2

### Patch Changes

- 1878fd5: Build with tsdown instead of two `tsc` passes.
  
  Every published path is unchanged — `cjs/*.js`, `cjs/*.d.ts`, `esm/*.js` and their
  sourcemaps all land where they always have, and the public API is identical. The
  emitted JavaScript itself is now produced by rolldown rather than `tsc`, which is
  why this is a release rather than a no-op.
  
  The tarball also now contains the `ts/` sources. `files` had listed a `src`
  directory that has never existed in this repo, so the `.js.map` files shipped with
  nothing to resolve against.

## 2.0.1

### Patch Changes

- 74d42b2: Restore the version line to the published series.

  `package.json` still carried semantic-release's `0.0.0-development` placeholder, so the
  first changesets release bumped it to `0.0.0` and npm made that `latest` — ahead of the
  real `2.0.0` by dist-tag, behind it by semver. This publishes `2.0.1` and returns `latest`
  to the 2.x line.

## 0.0.0

### Patch Changes

- 0d480ad: Point `repository`, `homepage` and `bugs` at `cyberuni/fsa-emitter`.

  `repository` is read when generating provenance attestations, so it has to be correct
  at publish time — not merely correct in the repo.
