+++
title = 'Sigye'
date = 2025-04-11T10:08:01-06:00
draft = false
summary = "simple time tracking on the command line"
tags = ["python", "ai"]
+++
# Sigye - Time Tracking via CLI

## TL;DR
Introducing [Sigye](https://github.com/swilcox/sigye) -- a CLI program for tracking your time.

## Overview
* Why: Even though I don't have to track time for billing purposes for my job, I thought it would be good to hold myself accountable and also have a record of how much time I spent on what.
* What: I wanted a CLI tool that was quick and low barrier to recording time. I did look at a few existing tools like [timetrace](https://github.com/dominikbraun/timetrace) but wanted something even simpler.
* Additional Goals:
  * some [DDD](https://en.wikipedia.org/wiki/Domain-driven_design) practice.
  * experiment with a little bit of translation and localization (Sigye means "clock" in Korean so I thought it would be neat to offer output in Korean).
  * ability to support multiple storage backends.

## Lessons Learned
* more use of AI in code generation (via CLINE and Claude artifacts and github copilot).
* [rich](https://rich.readthedocs.io/en/stable/introduction.html).
* po/mo files [internationalization](https://docs.python.org/3/library/gettext.html).
* [publishing packages using uv]({{< ref "publishing-via-uv-github-actions" >}}).
* deeper understanding of [click](https://click.palletsprojects.com/en/stable/).
* [peewee](https://github.com/coleifer/peewee) for SQLite access.
* [ryaml]() and [rtoml]() for faster json, yaml and toml writing and reading.

## User Experience

As something I leverage every workday, I wanted and have adapted the project to help me stay more personally accountable. And of course, if I ever get asked about where I'm spending my time or how many hours I've worked, I can get to that very easily.

And now having actually used the tool for several months, it's been *mostly* all I wanted it to be. Is there room for improvement? Absolutely! I still have some sharp edges to soften up a bit around editing and most specifically easier entry of past time records. But it's fulfilling its duties quite well as a whole.

```shell
❯ sigye --help
Usage: sigye [OPTIONS] COMMAND [ARGS]...

Options:
  --version                       Show the version and exit.
  -c, --config-file PATH          Path to config file  [default:
                                  /Users/steven/.sigye/config.yaml]
  -f, --filename PATH             Path to data file
  -o, --output_format [text|json|rich|yaml|markdown|csv|]
                                  Output format
  --help                          Show this message and exit.

Commands:
  delete (del,rm)  delete a time entry
  edit             edit a time entry using the system editor
  export           export time entries to a file
  list (ls)        display list of time entries for a time period
  start            start tracking work on a project
  status           displays currently tracked (if active)
  stop             stop tracking work on a project
```

## Other Observations

It's odd to me that Google search still doesn't think it's a python library or project even though it's on PyPI as [Sigye](https://pypi.org/project/sigye/) and has been for quite some time. *sigh*
