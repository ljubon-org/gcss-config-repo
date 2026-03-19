# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A GitOps-style configuration management repository for GitHub organization repository settings. YAML files define the desired state of repositories; GitHub Actions + Terraform Cloud apply those configurations via the reusable workflows in `ljubon-org/github-terraformer`.

## Repository Structure

- `repos/` — One YAML file per managed repository (e.g., `demo1.yaml`). Use `repos/repository.yaml.example` as the canonical reference for all supported fields.
- `config/app-list.yaml` — Maps GitHub Apps used as bypass actors in rulesets.
- `config/import-config.yaml` — Controls behavior of the importer (ignored repos, feature flags, API page size).
- `.github/workflows/` — All automation; delegates to reusable workflows in `ljubon-org/github-terraformer@test-env`.

## Workflows Overview

| Workflow | Trigger | Purpose |
|---|---|---|
| `bootstrap.yaml` | PR opened | Kicks off `tf-plan` |
| `tf-plan.yaml` | bootstrap / manual | Terraform plan |
| `tf-apply.yaml` | push to `main` | Terraform apply |
| `drift-check.yaml` | manual / schedule | Detect config drift |
| `import.yaml` | manual (repo name) | Import single repo |
| `bulk-import.yaml` | manual | Import many repos |
| `promote-imported-configs.yaml` | PR merged | Move configs from `importer_tmp_dir/` to `repos/` |
| `create-fork.yaml` | manual | Create a fork |

All workflows require secrets: `GITHUB_TOKEN`, `TFC_TOKEN`, `APP_PRIVATE_KEY`. Terraform Cloud org: `ljubo-demo`.

## Adding / Modifying a Repository Config

1. Create or edit a file in `repos/<repo-name>.yaml`.
2. Use `repos/repository.yaml.example` as reference — it documents every supported field with inline comments.
3. Open a PR; `bootstrap` → `tf-plan` runs automatically and shows the Terraform plan.
4. Merge to `main` to trigger `tf-apply`.

## Importing Existing Repositories

- **Single repo**: trigger `import.yaml` with the repository name.
- **Bulk**: trigger `bulk-import.yaml`; configs land in `importer_tmp_dir/` and are promoted to `repos/` when the resulting PR is merged (via `promote-imported-configs.yaml`).
- Repos listed in `config/import-config.yaml` under `ignore_repos` are skipped by the importer.

## Git & PR Workflow

- **Never add `Co-Authored-By` lines** to commit messages.
- When working in the `github-terraformer` repo, the local setup uses two remotes:
  - `origin` → `ljubon-org/github-terraformer` (your fork — push here)
  - `upstream` → `G-Research/github-terraformer` (public repo — never push here directly)
- Always commit and push to `origin` only, never to `upstream`.
- **Never commit or push directly to `gcss-1135--add-environment-with-deployment-policy`** — all changes to that branch must go through a PR.
- When changing any piece of code, update all docs where that code is referenced or mentioned.

## Key Config Concepts

- **Rulesets** (preferred) vs **branch protections** (v4, deprecated) — prefer rulesets for new configs.
- **Environments** support two deployment policies: `protected_branches` or `selected_branches_and_tags`.
- Bypass actors reference app IDs defined in `config/app-list.yaml`.
- Feature flag `features.github_environments: true` in `import-config.yaml` enables environment import.
