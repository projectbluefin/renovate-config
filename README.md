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
  
- **`renovate.json`** - This repo's own Renovate config
- **`.github/workflows/renovate.yml`** - Self-hosted Renovate workflow (runs every 30 minutes)
- **`.github/workflows/ci.yml`** - Validates Renovate configuration on PRs

## How It Works

The self-hosted Renovate bot:
1. Runs every 30 minutes via GitHub Actions
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

Repositories can inherit the org configuration by adding to their `.github/renovate.json5`:

```json
{
  "extends": ["github>projectbluefin/renovate-config:org-inherited-config"]
}
```

Or simply rely on autodiscovery if they don't need custom overrides.

## Testing

On pull requests, the bot runs in dry-run mode to validate changes without creating actual PRs.

You can also manually trigger the workflow with custom log levels (info or debug) for troubleshooting.

## Based On

This configuration is based on [ublue-os/renovate-config](https://github.com/ublue-os/renovate-config) and adapted for Project Bluefin.
