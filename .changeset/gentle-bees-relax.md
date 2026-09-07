---
'fsa-emitter': major
---

Part of an estate-wide sweep pinning `type-plus` to an exact `8.0.0-beta.10`. `fsa-emitter` is a wave-1 leaf — nothing else in the estate depends on it — so it goes first.

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
