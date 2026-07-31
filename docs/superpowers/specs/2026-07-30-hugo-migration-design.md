# Hugo migration — ubermuda.xyz

**Date:** 2026-07-30
**Status:** Approved, pending implementation

## Goal

Replace the hand-written `index.html` with a Hugo site that manages a landing page
and dated blog posts, deployed to GitHub Pages via GitHub Actions on the existing
custom domain `ubermuda.xyz`.

The current visual design is preserved exactly. This migration changes how the site
is built, not how it looks.

## Starting state

- `main` holds two tracked files: `index.html` (a self-contained page, ~160 lines,
  inline CSS and JS) and `CNAME` (`ubermuda.xyz`).
- GitHub Pages: `build_type: legacy`, source `main` / root, custom domain
  `ubermuda.xyz`, certificate approved, HTTPS enforced.
- Local Hugo is 0.160.1; latest is 0.164.0. No Go toolchain, no Node, no SCSS.

## Decisions

| Decision | Choice | Reason |
|---|---|---|
| Content types | Landing page + `posts/` | What the user wants to publish now |
| Theme | None — custom layouts | The existing design is deliberate and small enough to own outright |
| Deploy | GitHub Actions, Pages source = Actions | Official Hugo path; nothing generated in git |
| Hugo version | 0.164.0, pinned in both CI and local | CI must build what was authored locally |
| `CNAME` file | Left tracked and untouched | Inert under an Actions source, but documents intent and makes a revert to branch-deploy work unchanged |
| Sample post | None | The user writes the first real post |
| Git flow | `hugo-migration` branch → PR | Normal feature work; the direct-to-main exception was for the empty-repo bootstrap only |

## Structure

```
hugo.toml
content/
  _index.md            homepage intro copy
  posts/_index.md      section title
layouts/
  baseof.html          head, CSS, theme-toggle JS, <main> wrapper
  index.html           intro + link nav + recent posts
  list.html            /posts/ index
  single.html          one post
  partials/
    head.html
    header.html        ~/ubermuda.xyz + theme toggle
    nav.html           social links, driven by config
static/
CNAME
.gitignore
.github/workflows/hugo.yaml
docs/superpowers/specs/
```

`publishDir` stays at its default `public/`. It must never be set to `docs/`, which
holds this spec.

A `.gitignore` is added covering `public/`, `.hugo_build.lock`, and `.serena/`.
Without it the first local build drops the entire generated site into `git status`,
and anything committed here is published to a public site.

## Templates

`baseof.html` carries the existing CSS block verbatim — the `--bg`/`--fg`/`--mut`/
`--line`/`--acc` custom properties, both theme blocks, the `fadeUp`/`fadeIn`
keyframes, the `prefers-reduced-motion` guard, and the responsive breakpoint. The
theme-toggle script (localStorage key `ubermuda-theme`, `prefers-color-scheme`
fallback) moves across unchanged.

Post pages extend the same design language rather than introducing a new one:

- `list.html` reuses the `nav a` row pattern — date in the fixed-width `<b>` column,
  title beside it, same hover inversion to `--acc`.
- `single.html` adds a date line in `--mut` and prose spacing consistent with the
  existing `.intro p` rules (18px, 1.65 line-height, 640px measure).

Social links move out of hardcoded markup into `hugo.toml` params so `nav.html`
renders them from config.

## Pipeline

`.github/workflows/hugo.yaml`, adapted from the official Hugo workflow and stripped
to what this site actually uses:

- Removed: Go setup, Node setup, npm install, Dart Sass install. There is no
  `go.mod`, no `package-lock.json`, and no SCSS. Keeping install steps that cannot
  be reproduced or verified locally adds failure surface for no benefit.
- Kept: checkout, `configure-pages`, Hugo install pinned to 0.164.0, build cache,
  `hugo build --gc --minify --baseURL`, `upload-pages-artifact`, `deploy-pages`.
- Triggers on push to `main` plus `workflow_dispatch`.
- Permissions: `contents: read`, `pages: write`, `id-token: write`.

Pushing this file requires a token with `workflow` scope. The active `GH_TOKEN`
environment variable lacks it; the keyring `ubermuda` login has it. All git and `gh`
operations run with `env -u GH_TOKEN`.

## Risks and verification

Two failure modes here are silent — the site renders correctly while being wrong.
Both get explicit checks.

**1. Wrong `baseURL`.** The workflow builds with
`--baseURL "${{ steps.pages.outputs.base_url }}/"`. With a custom domain configured,
`configure-pages` should emit `https://ubermuda.xyz`. If it instead emits
`https://ubermuda.github.io/ubermuda.xyz`, every absolute link and the RSS feed get
the wrong prefix and nothing errors.

- Mitigation: `hugo.toml` sets `baseURL = 'https://ubermuda.xyz/'` so any non-CI
  build is correct regardless.
- Check: read the `baseURL` line in the first CI build log, and confirm one absolute
  URL in the deployed `/index.xml`.

**2. Pages source flip disturbs the custom domain.** Switching from `legacy` to
`workflow` build type could drop `cname`. GitHub's docs confirm the `CNAME` file is
ignored under an Actions source, so the domain lives in repo settings alone.

- Check, immediately after the flip:
  ```
  env -u GH_TOKEN gh api repos/ubermuda/ubermuda.xyz/pages \
    --jq '{build_type, cname, https_enforced, cert: .https_certificate.state}'
  ```
  `cert: "approved"` surviving is the signal that matters.
- Recovery: if `cname` drops to null, re-add it. The certificate is already issued
  for this domain, so reissue is fast. This was the failure hit during initial setup:
  setting a custom domain before DNS resolves means GitHub never queues a cert.

**3. No sample post.** With `content/posts/` empty, `list.html` and `single.html`
ship unexercised. Before merging, verify both by creating a throwaway post locally,
viewing it under `hugo server`, and deleting it. It must not be committed.

## Done when

- `https://ubermuda.xyz` serves the Hugo-built page, visually identical to the
  current `index.html`.
- `http://` and `www.` still 301 to the canonical apex URL.
- Pages API reports `build_type: workflow`, `cname: ubermuda.xyz`,
  `https_enforced: true`, `cert: approved`.
- The CI build log shows `baseURL` as `https://ubermuda.xyz/`.
- `index.html` is deleted from the repo root.
