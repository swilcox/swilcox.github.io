+++
title = 'From rye to uv'
date = 2024-08-21T23:25:23-05:00
draft = false
summary = 'The now heavily updated uv utility has enough features to replace rye'
tags = ['python', 'rye', 'uv', 'poetry', 'ruff', 'pdm']
+++
# From `rye` to `uv`
## News

So, a couple of days ago, [Astral announced major changes and enhancements to uv](https://astral.sh/blog/uv-unified-python-packaging). This enhancement essentially eliminates the need or will soon for [rye](https://rye.astral.sh/) which Astral had also taken over maintenance/enhancement from [Armin Ronacher](https://lucumr.pocoo.org/2024/8/21/harvest-season/). I think Armin's quote is particularly relevant here:

> Domination is a goal because it means that most investment will go into one stack. I can only re-iterate my wish and desire that Rye (and with it a lot of other tools in the space) should cease to exist once the dominating tool has been established. For me uv is poised to be that tool. It's not quite there today yet for all cases, but it will be in no time, and now is the moment to step up as a community and start to start to rally around it.

## Significant Features

`uv` now supports:
* package management via `add` / `remove` etc... like `rye` and other tools.
* tool management (similar to pipx and rye) except that you continue to access via `uvx` or `uv tool`.
* python installations via `uv python`.
* support for inline requirements in scripts [PEP-723](https://peps.python.org/pep-0723/).
* workspaces for essentially sub-libraries (I'm anxious to look into this).

## Conversion of a Project

I swapped my [wordmind](https://github.com/swilcox/wordmind) project over from rye to uv. In the process, I also decided to switch to a more standard package-based setup so that I could use the `pyproject.toml` for [version number management](https://swilcox.github.io/post/none_to_0_dot_1/) and consistency among other things. The process was pretty painless since it's a small project. The funny part of this project is that I believe I started on [pdm](https://pdm-project.org/en/latest/) and then to rye and now uv. The other thing I decided to do as part of this switch was to just start over with a fresh `pyproject.toml` but really, it looks like it probably could have been converted pretty easily with a few tweaks.

The nice part is pdm, rye and uv all use the standard dependencies. They also support `project.scripts` for projects that are installed packages. Dev dependencies are handled similarly but broken out by tool so for uv that's `tool.uv.dev-dependencies`. They all support lock file(s). A part that I like for uv is the guarantee of cross-platform handling in the lock file and the fact that there's only 1 lock file as opposed to multiple.

### Github Actions

Following [instructions available on Astral's site](https://docs.astral.sh/uv/guides/integration/github/). It was very easy to tweak my github-actions to use uv:

```yaml
name: ci_test
on: [pull_request]
jobs:
  pytest:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up uv
        # Install latest uv version using the installer
        run: curl -LsSf https://astral.sh/uv/install.sh | sh 
      - name: install python version
        run: |
          uv python install
      - name: install deps
        run: |
          uv sync
      - name: Execute Tests
        run: |
          uv run pytest
```

{{< callout note >}}
I am not pinning to a particular version of `uv` here. That's likely not a good idea, but for this project, it's low risk and allows me to stay up-to-date with the latest without code changes. For something super critical, it seems a little more likely you'd want to pin uv in your pipeline. But then again, you're probably not ready to switch to `uv` for something so critical.
{{< /callout >}}

## Final Thoughts

As [Simon Willison](https://simonwillison.net/2024/Aug/21/armin-ronacher/) and others have, I will also highlight this statement from Armin regarding fears over the tool(s) from Astral:

> there is an elephant in the room which is that Astral is a VC funded company. What does that mean for the future of these tools? Here is my take on this: for the community having someone pour money into it can create some challenges. For the PSF and the core Python project this is something that should be considered. However having seen the code and what uv is doing, even in the worst possible future this is a very forkable and maintainable thing. I believe that even in case Astral shuts down or were to do something incredibly dodgy licensing wise, the community would be better off than before uv existed. -- <i>Armin Ronacher</i>

There has been some resistance in the Python community to Astral's contributions partly out of fear and partly I think due to pride and other factors. Armin's attitude of *wanting* his tool (rye) to be replaced with something better (a goal he's mentioned from the beginning) is admirable. Not all tool authors have had the same attitude. I don't have time to survey the authors of all projects but I'm sure there's hurt feelings by some when projects they've spent years developing are suddenly overshadowed by alternative or re-implementations of said tools. By using a combination of `uv` and `ruff` there's really going to be no need to use: Poetry, PDM, rye, pip, pip-tools, pipenv, pyenv, pipx, flake8, pylint, black, yapf, isort and many others. And if there's something different/special about one of those just listed, whatever special feature is probably going to be implemented or made irrelevant in time. Again, these are all tools that have been *very significant* in the Python community. Thousands of hours of work were invested in these, so I get it. But the bigger picture is that moving closer to Rust's `cargo` utility in terms of simplicity (and speed) is a better way. It makes it easier for new developers and also old developers like me. After all, "simple is better than complex".
