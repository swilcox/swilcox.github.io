+++
title = 'Python_army_days'
date = 2024-09-16T09:26:04-05:00
draft = false
tags = ['python', 'army-days', 'uv', 'tdd', 'claude']
+++
# A Python version of `army-days`

It only seemed right that I would actually create a [python version of my army-days](https://github.com/swilcox/py-army-days) program.

## Goals

* Feature parity with other versions.
* Publish as a pypi package so folks can easily install it.
* Analyze how things are easier/harder in python versus other languages.
* Reasonable error handling and schema validation.
* Add on additional functionality as appropriate, specifically:
  * Sample configuration file generation.

{{< callout note >}}
While I'm trying to keep things the same across different languages, in terms of functionality, I'm not 100% committed to compatibility. Still, I do check running the different implementations against the same configuration file(s).
{{< /callout >}}

## Learnings

* This was a first time deploying to [PyPI](https://pypi.org/) from a [uv](https://docs.astral.sh/uv/guides/publish/#building-your-package) project. But it really wasn't bad at all.
  * I did need to do a quick rename of project, because I'd initially called the actual package name `py-army-days` which is a bit of a pain when installing and running compared to the much more suitable `army-days`.
  * I also ran into [this issue](https://docs.astral.sh/uv/guides/publish/#building-your-package) with the uv cache. I had to do the `--refresh` to get my new package.
* Not having to do classic dependency injection for certain things in python sure is nice for tests. So it was nice to use [freezegun](https://github.com/spulec/freezegun) to set a fixed `datetime` with a decorator. And also a fairly simple task to capture `stdout` as part of a test:

```python
    with io.StringIO() as buf:
      with contextlib.redirect_stdout(buf):
        _output_color_events(results)
      output_data = buf.getvalue()
```
* Speaking of output, I actually find the colorized ANSI output libraries for python (like colorama and a few others) to be frustrating. Colorama doesn't support underline (because of Windows compatibility issues) and all the other libraries seemed to avoid easily adding TrueColor support. My goal is subtle background shading on alternating lines and it was frustrating to not be able to easily find that. Turned out to be easier to ask [Claude](https://claude.ai) to generate python functions to return rgb background and foreground ANSI codes. So in essence, I rolled my own basic ANSI colors library.

* Obviously, could have a basic working program much much shorter than this. The entire program could be done without a single additional library and in a single script. But I was also aiming for decent error checking and organization and tests.


### TLDR;

Python is still the easiest and fastest way for me to code and also have tests.

### Future project

My own simple ANSI color text library.
