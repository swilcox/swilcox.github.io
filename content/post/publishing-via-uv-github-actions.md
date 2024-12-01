+++
title = 'Publishing Python Packages Using uv and Github Actions'
date = 2024-12-01T11:59:09-06:00
summary = 'more automated releases to pypi using github actions and using uv'
draft = false
tags = ['python', 'uv', 'github', 'pypi']
+++
# Publishing Python Packages Using `uv` and Github Actions

## Overview
[uv](https://docs.astral.sh/uv/guides/publish/#building-your-package) supports `build` and `publish` commands now. But to avoid needing to keep track of a token from PyPI, it's possible to publish from Github Actions.

## Goals
1. Not have to keep track of a token for PyPI.
2. Ability to publish a "release" on github and have it automatically publish to PyPI.
3. Continue to use `uv`.

## Disclaimer

{{< callout warning >}}
This is just my personal use and I can't guarantee that this is *fully* right. Use at your own risk. Also, please let me know of any corrections!
{{< /callout >}}

## Workflow action

For my project [sigye](https://github.com/swilcox/sigye), I set up the following `publish.yml` workflow:

```yaml
name: Publish to PyPI

on:
  release:
    types: [published]

jobs:
  publish:
    name: Build and Publish to PyPI
    runs-on: ubuntu-latest
    environment: pypi
    permissions:
      id-token: write  # Required for trusted publishing

    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v4
        with:
          enable-cache: true
          cache-dependency-glob: "uv.lock"

      - name: Set up Python
        run: uv python install

      - name: Install build dependencies
        run: uv sync --all-extras
      
      - name: Build the project
        run: uv build

      - name: Publish to PyPI
        run: uv publish
```

Link to [current file](https://github.com/swilcox/sigye/blob/main/.github/workflows/publish.yml)

## Github Configuration

I added a new environment in Github called `pypi`. There are a bunch of additional security options on Github environments and I didn't change any of those. Again, more research necessary.

## PyPI Configuration

In PyPI, I had already published (manually), so I just had to go into that project and tell it I wanted to add a Trusted Publisher. It pretty much just is answering questions related to your repository name, environment name, and github account.
