# Drop Dungeon fork of AppLovin-MAX-Defold

This fork exists so that Drop Dungeon (`kallehiitola/DropLoot`, `client/game.project`)
can pin a MAX Defold plugin that upstream has not released: upstream `master` is
still 1.1.0 (MAX SDK 12.2.0, 2024-02-21) while the native SDKs are at 13.6.x.
ADR-0024 in the game repository records the decision.

## What is here

- Branch `droploot` = the head of upstream pull request #17
  (`AppLovin/AppLovin-MAX-Defold#17`, "Upgrade AppLovin MAX integration to 2.0.0",
  Alexey Gulev, 2026-07-29, head `feb9a46a807553c349eb5a09f0295e689977d514`)
  plus this note. Requirements: Defold 1.13.0+, Android API 24+, iOS 15+.
- Tag `droploot-2.0.0` is the commit the game pins:
  `https://github.com/kallehiitola/AppLovin-MAX-Defold/archive/refs/tags/droploot-2.0.0.zip`.

## Review (2026-09-04)

The diff against upstream `master` (84 files, +11451 / -2720) was read for
anything unsafe before tagging:

- Native bridges (`MaxDefoldPlugin.java`, `MADefoldPlugin.mm`, `applovin_*.cpp/mm`):
  no reflection, process execution, dynamic loading or network calls of their
  own; everything goes through the MAX SDK API.
- `updater/*.py`: maintainer tooling that downloads the SDK artifacts from the
  vendor hosts with pinned SHA-256 checksums (`updater/versions.json`). It is
  not part of the built extension and never runs in a game build.
- `.github/workflows/bob.yml`: builds the example with a checksum-verified
  `bob.jar` against Defold's build servers. No secrets, no publishing.
- Manifests: `build.gradle` pins `com.applovin:applovin-sdk:13.6.3` and Google
  UMP 4.0.0; the Podfile pins `AppLovinSDK 13.6.3`; every optional mediation
  adapter is off unless the consuming `game.project` switches it on.
- Repository checks: `python3 -B updater/{adapters,android,ios}.py check` report
  up to date; `python3 -B -m unittest discover -s tests` passes on Linux
  (37 tests, 1 skipped: the IPA test needs a bundled IPA). On Windows the two
  iOS resource-tree checksum tests fail on path separators; that is the test
  harness, not the payload.

## SDK pin

13.6.3, as the pull request pins it. 13.6.4 (2026-08-11, both platforms) adds
"user information parameters" and removes the VK adapter detection, neither of
which the game uses. Bumping means updating `updater/versions.json` and
regenerating with `updater/ios.py` and `updater/android.py` (see
`updater/README.md` and `DEVELOPMENT.md`), then a fresh tag `droploot-<version>`;
it is not done ahead of a device test on the version the pull request was
written against.

## Updating

```sh
git fetch upstream
git fetch upstream pull/17/head:pr-17     # or the merged commit once upstream lands it
git checkout droploot && git rebase pr-17  # or merge upstream/master
python3 -B updater/adapters.py check && python3 -B updater/android.py check && python3 -B updater/ios.py check
python3 -B -m unittest discover -s tests
git tag droploot-<version> && git push origin droploot droploot-<version>
```
