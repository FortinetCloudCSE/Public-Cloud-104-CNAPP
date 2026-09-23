# Reference — Public-Cloud-104-CNAPP

Detail doc for [CLAUDE.md](../../CLAUDE.md). Read when you need the full file map, current
`scripts/repoConfig.json` values, environment variables, or `static.yml` internals.

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

## `scripts/repoConfig.json` current values

`workshopTitle` is still the template default `"Hugo for Fortinet TECWorkshops"` and `author` is
`"Gabe O'Brien"`. The banner fields, however, are real: `logoBannerText`
`"Public Cloud 104 CNAPP"`, `bannerLine1/2/3` = `"Public Cloud 104"` / `"Intro to Cloud Native Security"` / `"with Lacework FortiCNAPP"`. `themeVariant` `"Xperts2025"`, `marketingCode`
`"XPerts25"`, `errorLevel` `"warning"`, `googleServicesID` `"G-5RZBH288ST"`, `shortcuts` empty.

## Environment Variables

None required for authoring. CI uses the workflow-scoped `GITHUB_TOKEN` (permissions
`contents: read`, `pages: write`, `id-token: write`) for the Pages deployment.

`CentralRepo/scripts/batch_repo_update.py` reads `GITHUB_TOKEN` from the environment — relevant
only if you run a template upgrade, which writes to `main` directly.

Optional locally: `DOCKER_CONTEXT` / `DOCKER_HOST` — fortihugorunner honors the active Docker
context.

## `.github/workflows/static.yml` internals

md5 `1583cc7583b39c3b43c152aec99fda94` is the consensus version — 12 other workshop checkouts on
this machine carry the byte-identical file. It was last changed here by `f665f29` (Robert Reris,
"updating GHA workflow for dev builds"). What it does now: triggers on push to `main` plus
`workflow_dispatch` with two inputs (`runner_type`: ubuntu-latest / self-hosted / org-runners;
`image_variant`: prod / dev). Default image is `public.ecr.aws/k4n6m5h8/fortinet-hugo:latest`; a
manual dispatch with `image_variant=dev` swaps in `hugotester:latest`. It pulls with exponential
backoff + jitter (7 attempts) to survive ECR `toomanyrequests`, runs the build container with the
workspace mounted at `/home/UserRepo`, copies `/home/CentralRepo/public` to `./docs`, prunes
images and builder cache, then uploads `./docs` and deploys via `actions/deploy-pages@v4`.
Concurrency group `pages`, `cancel-in-progress: false`.

## CentralRepo mount contract

The container holds the theme/config at `/home/CentralRepo`; your repo mounts at
`/home/UserRepo`. `hugo_build.sh` runs `hugo --minify --cleanDestinationDir --contentDir /home/UserRepo/content`, output landing in `/home/CentralRepo/public`. Local overrides would be
copied by `local_copy.sh` from `../UserRepo/layouts/{shortcodes,partials}` — this repo has none.

The Dockerfile pins CentralRepo to branches, not tags: dev stage `ADD ...CentralRepo.git#prreviewJune23`, prod stage `#main`. Theme changes on those branches land here
with no version bump. Both stages are otherwise identical (Alpine CA-cert HTTP workaround,
python3/py3-pip/tini-static, entrypoint `/home/CentralRepo/scripts/local_copy.sh`). There is no
`ARG LOCAL` switch, so there is no local-CentralRepo-checkout build path.

## Shortcodes in use

Only two shortcodes appear in content: `figure` and `quizframe`. `Cloud-101/_index.md` uses
`{{< figure src="img/... " >}}` 34 times.

The quiz is present but commented out. `content/_index.md` ends with an HTML comment wrapping
`{{< quizframe page="/gamebytag?tag=Before" height="800" width="100%" >}}` — so nothing renders
today. `quizframe` is a CentralRepo shortcode (`CentralRepo/layouts/shortcodes/quizframe.html`);
it builds the iframe src from site params `quizUrl` + `repoName` and appends `fortiemail` /
`fortiuser` cookies plus `workshopID`. Uncommenting it is all that is needed to re-enable. Git
history shows the quiz being toggled on and off repeatedly (`fix: enabled quiz` → `tmp: remove quiz`) — check with the owner before changing its state. The closing comment marker is `--->`,
not `-->`. Malformed but tolerated. Don't "fix" it blindly without re-rendering.

## Live remote branches (as of last audit)

`gabe/cloud101-content-from-old-lab-plus-quiz-test`, `gabe/feedback-from-dryrun-with-adam`,
`gabe/feedback-from-dryrun-with-adam-2`, `qwiklabs-images-05`, `qwiklabs-minor-updates`,
`jkopkoEdits`. Local branch `jkopkoEdits` last observed pointing at the exact same commit as
`origin/main` (`7c51951`) — zero local content changes, 18 commits ahead of the stale
`origin/jkopkoEdits`. Pushing it would just fast-forward that remote branch to main's content.
Check `git status`/`git log` before assuming there is work in flight — this list goes stale.
