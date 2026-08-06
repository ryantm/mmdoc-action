# mmdoc Action

Generate [mmdoc](https://github.com/ryantm/mmdoc) single-page and multi-page HTML documentation in a GitHub Actions workflow.

## Deploy to GitHub Pages

Your repository must contain a `toc.md` and the Markdown files it references. With no inputs, the action uses the repository name as the project name, reads Markdown from the repository root, and writes to `out`.

```yaml
name: Deploy documentation

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - id: docs
        uses: ryantm/mmdoc-action@v1
      - uses: actions/configure-pages@v6
      - uses: actions/upload-pages-artifact@v5
        with:
          path: ${{ steps.docs.outputs.path }}/multi

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v5
```

In the repository's **Settings → Pages**, set **Source** to **GitHub Actions** once before the first deployment. The workflow builds both sites and deploys `out/multi`; `out/single` remains available if you want to upload it separately.

## Build without deploying

```yaml
steps:
  - uses: actions/checkout@v7
  - uses: ryantm/mmdoc-action@v1
```

The generated sites are available in `out/multi` and `out/single`.

## Inputs

All inputs are optional and correspond to mmdoc's CLI arguments.

| Input | Default | mmdoc argument | Description |
| --- | --- | --- | --- |
| `project-name` | Repository name | `PROJECT-NAME` | Project or site name |
| `src` | `.` | `SRC` | Directory containing `toc.md` and Markdown files |
| `out` | `out` | `OUT` | Generated output directory |

The action exposes the absolute generated directory as the `path` output.

### Configure every option

```yaml
- id: docs
  uses: ryantm/mmdoc-action@v1
  with:
    project-name: Example Project
    src: docs
    out: public

- run: echo "Generated documentation at ${{ steps.docs.outputs.path }}"
```

## mmdoc version updates

The action pins the mmdoc version recorded in [`MMDOC_VERSION`](MMDOC_VERSION). A daily workflow checks the latest `ryantm/mmdoc` release and opens a pull request when an update is available. Dependabot separately updates the GitHub Actions used by this repository.

## Runner support

The action installs Nix and works on GitHub-hosted Linux and macOS runners.

## License

[CC0 1.0 Universal](LICENSE)
