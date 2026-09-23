# Gotchas — Public-Cloud-104-CNAPP

Detail doc for [CLAUDE.md](../../CLAUDE.md). Read when touching content pages, the quiz,
`repoConfig.json`, images, or before predicting what a template upgrade will do.

- **`content/Cloud-101/qwiklabs/index.md` has no front matter at all** — no `title`, no `weight`.
  Hugo derives its title from the filename/path. If ordering or the sidebar label matters, front
  matter has to be added.
- **That same page hardcodes absolute production URLs for its images**
  (`https://fortinetcloudcse.github.io/Public-Cloud-104-CNAPP/cloud-101/qwiklabs/*.png`) even
  though the PNGs sit next to it in the bundle. Locally the images load from prod, not from your
  working copy — edits to those files will not show up in preview.
- **Page ordering is `weight` in front matter,** not filename. Both `_index.md` files are
  `weight: 1`.
- **`content/unusedimages/01Cloud101/img/` still exists and is tracked** (19 PNGs: terraform, EC2
  instance-connect, security screenshots). Referenced by no page. `01Cloud101` is a stale name —
  the live section is `Cloud-101`. Don't delete or rewire without asking.
- **`repo_upgrade_spec.json` is documentation, not the executed contract.** It documents
  `"files_to_delete"` and `"folders_to_delete": ["docs"]`, but `batch_repo_update.py` never reads
  it — the script's own hardcoded `FILES_TO_COPY` / `FILES_TO_DELETE` / `FOLDERS_TO_DELETE =
  ["docs"]` constants are what run, and the two can drift silently. Read the script, not the
  spec, when predicting what an upgrade will do.
- **No `layouts/` directory** — every shortcode resolves from the CentralRepo theme. Adding one
  means creating `layouts/shortcodes/` from scratch — note `repo_upgrade_spec.json` deletes
  `layouts/shortcodes/FTNThugoFlow.html` on upgrade.
- **A template upgrade would REGRESS this repo's `static.yml`.** The consensus version here
  discards the build container's exit code (`docker wait` with no status capture), so a failed
  Hugo build still deploys. `ai-101` carries an improved copy that traps cleanup, captures the
  status, echoes `docker logs`, and fails the job with `::error::` — worth pulling upstream into
  CentralRepo rather than leaving it in one repo.
- **`Jenkinsfile` is warn-only.** It loops `content/*/` looking for any file matching
  `discussion|questions|q&a`, prints "sections not found" and echoes a warning, catches all
  exceptions, and reports build status to GitHub via `GitHubCommitStatusSetter`. It never fails
  on missing sections.
- **`fdevsec.yaml` is fully configured** — real `org` and `app` UUIDs, scanners `sast, secret, sca, iac, container`, `serial_scan: false`, `fail_pipeline.risk_rating: 7`. No placeholder to
  fill in.
- **`package.json` / `package-lock.json` are tracked despite being listed in `.gitignore`**
  (lines 6–7 — gitignore does not untrack existing files). `package.json`'s `hugo` script is
  stale: it mounts `config.toml` and `layouts/`, neither of which exists here. Do not use it; use
  `fortihugorunner` or the `docker run` command in CLAUDE.md.
- **Only `static.yml` exists in `.github/workflows/`.** Sibling repos (ai-101, AWS-FGT-301,
  faig-training-workshop) also carry `lacework-code-security-pr.yml` and
  `codex-advisory-review.yml`. PRs here get no automated security review.
- **Multiple authors work this repo concurrently** — see reference.md's live-branch list before
  assuming a branch is yours alone or stale.

## Common Tasks — worked detail

**Add a workshop section**: create a page bundle under `content/` (mirror `Cloud-101/`) with
`title`, `linkTitle`, `weight` front matter and an `img/` subdir; preview with `launch-server`.
Note `content/Cloud-101/_index.md` is one 293-line page — splitting it is a real refactor, not an
append.

**Re-enable the quiz**: uncomment the `quizframe` block at the bottom of `content/_index.md`.
Confirm with the owner first — it has been toggled on and off several times (see reference.md).

**Debug a broken published page**: run the CI build command locally (`docker run --rm -v
"$PWD:/home/UserRepo" fortinet-hugo:latest build`) — the dev server is more forgiving than the
static build, and `errorLevel: "warning"` means Hugo will not fail on missing images or broken
refs either way.
