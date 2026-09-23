# CLAUDE.md — Public-Cloud-104-CNAPP

> Global preferences (planning workflow, code quality, operations): `~/.claude/CLAUDE.md`
>
> On-demand docs — read only when relevant, no cost otherwise:
> | Doc | Read when |
> |-----|-----------|
> | [plans/claude/reference.md](plans/claude/reference.md) | full file map, `repoConfig.json` values, env vars, `static.yml`/Dockerfile/CentralRepo mount internals, live branch list |
> | [plans/claude/gotchas.md](plans/claude/gotchas.md) | touching content pages, images, the quiz, `Jenkinsfile`/`fdevsec.yaml`, or predicting a template upgrade's effect |
> | [plans/README.md](plans/README.md) | writing a plan/spec/log file — naming, lifecycle, why not `docs/plans/` |

## Project in One Line

A FortinetCloudCSE hands-on workshop — currently a single "Cloud 101" lab (AWS account via
Qwiklabs, EC2, security groups, IAM role) plus a separate Qwiklabs login walkthrough — published
as a Hugo static site to GitHub Pages. Content-only; the labs themselves run in Qwiklabs.

## Stack Quick Reference

| Layer | Tech | Port |
|-------|------|------|
| Site generator | Hugo (`hugomods/hugo:std`) via `public.ecr.aws/k4n6m5h8/fortinet-hugo:latest` | 1313 (local dev) |
| Site theme/config/layouts | [CentralRepo](https://github.com/FortinetCloudCSE/CentralRepo) — mounted at build time, **not** in this repo | — |
| Local dev driver | [fortihugorunner](https://github.com/FortinetCloudCSE/fortihugorunner) CLI | — |
| Quiz backend | Cloud Run service (`quizUrl` in `scripts/repoConfig.json`) | — |
| Hosting | GitHub Pages (`https://fortinetcloudcse.github.io/Public-Cloud-104-CNAPP/`) | — |

No `layouts/`, no `hugo.toml`, no `config.toml`, no `docs/` (source), no `terraform/`, no test
suite. Full file map: [reference.md](plans/claude/reference.md).

## Build & Run Commands

```bash
# Preview the site locally (requires Docker + fortihugorunner on PATH)
fortihugorunner pull-image --env author-dev
fortihugorunner launch-server \
  --docker-image fortinet-hugo:latest \
  --host-port 1313 --container-port 1313 --watch-dir .
# open http://localhost:1313

# Reproduce the CI static build exactly
docker run --rm -v "$PWD:/home/UserRepo" fortinet-hugo:latest build
```

There is no test suite. Content changes are validated by rendering locally.

## Critical Rules

- **Never put plan/log/spec files in `docs/`.** It's doubly machine-owned: gitignored, and the
  template upgrade tool (`CentralRepo/scripts/batch_repo_update.py`, commits **directly to
  `main`**) deletes everything under `docs/` on every upgrade. **Use root-level `plans/`
  instead** — see [plans/README.md](plans/README.md) for naming/lifecycle.
- **`docs/` is build output, not source** — CI `rm -rf docs`, copies the Hugo build in, uploads
  as the Pages artifact. Never commit it.
- **No `hugo.toml`/`config.toml` here, on purpose** — `repo_upgrade_spec.json` deletes both on
  upgrade; the TOML is generated in-container from `scripts/repoConfig.json`. That file is the
  only site-chrome config in the repo.
- **No `layouts/` directory** — every shortcode resolves from the CentralRepo theme.
- **`errorLevel` is `"warning"`** — Hugo warnings never fail the build; broken links/images ship
  silently. Don't rely on CI to catch them.
- **`Dockerfile` and `.github/workflows/static.yml` are template-managed** — `batch_repo_update.py`
  overwrites both from the operator's CentralRepo checkout and pushes to `main`; hand-edits here
  get lost. Fix them in CentralRepo, not here.
- **Deploy triggers only on push to `main`** (or manual `workflow_dispatch`) — nothing on a
  feature branch publishes.
- **`package.json`'s `hugo` script is stale** (mounts nonexistent `config.toml`/`layouts/`) —
  never use it; use `fortihugorunner` or the `docker run` command above.

## Environment Variables

None required for authoring. CI uses the workflow-scoped `GITHUB_TOKEN`. Full detail:
[reference.md](plans/claude/reference.md#environment-variables).

## Common Tasks

**Add a workshop section**: page bundle under `content/` (mirror `Cloud-101/`) with `title`,
`linkTitle`, `weight` front matter + `img/`; preview with `launch-server`. Detail:
[gotchas.md](plans/claude/gotchas.md#common-tasks--worked-detail).

**Write a plan or session log**: root-level `plans/`, never `docs/plans/`. See
[plans/README.md](plans/README.md).

**Change site chrome** (title, banner, theme variant, analytics, quiz URL): edit
`scripts/repoConfig.json` — the only config file in the repo. Current values:
[reference.md](plans/claude/reference.md#scriptsrepoconfigjson-current-values).

**Re-enable the quiz**: uncomment the `quizframe` block at the bottom of `content/_index.md`.
Confirm with the owner first. Detail: [gotchas.md](plans/claude/gotchas.md#common-tasks--worked-detail).

**Publish current work**: merge into `main`. Nothing deploys from a feature branch.

**Debug a broken published page**: run the CI build command locally — detail:
[gotchas.md](plans/claude/gotchas.md#common-tasks--worked-detail).
