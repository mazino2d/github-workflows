# github-workflows

Reusable GitHub Actions workflows and composite actions shared by the
`mazino2d/*` repositories. Each repository keeps only thin callers: triggers,
inputs and permissions. The logic lives here.

## Using it

Call a workflow at the `v1` tag. `v1` moves to every `v1.x.y` release, so
callers pick up fixes without a pull request; a breaking change ships as `v2`.

```yaml
jobs:
  ci:
    uses: mazino2d/github-workflows/.github/workflows/python-ci.yml@v1
    with:
      run: |
        ruff check src/ tests/
        pytest tests/
```

Every reusable workflow ends in a job called `gate`, which passes when every
other job succeeded or was skipped. Branch protection requires
`<caller job id> / gate` (for example `ci / gate`), so path-filtered jobs can be
skipped without leaving a required check missing.

Reusable workflows can only use the permissions their caller grants. Each
workflow lists what the calling job needs in its header comment.

## Workflows

| Workflow | What it does |
| --- | --- |
| [`pages.yml`](.github/workflows/pages.yml) | Builds a MkDocs or Jekyll site and deploys it to GitHub Pages (pull requests only build) |
| [`python-ci.yml`](.github/workflows/python-ci.yml) | Sets up Python, installs, runs check commands (one per line), with an optional path filter |
| [`node-ci.yml`](.github/workflows/node-ci.yml) | Sets up Node from `.nvmrc`, runs `npm ci` and check commands, with an optional path filter |
| [`pypi-publish.yml`](.github/workflows/pypi-publish.yml) | Builds and uploads to PyPI or TestPyPI with twine |
| [`docker.yml`](.github/workflows/docker.yml) | Builds one image, or every changed `<root>/*/Dockerfile`; tests inside the image; pushes to GHCR from the default branch |
| [`gitops-bump.yml`](.github/workflows/gitops-bump.yml) | Opens an auto-merging pull request that bumps an image tag in a GitOps values file |
| [`terraform.yml`](.github/workflows/terraform.yml) | Plans (with a sticky PR comment) or applies every changed Terraform stack |
| [`k8s-validate.yml`](.github/workflows/k8s-validate.yml) | Lints Helm charts and builds Kustomize overlays |
| [`lint.yml`](.github/workflows/lint.yml) | Runs actionlint and, optionally, markdownlint |
| [`codeql-actions.yml`](.github/workflows/codeql-actions.yml) | Runs a CodeQL scan of the repository's workflows |

## Composite actions

| Action | What it does |
| --- | --- |
| [`gate`](actions/gate/action.yml) | Fails unless every needed job succeeded or was skipped |
| [`path-match`](actions/path-match/action.yml) | Reports whether the event changed any of the given path globs |
| [`changed-dirs`](actions/changed-dirs/action.yml) | Lists the directories holding a marker file that the event touched, as a JSON matrix |
| [`sticky-comment`](actions/sticky-comment/action.yml) | Creates, updates or deletes one PR comment per marker |
| [`flutter-setup`](actions/flutter-setup/action.yml) | Installs Flutter and packages (cached), runs codegen and l10n |

## MkDocs sites

[`mkdocs/base.yml`](mkdocs/base.yml) holds the shared theme and Markdown
setup. A site's `mkdocs.yml` inherits it and keeps only what is its own:

```yaml
INHERIT: .mkdocs/base.yml
site_name: My Blog
site_url: https://mazino2d.github.io/my-repo/
nav:
  - Home: index.md
```

`pages.yml` fetches the base config into `.mkdocs/` before building. To build
locally, fetch it the same way (add `.mkdocs/` to `.gitignore`):

```sh
mkdir -p .mkdocs && curl -sSfL -o .mkdocs/base.yml \
  https://raw.githubusercontent.com/mazino2d/github-workflows/v1/mkdocs/base.yml
mkdocs serve
```

## Developing

- [`ci.yml`](.github/workflows/ci.yml) lints the repo, tests the composite
  actions from the current commit, and calls every reusable workflow against
  [`fixtures/`](fixtures).
- Reusable workflows reference the composite actions at `@v1`. A change to an
  action therefore reaches them only after it has been released.
- Use conventional commit messages. release-please opens the release PR, and
  merging it tags `vX.Y.Z` and moves `v1` to the same commit.
