# renovate-config

Shared [Renovate](https://docs.renovatebot.com) presets for my repositories. Renovate finds this repository by its name when it onboards a new one and suggests extending it.

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>vmenon04/renovate-config"]
}
```

A repository whose CI tests every change may also opt into automatic merging:

```json
  "extends": ["github>vmenon04/renovate-config", "github>vmenon04/renovate-config:automerge"]
```

Exceptions that belong to one repository (a database whose major must never move by itself, say) go in that repository's `renovate.json`.

## `default.json`: the baseline

| Rule | Why |
| --- | --- |
| Renovate's `config:best-practices` | Pins Docker images to a digest and GitHub Actions to a commit, so a re-pushed tag can't change what runs; migrates old config; flags abandoned packages. |
| Early Monday mornings, New York time | Updates arrive in one batch a week instead of whenever something is released. Security fixes ignore the schedule. |
| Releases must be three days old, and no PR before that | Most broken or malicious releases are pulled or reported within days. Security fixes skip the wait. |
| GHCR images exempt from the wait | GHCR publishes no release dates, so the wait would hold these updates forever. They come at once, for review. |
| Weekly lock file refresh, with a warning | Keeps indirect dependencies current. The three-day wait can't be enforced on it, so its PR says so. |
| GitHub Actions grouped | One PR a week instead of one per Action. |
| `runtime` label on Python, Node, Postgres and Redis minors; `major` label on majors | These are the updates that change behaviour. |
| Nothing merges by itself | A person merges every PR unless the repository opts in below. |

## `automerge.json`: opt-in

Minor, patch and digest updates of versions 1.0 and later merge themselves once every status check passes, the three-day wait included. Never merged automatically: majors, anything still at 0.x, language and database minors, GHCR images, and lock file refreshes.

Only for repositories where CI tests what an update changes, and where a merge is not a production deploy without a staging step.

## Changing a preset

Every repository that extends a preset picks up `main` on its next run, so `main` is protected: changes go through a pull request, and `validate` (Renovate's own config validator) must pass. To check locally:

```sh
npx --package renovate -- renovate-config-validator --strict default.json automerge.json renovate.json
```

This repository is public so that public repositories can extend it. It must never hold secrets.
