# Releases & Channels

## release-please

[release-please](https://github.com/googleapis/release-please) runs on every push to `main` and `beta`. It maintains a release PR per branch; merging that PR tags the release and updates:

- `CHANGELOG.md`
- `version.txt`
- `MARKETING_VERSION` in `Configs/Version.xcconfig` (generic-updater marker)

Versions are derived from Conventional Commits: `feat:` → minor, `fix:` → patch, `feat!:`/`BREAKING CHANGE:` → major (pre-1.0: minor).

## Channels

| Channel | Source | Version | Audience |
|---|---|---|---|
| **alpha** | Debug builds, feature branches | untagged | developers |
| **beta** | `beta` branch | `x.y.z-beta.n` prerelease tags | early testers |
| **stable** | `main` | `x.y.z` tags | everyone |

The running app derives its channel from the build itself — Debug ⇒ alpha, a `-beta` marketing version ⇒ beta, otherwise stable. Features gate on the channel via [feature flags](https://github.com/apps3k-com/CodingBuddy/blob/main/docs/FEATURE_FLAGS.md).

## Cutting a beta

```bash
git checkout main && git pull
git checkout -B beta && git push -u origin beta
# release-please opens a prerelease PR on beta → merge → x.y.z-beta.1
```

Stable releases happen by merging the release PR that release-please keeps open on `main`.

## Operator flow

1. Merge a reviewed change to `main` or `beta`.
2. Merge the release-please PR for that branch.
3. Confirm the GitHub Release and its generated notes.
4. Test the tagged build before sharing it with the intended channel audience.

CodingBuddy has no server deployment: a GitHub Release is the delivery record.
The repository deliberately does not attach an unsigned app archive. The macOS
CI remains GitHub-hosted because the public repository must not execute pull
request code on the privileged self-hosted runner fleet. Documentation-only
changes skip that expensive macOS build; a newer PR update cancels its obsolete
in-flight build.

## Enforcement pipeline

| Check | Where | What |
|---|---|---|
| Conventional commit message | `commit-msg` hook | every local commit |
| Conventional PR title | `enforce.yml` | squash title feeds release-please |
| Feature-flag registry sync | `pre-push` hook + `enforce.yml` | `FeatureFlags.swift` ⇄ `docs/FEATURE_FLAGS.md` |
| Docs updated | `enforce.yml` | app changes require `docs/wiki/` updates or `Docs: none` |
| Build + tests | `ci.yml` | every PR and push to `main`/`beta` |
| Wiki publish | `wiki-sync.yml` | `docs/wiki/` → GitHub wiki on merge |
