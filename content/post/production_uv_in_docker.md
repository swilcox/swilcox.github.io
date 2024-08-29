+++
title = 'Production uv in docker'
date = 2024-08-28T19:45:39-05:00
draft = false
tags = ['uv', 'python', 'docker']
summary = 'A link to an interesting article about using uv in a Docker container deployment.'
+++
# uv use in docker containers

I found this [great article by Hynek Schlawack](https://hynek.me/articles/docker-uv/) about what a possible docker container setup would be like for a python program using uv.

Quoting here the side-benefit of the article which applies beyond uv and extends to any dependency management scheme with python when deploying via containers:

{{< callout info "*Hynek Schlawack*" >}}
As a reminder, in production, you want the following properties from your Docker workflow:

    1. Multi-stage builds, so you don’t ship your build tools.

    2. Judicious layering, for fast builds. Layers should be added in the inverse order they are likely to change so they can be cached for as long as possible.

    This also means that dependency installations (what’s in uv.lock) and application installations (what you wrote) should be strictly separate. While developing, your code is more likely to change than your dependencies.

    3. Bonus: build-cache mounts, so, for example, wheels don’t have to be rebuilt if your dependency layer needs to be recreated.

    4. Bonus: byte-compile your Python files for faster container startup times.
{{< /callout >}}

So the other tidbit is to watch this issue in uv (as it would make things simpler):

* [Support custom venv locations for uv sync #5229](https://github.com/astral-sh/uv/issues/5229)
