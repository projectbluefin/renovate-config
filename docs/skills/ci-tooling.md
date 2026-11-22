---
name: ci-tooling
description: >-
  Use when editing or validating this repository's GitHub Actions workflows
  (`.github/workflows/*.yml`) or Renovate config files, or when `actionlint`,
  `shellcheck`, or `renovate-config-validator` is mentioned. Covers how the repo
  is linted and validated, and the quoting rule shellcheck enforces in `run:`
  scripts.
metadata:
  context7-sources:
    - /rhysd/actionlint
    - /koalaman/shellcheck
---

# renovate-config CI tooling

## When to Use

- Editing `.github/workflows/renovate.yml` or `.github/workflows/ci.yml`.
- Editing `org-inherited-config.json`, `renovate-config.json`, `default.json`, or `renovate.json`.
- Running the PR preflight, or a change where `actionlint` or `renovate-config-validator` runs.

## When NOT to Use

- Repos other than `projectbluefin/renovate-config`. Other factory repos carry their own `docs/skills/` and validation gates.
- Judging Renovate's runtime behavior on a repository (that is the org config's job, not the CI tooling's).

## What runs

- **`ci.yml` ("Validate Renovate")** runs `renovate-config-validator --strict` on `org-inherited-config.json` and on `renovate-config.json`, on any PR touching a config file or either workflow.
- **`renovate.yml` ("Renovate Self-Hosted")** runs the bot every 3 hours (`cron: "0 */3 * * *"`); on PRs it runs in `RENOVATE_DRY_RUN=full`.
- **`actionlint`** is not wired into CI. It is the local workflow lint and the factory PR preflight gate.
- This repo ships **no `.pre-commit-config.yaml`**, so `pre-commit run --all-files` exits with `InvalidConfigError: .pre-commit-config.yaml is not a file`. That error is expected here and is not a gate.

## Core Process

1. Edit the workflow or config file.
2. `actionlint .github/workflows/*.yml` — must exit `0`.
3. `renovate-config-validator --strict org-inherited-config.json` and `renovate-config-validator --strict renovate-config.json` — both must report `Config validated successfully`.
4. Push; the PR's `Validate Renovate` check must go green (a config/workflow change triggers it; a docs-only change does not).

## The shellcheck rule actionlint enforces

`actionlint` runs [shellcheck][] on every `run:` script whose shell is `bash` or `sh`, and exits non-zero on its findings — including `info`-level ones such as `SC2086`. Quote every shell expansion in a `run:` script.

```yaml
# fails actionlint: SC2086:info — Double quote to prevent globbing and word splitting
- run: renovate-config-validator --strict ${CONFIG_FILE}

# passes
- run: renovate-config-validator --strict "${CONFIG_FILE}"
```

actionlint detects the shell from `shell:` (workflow, job, or step level) and the runner OS; it skips shellcheck for `pwsh` and other non-`bash`/`sh` shells. It disables the shellcheck rules that conflict with `${{ }}` expressions (SC1091, SC2050, SC2194, SC2154, SC2157, SC2043), so those never appear. Shellcheck can be turned off entirely with `actionlint -shellcheck=`.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "SC2086 is only `info`, so it can't fail CI." | `actionlint` exits non-zero on info-level shellcheck findings, so the local preflight fails. |
| "`pre-commit` failed, so the preflight is broken." | This repo has no pre-commit config; the `InvalidConfigError` is the expected result, not a gate. |
| "The workflow ran green in CI, so the lint is fine." | `actionlint` is not in CI. A green `Validate Renovate` says nothing about shellcheck cleanliness. |

## Red Flags

- An unquoted `$VAR` / `${VAR}` in a `run:` script.
- Declaring a workflow change complete without `actionlint .github/workflows/*.yml` exiting `0`.
- Adding a config file without extending `ci.yml`'s `on.pull_request.paths` so it is validated.

## Verification

- [ ] `actionlint .github/workflows/*.yml` exits `0`.
- [ ] `renovate-config-validator --strict` passes on `org-inherited-config.json` and `renovate-config.json`.
- [ ] The PR's `Validate Renovate` check is green (when the change touches a config file or workflow).

[shellcheck]: https://www.shellcheck.net/
