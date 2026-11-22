# renovate-config

Self-hosted Renovate bot configuration for Project Bluefin organization.

## Overview

This repository contains the configuration for a self-hosted Renovate bot that automatically updates dependencies across all Project Bluefin repositories.

## Files

- **`org-inherited-config.json`** - Shared configuration inherited by all org repositories
  - Best practices preset
  - Semantic commits
  - Custom regex manager for image-versions YAML files
  
- **`renovate-config.json`** - Bot configuration
  - Autodiscovery for `projectbluefin/*` repositories
  - Inherits from this repo's config
  - Requires config in repositories
  - `repositories`: per-repository overrides. `projectbluefin/contribute` gets an `allowedCommands` list, one anchored regex per literal `postUpgradeTasks` command; no other repository gets one, so the runner executes no commands for them. An entry trusts that repository's writers with the runner's org-wide token (see [Trusting Repository Developers](https://docs.renovatebot.com/security-and-permissions/#trusting-repository-developers))
  
- **`default.json`** - The org's default preset, served to any repo that extends `local>projectbluefin/renovate-config`; wraps `config:recommended`
- **`renovate.json`** - This repo's own Renovate config (dependency updates for renovate-config itself)
- **`.github/workflows/renovate.yml`** - Self-hosted Renovate workflow (runs every 3 hours)
- **`.github/workflows/ci.yml`** - Validates Renovate configuration on PRs

## How It Works

The self-hosted Renovate bot:
1. Runs every 3 hours via GitHub Actions
2. Uses a GitHub App for authentication (`RENOVATE_APP_ID` and `RENOVATE_PRIVATE_KEY` secrets)
3. Autodiscovers all `projectbluefin/*` repositories
4. Creates PRs for dependency updates based on org-wide configuration
5. Can be manually triggered via workflow_dispatch

## Setup Requirements

To enable the self-hosted Renovate bot, you need:

1. **GitHub App**: Create a GitHub App with these permissions:
   - Repository permissions:
     - Contents: Read & Write
     - Pull Requests: Read & Write
     - Issues: Read & Write
     - Metadata: Read-only
   - Organization permissions:
     - Members: Read-only

2. **Repository Secrets**:
   - `RENOVATE_APP_ID` - The GitHub App ID
   - `RENOVATE_PRIVATE_KEY` - The GitHub App private key (PEM format)

3. **Install the GitHub App** on the `projectbluefin` organization

## Usage in Repositories

Org-wide rules in `org-inherited-config.json` are inherited automatically by every repository via Renovate's `inheritConfig` setting — no per-repo `extends` is needed for the org rules.

A repository still needs its own config file (`renovate.json` or `.github/renovate.json5`), since `requireConfig` is `"required"`. To also get the org's `config:recommended`-based default base, it can extend the default preset:

```json
{
  "extends": ["local>projectbluefin/renovate-config"]
}
```

`local>projectbluefin/renovate-config` resolves to `default.json`; the named `org-inherited-config` preset is `local>projectbluefin/renovate-config:org-inherited-config`.

## Testing

On pull requests, the bot runs in dry-run mode to validate changes without creating actual PRs.

You can also manually trigger the workflow with custom log levels (info or debug) for troubleshooting.

## Based On

This configuration is based on [ublue-os/renovate-config](https://github.com/ublue-os/renovate-config) and adapted for Project Bluefin.
