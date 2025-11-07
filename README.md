# Automata Action

Update a Nix flake and land changes using Sapling. Supports submitting via
`ghstack` or a regular GitHub Pull Request, with optional GPG commit signing.

## What It Does

- Runs `nix flake update --accept-flake-config`
- Configures Sapling settings via `nix run "<flake-url>" -- config`
- Creates a CI commit via `nix run "<flake-url>" -- ci`
- Submits changes via:
  - `ghstack submit` when `method: ghstack` (default)
  - `pr submit` when `method: pr`
- Optionally imports a GPG private key and signs commits when `sign-commits: 'true'`

## Prerequisites

This action expects Nix to be available on the runner. If you’re using GitHub-hosted runners, add these steps before this action:

```yaml
- uses: DeterminateSystems/nix-installer-action@v20
- uses: DeterminateSystems/magic-nix-cache-action@v13
```

## Inputs

- `commit-message` (default: `Update Flake`)
  - Commit message used by the `ci` step
- `extra-args` (default: `""`)
  - Extra arguments passed to `nix run` for all subcommands
- `flake-url` (default: `nixpkgs#sapling`)
  - Nix flake URL or path to use (e.g., a tool flake that provides `config`, `ci`, `ghstack`, and `pr` subcommands)
- `github-token` (default: `${{ github.token }}`)
  - GitHub token to authenticate `ghstack` or PR submission
- `ghstack-username` (default: `github-actions`)
  - Username used by Sapling’s `ghstack` integration
- `gpg-private-key` (default: `""`)
  - ASCII-armored GPG private key used to sign commits when `sign-commits: 'true'`
- `gpg-passphrase` (default: `""`)
  - Passphrase for the provided GPG private key (if needed)
- `method` (default: `ghstack`)
  - Publication method: `ghstack` or `pr`
- `sign-commits` (default: `"false"`)
  - Set to `"true"` to enable GPG commit signing
- `username` (default: `github-actions[bot]`)
  - Sapling `ui.username` to set locally before submitting

## Quick Start (ghstack)

```yaml
name: ci
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  automata:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: shikanime-studio/setup-nix@v1

      - name: Update flake and land via ghstack
        uses: shikanime-studio/automata-action@v1
        with:
          commit-message: "Update flake and CI"
          flake-url: nixpkgs#sapling # or your own tool flake
          method: ghstack # default
          ghstack-username: github-actions
          username: github-actions[bot]
```

## Quick Start (PR)

```yaml
name: ci
on:
  workflow_dispatch:

jobs:
  automata:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: shikanime-studio/setup-nix@v1

      - name: Update flake and open PR
        uses: shikanime-studio/automata-action@v1
        with:
          commit-message: "Update flake and CI"
          method: pr
          github-token: ${{ secrets.GITHUB_TOKEN }}
          username: github-actions[bot]
```

## Optional: GPG Commit Signing

Provide a private key and enable signing:

```yaml
- name: Update flake with signed commits
  uses: shikanime-studio/automata-action@v1
  with:
    sign-commits: "true"
    gpg-private-key: ${{ secrets.CI_GPG_PRIVATE_KEY }}
    gpg-passphrase: ${{ secrets.CI_GPG_PASSPHRASE }}
```
