# Modelplane docs site

The Hugo project behind [docs.modelplane.ai](https://docs.modelplane.ai):
config, layouts, the geekboot theme, and the CSS and JavaScript pipelines.

The prose is not here. Content, the example manifests the pages embed, and the
API definitions the reference is generated from all live in the
[modelplane](https://github.com/tr0njavolta/mp-fork) repo, checked out as the
`modelplane/` submodule and mounted into the site by `hugo.toml`. Edit pages
there, not here.

## Working on it

Clone with the submodule, or init it after the fact:

```bash
git clone --recurse-submodules https://github.com/tr0njavolta/mp-docs-test.git
# or
git submodule update --init
```

Two loops, depending on what you're changing.

**Editing prose or a layout** — live reload, rebuilds on save:

```bash
nix run '.?submodules=1#serve'      # http://localhost:1313
```

This serves `main` alone, at the root, because the submodule is the only content
on disk. The version switcher and the older-version banners don't render here —
there is only one version to switch between, and the paths they point at don't
exist locally. That's deliberate, not a bug.

**Checking anything version-related** — builds every version and serves the
artifact exactly as it deploys, latest at `/` and the rest under their prefixes:

```bash
nix run '.?submodules=1#preview'    # http://localhost:1313, or pass a port
```

Slower, no live reload, and the only way to see the switcher and banners work.

**Before pushing:**

```bash
nix flake check '.?submodules=1'    # builds every version, link-checks them all
```

Nix ignores a flake's submodules unless asked, so `?submodules=1` is not
optional. Without it the build fails with an empty-submodule error rather than
producing an empty site.

> Running `hugo` directly in `nix develop` panics with `Cannot find module
> 'postcss-lightningcss'`. Bare `hugo` defaults to the production environment,
> which runs the PostCSS pipeline against a `node_modules` that only the Nix
> build provides. Use the commands above; `hugo server` builds in development
> and skips it.

## Versions

Every version of the docs is built from this one branch and served from one
Vercel project, each under its own path prefix:

| Path | Version | Content comes from |
|---|---|---|
| `/` | the latest release | the `content-0-3` flake input |
| `/v0.2/` | an archived release | the `content-0-2` flake input |
| `/main/` | the dev build | the `modelplane/` submodule |

The latest release is served at the root itself, not redirected to, and each new
release takes that position over. `data/versions.json` is the list, read by both
`nix/docs.nix` and the version dropdown so they can't drift; the latest entry is
the one with an empty `path`.

Shipping X.Y means: add its flake input, set `latest`, give X.Y an empty `path`,
and move the release it replaces to `vX.Y`.

There are no release branches in this repo and no per-version Vercel project.
Adding a version is one flake input plus one line in `data/versions.json`:

```bash
nix flake lock --override-input content-0-4 \
  github:tr0njavolta/mp-content-test/release-0.4
```

A theme or layout fix therefore reaches every archived version on the next
build, which per-branch builds could never do.

## Pulling in content changes

`.github/workflows/content.yml` bumps every pin - the submodule for main, and
each archived version's flake input - to the tip of the branch it tracks, and
opens a pull request. Merging it is what publishes. Run it early with:

```bash
gh workflow run content.yml --repo tr0njavolta/mp-docs-test
```

By hand:

```bash
git submodule update --remote modelplane   # main
nix flake update content-0-3               # one archived version
```

## Rebuilding the JavaScript bundle

The bundle under `themes/geekboot/assets/js` is committed, so the site build
needs no Node step for it. After changing anything under `utils/webpack/src`:

```console
nix run '.?submodules=1#generate'
git diff themes/geekboot/assets/js
```
