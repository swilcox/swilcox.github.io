+++
title = 'Sigye'
date = 2024-11-23T10:08:01-06:00
draft = true
summary = "simple time tracking on the command line"
tags = ["python", "ai"]
+++
# Sigye - Time Tracking via CLI
## Overview
* Why: Even though I don't have to track time for billing purposes for my job, I thought it would be good to hold myself accountable and also have a record of how much time I spent on what.
* What: I wanted a CLI tool that was quick and low barrier to recording time. I did look at a few existing tools like [timetrace](https://github.com/dominikbraun/timetrace) but wanted something even simpler.
* Additional Goals:
  * some [DDD](https://en.wikipedia.org/wiki/Domain-driven_design) practice.
  * experiment with a little bit of translation and localization.
  * ability to support multiple storage backends.

## Lessons Learned
* more use of AI in code generation (via CLINE and Claude artifacts and github copilot).
* use of [rich](https://rich.readthedocs.io/en/stable/introduction.html).
* po/mo files [internationalization](https://docs.python.org/3/library/gettext.html).
* [publishing packages using uv]({{< ref "publishing-via-uv-github-actions" >}}).
* deeper understanding of [click](https://click.palletsprojects.com/en/stable/).
* use of [peewee](https://github.com/coleifer/peewee) for SQLite access.
* use of [ryaml]() and [rtoml]() for faster json, yaml and toml writing and reading.

## User Experience

As something I leverage every workday, I wanted and have adapted the project to 