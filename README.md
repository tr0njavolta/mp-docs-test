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

```console
git clone --recurse-submodules https://github.com/tr0njavolta/modelplane-docs.git
# or
git submodule update --init
```

Serve locally with live reload:

```console
nix run '.?submodules=1#serve'
```

Build the production site and run the checks CI runs:

```console
nix build '.?submodules=1#site'
nix flake check '.?submodules=1'
```

Nix ignores a flake's submodules unless asked, so `?submodules=1` is not
optional. Without it the build fails with an empty-submodule error rather than
publishing a site with no pages.

## Pulling in content changes

`.github/workflows/content.yml` bumps the submodule to the tip of the content
repo's `main` daily, and on demand. Merging that commit is what deploys new
prose. To do it by hand:

```console
git submodule update --remote modelplane
git commit -am "Update docs content"
```

## Rebuilding the JavaScript bundle

The bundle under `themes/geekboot/assets/js` is committed, so the site build
needs no Node step for it. After changing anything under `utils/webpack/src`:

```console
nix run '.?submodules=1#generate'
git diff themes/geekboot/assets/js
```
