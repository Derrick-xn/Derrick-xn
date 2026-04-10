# Profile README Cards Recovery Design

## Context

This repository is a GitHub profile repository whose visible content is currently driven entirely by `README.md`.

The existing README embeds two `github-readme-stats` cards from the public Vercel instance:

- GitHub stats card
- Top languages card

Those external URLs are no longer reliable for this profile and now surface a paused or unavailable deployment message instead of valid cards.

## Goal

Restore the two existing cards with minimal visual change and no paid hosting or dedicated server resources.

## Non-Goals

- Redesign the profile README
- Add new profile sections or badges
- Introduce a separately hosted frontend
- Guarantee private stats without user-managed GitHub credentials

## Chosen Approach

Use GitHub Actions to generate static SVG cards inside this repository and update `README.md` to reference those local files.

This keeps the visible README structure close to the current version while removing the runtime dependency on the public `github-readme-stats.vercel.app` deployment.

## Why This Approach

- No external server or paid hosting is required
- Generated SVG files are stable once committed
- The card style stays aligned with the current `github-readme-stats` look
- The workflow can run on a schedule and on demand
- The repository can support a future private-stats token without redesigning the setup

## Alternatives Considered

### 1. Keep using the public `vercel.app` endpoint

Rejected because the upstream project explicitly documents the public instance as best-effort and unreliable.

### 2. Self-host a personal deployment

Rejected because the user does not have a deployment endpoint and wants a zero-cost setup.

### 3. Replace dynamic cards with plain static text or badges

Rejected because it would change the existing presentation more than necessary.

## Planned Repository Changes

### `README.md`

Replace the two current remote image URLs with local repository paths:

- `./profile/stats.svg`
- `./profile/top-langs.svg`

The surrounding README structure and copy should remain unchanged.

### `.github/workflows/grs.yml`

Add a workflow that:

- runs on a daily schedule
- supports manual dispatch
- checks out the repository
- generates the stats card SVG
- generates the top-languages SVG
- commits refreshed SVG assets when they change

### `profile/`

Store generated SVG outputs in:

- `profile/stats.svg`
- `profile/top-langs.svg`

## Token Strategy

The workflow should support two modes:

### Default mode

Use the repository-provided `GITHUB_TOKEN` so the setup works immediately with no extra user action. This restores public stats only.

### Optional enhanced mode

If the user later adds a repository secret such as `PROFILE_STATS_TOKEN`, the workflow should be easy to switch to that token for private-stat visibility.

Because GitHub secrets cannot be created from this local workspace, the implementation must not block on that step.

## Card Options

The generated cards should preserve the current intent as closely as possible:

- stats card keeps `show_icons=true`
- stats card keeps the `radical` theme
- stats card keeps the existing private-stats intent in configuration where practical, with the understanding that actual private data requires a user-supplied PAT
- top-languages card remains a `top-langs` card for the same username

## Verification Plan

After implementation:

1. Validate workflow YAML syntax locally as far as practical.
2. Confirm `README.md` references local SVG paths instead of the broken public URLs.
3. Trigger or prepare a manual GitHub Actions run path for the user.
4. Verify that the repository contains the expected generated SVG file paths.

## Risks

### First run dependency

The local SVG files will not exist until the workflow generates them or placeholder files are added.

### Private stats expectation gap

Without a user-provided PAT in repository secrets, the restored cards will only include public statistics.

### GitHub Actions write permissions

The workflow must request repository contents write permission so it can commit refreshed SVG outputs.

## Implementation Boundary

The implementation should remain intentionally small:

- one workflow file
- minimal `README.md` edits
- generated SVG output directory

No additional feature work should be bundled into this change.
