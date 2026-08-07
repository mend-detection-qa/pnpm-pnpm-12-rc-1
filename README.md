# git-ssh-url-normalization

**Pattern:** `git-ssh-url-normalization` (also covers
`empty-version-range`)
**PM:** pnpm 12 RC 1 (`lockfileVersion: '9.0'`)
**Categories:** `registry_source`, `lockfile_format`,
`version_constraints`
**Generated:** 2026-08-07

## Feature exercised

pnpm 12 RC 1 normalizes git dependency URLs to canonical HTTPS
forms in `pnpm-lock.yaml`. An SSH URL declared in `package.json`
such as:

```
git+ssh://git@github.com/sindresorhus/is-positive.git#semver:^3.1.0
```

is written into the lockfile as:

```
https://github.com/sindresorhus/is-positive.git#commit=<sha>
```

The SSH form never appears in the lockfile for known hosts
(GitHub, GitLab, Bitbucket). This probe also exercises the
empty version range (`""`) feature of pnpm 12 RC 1: a dep
declared with `""` in `package.json` resolves to the
lockfile-pinned version.

## Dependencies in this probe

| Package | Declared specifier | Resolved version | Source |
|---|---|---|---|
| `chalk` | `5.3.0` | `5.3.0` | registry |
| `debug` | `""` | `4.3.5` | registry |
| `is-positive` | `git+ssh://git@github.com/sindresorhus/is-positive.git#semver:^3.1.0` | `3.1.0` at SHA `c9b30e71d704cd30fa71f2edd1ecc7dcc4985493` | git (HTTPS normalized) |
| `ms` | (transitive of `debug`) | `2.1.2` | registry |

## Expected dependency tree

- Root: `pnpm12-git-ssh-url-normalization@0.0.1`
- Direct deps: `chalk`, `debug`, `is-positive`
- `debug` -> `ms` (transitive)
- `chalk` -> no deps
- `is-positive` -> no deps (pure ESM, no runtime deps)

### Git dep (SSH-to-HTTPS normalization)

Mend reads `pnpm-lock.yaml`, so it will see:

```
https://github.com/sindresorhus/is-positive.git
```

Expected `expected-tree.json` entry:

```json
"is-positive": {
  "source": "git",
  "version": "3.1.0",
  "source_detail": {
    "url": "https://github.com/sindresorhus/is-positive.git",
    "commit": "c9b30e71d704cd30fa71f2edd1ecc7dcc4985493"
  }
}
```

Failure mode: UA reads `package.json` SSH URL instead of
lockfile HTTPS URL.

### Empty version range

`debug` is declared with `""` in `package.json`. The lockfile
pins `debug@4.3.5`. Expected: Mend reports `debug@4.3.5`.

Failure mode: dep omitted, version reported as `""`, or a
different version resolved from wildcard semantics.

## Mend config

Bucket A — `js-pnpm` has no dynamic version detection.
`.whitesource` emitted with `scanSettings.versioning` pinning:

- `pnpm: "12.0.0-rc.1"` — exact pnpm version this probe
  targets (pnpm 12 RC 1 feature).
- `node: "22.14.0"` — LTS Node pinned for reproducibility.

`configMode` is `"AUTO"` (no `whitesource.config` in this
probe root).

## Resolver notes

The Mend UA `PnpmLockCollector` reads `pnpm-lock.yaml` and
routes to `PnpmParserV9Impl` (lockfileVersion 9.0). It reads
deps from the `snapshots` section (not `packages`), and direct
deps from `importers["."].dependencies`. The resolved URL for
the git dep comes from the packages map key — which in pnpm 12
is the HTTPS-normalized form. Mend does not re-parse
`package.json` for the URL when a lockfile is present; if it
does, that is the bug this probe targets.

## Known Mend UA limitation

The pnpm lockfile parser supports lockfileVersion 5.x, 6.x,
and 7.x–9.x. pnpm 12 RC 1 produces a v9-format lockfile
(`lockfileVersion: '9.0'`). Whether the UA's `PnpmParserV9Impl`
handles the HTTPS-normalized git key format is the unknown this
probe probes.

## Files

```
git-ssh-url-normalization-20260807-150510/
├── .whitesource          Bucket-A versioning pins
├── README.md             This file
├── expected-tree.json    Ground-truth dependency tree
├── package.json          Manifest with SSH git URL + empty ""
└── pnpm-lock.yaml        v9 lockfile with HTTPS-normalized URL
```
