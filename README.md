# cad0p/.github

Account-wide defaults and shared CI for repositories owned by [`cad0p`](https://github.com/cad0p). Public by necessity: default community health files for a personal account are only served from a **public** `.github` repository.

## Contents

- **Default community health files** — `SECURITY.md`, `CONTRIBUTING.md`, `PULL_REQUEST_TEMPLATE.md`. These apply to every `cad0p` repository that does not define its own.
- **Reusable workflows** — `.github/workflows/*.yml`. Consuming repositories call them pinned at a full commit SHA.

Not here by design: `AGENTS.md` (per-repo, byte-for-byte), `setup-repo.sh` (local-checkout tooling), repository-specific workflows until a second shared workflow justifies it.

## Using a reusable workflow

```yaml
name: pin-guard
on: pull_request_target
permissions: {}
jobs:
  guard:
    permissions:
      contents: read
      pull-requests: read
      checks: write
    uses: cad0p/.github/.github/workflows/pin-guard.yml@<40hex> # vX.Y.Z
```

Rules:

- **Pin by full 40-character SHA + version comment.** SHA bumps are human-only — never automated; a bump to an unreviewed SHA is the only way a change here reaches consumers (SHAs are content-addressed, so a change here cannot rewrite what pinned callers fetch).
- `pull_request_target` workflows are sourced from the repository's **default branch** (GitHub behavior, effective 2025-12-08) and the called file is immutable at the pinned SHA — a PR cannot alter the gate.
- Called jobs appear as `<caller-job> / <called-job>`. `pin-guard` posts its own check-run named `pin-guard` on the PR head SHA — add **that** context to the repository's ruleset.

### `pin-guard` (Renovate automerge rail)

Base-branch validation for Renovate pin PRs: verifies the bot identity, requires a pin-shaped diff, and derives the bump class from the diff itself (labels are never trusted); posts fail-closed on anything else. Lands with its build item (2026-09).
