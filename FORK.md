<!-- SPDX-FileCopyrightText: 2026 Yangtse Su <yangtsesu@gmail.com> -->
<!-- SPDX-License-Identifier: GPL-2.0-or-later -->

# This fork's rules (`YangtseSu/rofd`)

This repository is a fork of [`linuxdeepin/rofd`](https://github.com/linuxdeepin/rofd)
that carries a handful of changes we need before they land upstream. This file
describes how the fork works. It lives on `dev` only, because `feat/*` / `fix/*`
branches must keep a diff against upstream that is the change alone.

## Branches

| Branch | What it is |
| --- | --- |
| `main` | A **mirror of upstream** `linuxdeepin/rofd`. Nothing of ours is committed here; upstream's own `qml` and legacy branches are untouched too. |
| `feat/*`, `fix/*` | One branch per change, based on `main`. Each is proposed upstream as a PR, so its diff must be nothing but the change. The source of truth for that change until it merges upstream. |
| `dev` | The integration branch: what we ship. A new release is tagged here. |

The flow is: branch `feat/x` off `main`, open the upstream PR, then
`git merge --no-ff feat/x` into `dev`.

`--no-ff` matters. It keeps one merge commit per carried change, which makes the
inventory derivable instead of narrated:

```bash
git log --oneline --merges origin/main..dev    # what we carry, one line each
git diff --stat origin/main..dev               # the full delta against upstream
git show --first-parent dev                    # what our own history looks like
```

## Releases

A release is a tag on `dev`. There is no release branch.

1. Merge the new changes into `dev`, then set the version to
   `<upstream-version>-fork.<n>` (e.g. `0.4.3-fork.2`) in `Cargo.toml`, the
   three `crates/*/Cargo.toml`, and the assertion in
   `crates/rofd-core/tests/public_api.rs`; refresh `Cargo.lock`.
2. Push `dev` and let CI (`fork-checks`) pass — a push to `dev` runs it strictly,
   with no test skipped and no step tolerated.
3. Tag `v<version>` on the `dev` commit, and **never move a published tag**: the
   consumers pin it (`librofd-ffi`'s `source=` URL and `sha256sums` in the AUR,
   and any tarball already downloaded).
4. Create the GitHub release with a `git archive` tarball of the tag (it includes
   `vendor/`, so `--frozen` builds work offline) and record its sha256 in the
   consumer inventory.

Bumping the version on every release is not cosmetic: two different trees under
one version cannot both be pinned by a checksum, and a package that does not
change version is never upgraded.

## Divergence from this repository's `AGENTS.md`

`AGENTS.md` is upstream's file and stays untouched, so a merge from upstream
never conflicts with us. Its **Versioning and Releases** section describes
upstream's Debian-driven flow, with `debian/changelog` prepended on every release
and "the `vX.Y.Z` tag must match the Debian version". For `dev` releases that is
overridden:

- the version is `<upstream-version>-fork.<n>`, not a plain `vX.Y.Z`;
- `debian/changelog` is not prepended and the Debian package is not part of our
  release — the shipped artifact is the AUR `librofd-ffi` package built from the
  tag tarball;
- everything under **Build and Test** and **Code and API Rules** applies
  unchanged, and so does the never-move-a-published-tag rule.

## CI

`.github/workflows/fork-checks.yml` lives on `dev`. Upstream's Distribution
Builder gates every job behind `if: false` and needs a linuxdeepin member, so
without this workflow nothing would ever be checked.

- A push to `dev` runs all of it strictly: `cargo fmt --check`, clippy with
  `-D warnings`, the three crates' tests, the `debian/rules` build command, and
  `crates/rofd-ffi/tests/run_c_tests.sh`.
- Two tests are red on upstream `main` (`decoded_limits_have_exact_and_one_over_boundaries`,
  fixed by PR #14, and the region pixel diff, fixed by PR #15). `dev` carries the
  fixes, so they gate here. `feat/*` / `fix/*` branches are checked on demand:

  ```bash
  gh workflow run fork-checks.yml --ref dev -f refs=refs/heads/feat/<name>
  ```

  and then run in the loose mode that keeps those two tests visible but
  non-gating, so the branch's own regressions still fail the run.
