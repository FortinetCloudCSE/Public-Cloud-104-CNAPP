# CLAUDE.md — Public-Cloud-104-CNAPP

> Global preferences (planning workflow, code quality, operations): `~/.claude/CLAUDE.md`

## Project in One Line

A FortinetCloudCSE hands-on workshop — currently a single "Cloud 101" lab (AWS account via Qwiklabs, EC2, security groups, IAM role) plus a separate Qwiklabs login walkthrough — published as a Hugo static site to GitHub Pages. Content-only; the labs themselves run in Qwiklabs.

## Stack Quick Reference

| Layer | Tech | Port |
|-------|------|------|
| Site generator | Hugo (`hugomods/hugo:std`) via `public.ecr.aws/k4n6m5h8/fortinet-hugo:latest` | 1313 (local dev) |
| Site theme/config/layouts | [CentralRepo](https://github.com/FortinetCloudCSE/CentralRepo) — mounted at build time, **not** in this repo | — |
| Local dev driver | [fortihugorunner](https://github.com/FortinetCloudCSE/fortihugorunner) CLI | — |
| Quiz backend | Cloud Run service (`quizUrl` in `scripts/repoConfig.json`) | — |
| Hosting | GitHub Pages (`https://fortinetcloudcse.github.io/Public-Cloud-104-CNAPP/`) | — |

## Key File Map

```
content/                            — only THREE markdown files exist
  _index.md                         — home page (archetype "home"), weight 1
  Cloud-101/
    _index.md                       — the entire lab, ~293 lines in one file, weight 1
    img/                            — 34 images shared by Cloud-101/_index.md
    qwiklabs/
      index.md                      — Qwiklabs login walkthrough (NO front matter)
      *.png                         — 4 images beside the page, referenced by absolute prod URL
  unusedimages/01Cloud101/img/      — 19 parked PNGs, tracked, referenced by nothing
scripts/repoConfig.json             — the ONLY site config in this repo
Dockerfile                          — local Hugo image build (dev → CentralRepo#prreviewJune23, prod → #main)
Jenkinsfile                         — content-lint pipeline (warn-only)
fdevsec.yaml                        — FortiDevSec scan config, real org/app IDs filled in
.github/workflows/static.yml        — build + deploy to Pages on push to main
repo_upgrade_spec.json              — template sync spec, version Hugo-v2.1
.repo_upgrade_version               — single line: `Hugo-v2.1`
migration_log.csv
migration_log_dry_run_20250909_162614.csv
migration_log_run_20250909_162642.csv
```

No `layouts/`, no `hugo.toml`, no `config.toml`, no `docs/`, no `terraform/`, no test suite.

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

## Critical Patterns & Gotchas

- **Never put plan/log/spec files in `docs/`.** `docs/` is doubly machine-owned: `.gitignore` line 3 is `docs/`, and `repo_upgrade_spec.json` has `"folders_to_delete": ["docs"]` — the template upgrade tool (`CentralRepo/scripts/batch_repo_update.py`, which commits **directly to `main`** over the GitHub API with `FOLDERS_TO_DELETE = ["docs"]` hardcoded) deletes every blob under `docs/` wholesale. The global `docs/plans/` convention does not apply here. **Use a root-level `plans/` directory instead** — inert to Hugo, not gitignored, and outside `folders_to_delete`.
- **`docs/` is build output, not source.** CI does `rm -rf docs`, copies `/home/CentralRepo/public` out of the container into it, and uploads that as the Pages artifact. Never commit it.
- **CentralRepo mount contract:** the container holds the theme/config at `/home/CentralRepo`; your repo mounts at `/home/UserRepo`. `hugo_build.sh` runs `hugo --minify --cleanDestinationDir --contentDir /home/UserRepo/content`, output landing in `/home/CentralRepo/public`. Local overrides would be copied by `local_copy.sh` from `../UserRepo/layouts/{shortcodes,partials}` — this repo has none.
- **No `hugo.toml` or `config.toml` here — on purpose.** `repo_upgrade_spec.json` explicitly lists both under `files_to_delete`; the TOML is generated inside the container from `scripts/repoConfig.json`. Site chrome lives in **`scripts/repoConfig.json`** only.
- **No `layouts/` directory.** Every shortcode resolves from the CentralRepo theme. Adding one means creating `layouts/shortcodes/` from scratch — note `repo_upgrade_spec.json` deletes `layouts/shortcodes/FTNThugoFlow.html` on upgrade.
- **`scripts/repoConfig.json` current values:** `workshopTitle` is still the template default `"Hugo for Fortinet TECWorkshops"` and `author` is `"Gabe O'Brien"`. The banner fields, however, are real: `logoBannerText` `"Public Cloud 104 CNAPP"`, `bannerLine1/2/3` = `"Public Cloud 104"` / `"Intro to Cloud Native Security"` / `"with Lacework FortiCNAPP"`. `themeVariant` `"Xperts2025"`, `marketingCode` `"XPerts25"`, `errorLevel` `"warning"`, `googleServicesID` `"G-5RZBH288ST"`, `shortcuts` empty.
- **`errorLevel` is `"warning"`, so Hugo warnings never fail the build.** Broken links and missing images ship silently.
- **Only two shortcodes appear in content: `figure` and `quizframe`.** `Cloud-101/_index.md` uses `{{< figure src="img/... " >}}` 34 times.
- **The quiz is present but commented out.** `content/_index.md` ends with an HTML comment wrapping `{{< quizframe page="/gamebytag?tag=Before" height="800" width="100%" >}}` — so nothing renders today. `quizframe` is a CentralRepo shortcode (`CentralRepo/layouts/shortcodes/quizframe.html`); it builds the iframe src from site params `quizUrl` + `repoName` and appends `fortiemail` / `fortiuser` cookies plus `workshopID`. Uncommenting it is all that is needed to re-enable. Git history shows the quiz being toggled on and off repeatedly (`fix: enabled quiz` → `tmp: remove quiz`) — check with the owner before changing its state.
- **The closing comment marker is `--->`, not `-->`.** Malformed but tolerated. Don't "fix" it blindly without re-rendering.
- **`content/Cloud-101/qwiklabs/index.md` has no front matter at all** — no `title`, no `weight`. Hugo derives its title from the filename/path. If ordering or the sidebar label matters, front matter has to be added.
- **That same page hardcodes absolute production URLs for its images** (`https://fortinetcloudcse.github.io/Public-Cloud-104-CNAPP/cloud-101/qwiklabs/*.png`) even though the PNGs sit next to it in the bundle. Locally the images load from prod, not from your working copy — edits to those files will not show up in preview.
- **Page ordering is `weight` in front matter,** not filename. Both `_index.md` files are `weight: 1`.
- **`content/unusedimages/01Cloud101/img/` still exists and is tracked** (19 PNGs: terraform, EC2 instance-connect, security screenshots). Referenced by no page. `01Cloud101` is a stale name — the live section is `Cloud-101`. Don't delete or rewire without asking.
- **`.github/workflows/static.yml` (md5 `1583cc7583b39c3b43c152aec99fda94`) is the consensus version** — 12 other workshop checkouts on this machine carry the byte-identical file. It was last changed here by `f665f29` (Robert Reris, "updating GHA workflow for dev builds"). What it does now: triggers on push to `main` plus `workflow_dispatch` with two inputs (`runner_type`: ubuntu-latest / self-hosted / org-runners; `image_variant`: prod / dev). Default image is `public.ecr.aws/k4n6m5h8/fortinet-hugo:latest`; a manual dispatch with `image_variant=dev` swaps in `hugotester:latest`. It pulls with exponential backoff + jitter (7 attempts) to survive ECR `toomanyrequests`, runs the build container with the workspace mounted at `/home/UserRepo`, copies `/home/CentralRepo/public` to `./docs`, prunes images and builder cache, then uploads `./docs` and deploys via `actions/deploy-pages@v4`. Concurrency group `pages`, `cancel-in-progress: false`.
- **`Dockerfile` and `.github/workflows/static.yml` are template-managed.** `batch_repo_update.py` overwrites both from the **operator's local CentralRepo checkout** (`FILES_TO_COPY`) and pushes straight to `main` — hand-edits here get lost. Fix them in CentralRepo, not here.
- **`repo_upgrade_spec.json` is documentation, not the executed contract.** `batch_repo_update.py` never reads it — the script's own hardcoded `FILES_TO_COPY` / `FILES_TO_DELETE` / `FOLDERS_TO_DELETE` constants are what run, and the two can drift silently. Read the script, not the spec, when predicting what an upgrade will do.
- **A template upgrade would REGRESS this repo's `static.yml`.** The consensus version here discards the build container's exit code (`docker wait` with no status capture), so a failed Hugo build still deploys. `ai-101` carries an improved copy that traps cleanup, captures the status, echoes `docker logs`, and fails the job with `::error::` — worth pulling upstream into CentralRepo rather than leaving it in one repo.
- **The Dockerfile pins CentralRepo to branches, not tags:** dev stage `ADD ...CentralRepo.git#prreviewJune23`, prod stage `#main`. Theme changes on those branches land here with no version bump. Both stages are otherwise identical (Alpine CA-cert HTTP workaround, python3/py3-pip/tini-static, entrypoint `/home/CentralRepo/scripts/local_copy.sh`). There is no `ARG LOCAL` switch, so there is no local-CentralRepo-checkout build path.
- **`Jenkinsfile` is warn-only.** It loops `content/*/` looking for any file matching `discussion|questions|q&a`, prints "sections not found" and echoes a warning, catches all exceptions, and reports build status to GitHub via `GitHubCommitStatusSetter`. It never fails on missing sections.
- **`fdevsec.yaml` is fully configured** — real `org` and `app` UUIDs, scanners `sast, secret, sca, iac, container`, `serial_scan: false`, `fail_pipeline.risk_rating: 7`. No placeholder to fill in.
- **`package.json` / `package-lock.json` are tracked despite being listed in `.gitignore`** (lines 6–7 — gitignore does not untrack existing files). `package.json`'s `hugo` script is stale: it mounts `config.toml` and `layouts/`, neither of which exists here. Do not use it; use `fortihugorunner` or the `docker run` command above.
- **Only `static.yml` exists in `.github/workflows/`.** Sibling repos (ai-101, AWS-FGT-301, faig-training-workshop) also carry `lacework-code-security-pr.yml` and `codex-advisory-review.yml`. PRs here get no automated security review.
- **Deploy triggers only on push to `main`** (or a manual dispatch), so nothing on a feature branch publishes.
- **Local branch is `jkopkoEdits` but it currently points at the exact same commit as `origin/main` (`7c51951`)** — zero local content changes, and it is 18 commits ahead of the stale `origin/jkopkoEdits`. Pushing it would just fast-forward that remote branch to main's content. Check `git status` before assuming there is work in flight.
- **Multiple authors work this repo concurrently.** Live remote branches: `gabe/cloud101-content-from-old-lab-plus-quiz-test`, `gabe/feedback-from-dryrun-with-adam`, `gabe/feedback-from-dryrun-with-adam-2`, `qwiklabs-images-05`, `qwiklabs-minor-updates`, `jkopkoEdits`.

## Environment Variables

None required for authoring. CI uses the workflow-scoped `GITHUB_TOKEN` (permissions `contents: read`, `pages: write`, `id-token: write`) for the Pages deployment.

`CentralRepo/scripts/batch_repo_update.py` reads `GITHUB_TOKEN` from the environment — relevant only if you run a template upgrade, which writes to `main` directly.

Optional locally: `DOCKER_CONTEXT` / `DOCKER_HOST` — fortihugorunner honors the active Docker context.

## Common Tasks

**Add a workshop section**: create a page bundle under `content/` (mirror `Cloud-101/`) with `title`, `linkTitle`, `weight` front matter and an `img/` subdir; preview with `launch-server`. Note `content/Cloud-101/_index.md` is one 293-line page — splitting it is a real refactor, not an append.

**Write a plan or session log**: put it in root-level **`plans/`** as `NNNN_YYYY-MM-DD_<git-username>_<slug>.md`, never `docs/plans/` — see the first gotcha above. `NNNN` is a per-repo sequence; the session log is optional; on completion, durable facts get promoted into this file and the plan is left to decay. See `plans/README.md`.

**Change site chrome** (title, banner, theme variant, analytics, quiz URL): edit `scripts/repoConfig.json`. That is the only config file in the repo.

**Re-enable the quiz**: uncomment the `quizframe` block at the bottom of `content/_index.md`. Confirm with the owner first — it has been toggled on and off several times.

**Publish current work**: merge into `main`. Nothing deploys from a feature branch.

**Debug a broken published page**: run the CI build command locally (`docker run --rm -v "$PWD:/home/UserRepo" fortinet-hugo:latest build`) — the dev server is more forgiving than the static build, and `errorLevel: "warning"` means Hugo will not fail on missing images or broken refs either way.
