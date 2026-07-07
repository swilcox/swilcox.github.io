# Agent Instructions

## Project Overview

This repository is a Hugo static site for `swilcox.github.io`.

- Site config lives in `hugo.toml`.
- Published content lives under `content/`, primarily `content/post/`.
- The active theme is `themes/github-style`.
- `themes/scw1` appears to be an additional/local theme and is not active unless `hugo.toml` is changed.
- `public/` is generated build output.

## Editing Guidelines

- Keep changes scoped to the requested task.
- Do not edit `public/` unless the task explicitly asks for generated output to be updated.
- Do not commit or clean unrelated untracked files.
- Prefer editing source content, layouts, theme assets, or config files over generated files.
- Use TOML front matter for posts, matching the existing `+++` style.
- Preserve the existing casual writing voice in blog posts.
- Keep filenames lowercase and hyphenated or underscored, following nearby content.

## Content Conventions

New posts should generally use:

```toml
+++
title = 'Post Title'
date = 2026-07-07T12:00:00-05:00
draft = true
summary = ''
tags = []
+++
```

Use leaf bundles (`content/post/name/index.md`) when a post has colocated images or other assets. Use a plain Markdown file (`content/post/name.md`) for text-only posts.

The site has a custom `callout` shortcode available:

```go-html-template
{{< callout note >}}
Callout content.
{{< /callout >}}
```

Callout icons are in `static/images/callouts/`.

## Local Development

Run the site locally with:

```sh
hugo server --buildDrafts
```

Build the production site with:

```sh
hugo --gc --minify
```

The GitHub Pages workflow uses Hugo extended `0.128.0` and deploys the generated `public/` artifact from `.github/workflows/hugo.yaml`.

## Verification

Before finishing changes, run the narrowest useful check:

- For content-only changes, run `hugo`.
- For layout, theme, CSS, or shortcode changes, run `hugo --gc --minify`.
- If changing GitHub Actions, review `.github/workflows/hugo.yaml` and keep it aligned with Hugo/GitHub Pages requirements.

If Hugo is unavailable in the environment, state that verification could not be run.

