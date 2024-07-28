+++
title = 'None to version 0.1.0 as Fast as Possible'
date = 2024-07-28T15:34:48-05:00
draft = false
tags = ["python", "rye", "click", "pdm", "poetry"]
summary = "defining your version number in one place."
+++
# Stop Manually Defining Versions in Code

I recently was going through projects at work and updating all the places where `version` gets defined or set. Configuring [commitizen](https://commitizen-tools.github.io/commitizen/) is a whole different topic for a different day, but I (embarrassingly) only recently discovered a better technique for defining the version of a library or tool in code that is more DRY and just feels "more correct". So in the name of suggesting what feels like "more correct", I suggest you stop manually setting a variable in your code to declare the version.

So *old* normal would be something like this probably in a `__init__.py` file of a library:

```python
__version__ = "0.1.0"
```

It's simple. It's one line of code and it allows you then import your library and then fetch `__version__` to know the version. Or you might also declare `VERSION` this way or equal to `__version__`. All good, I'm not worried about that particular standards argument.

The issue you have is that `version` is also now typically declared in your `pyproject.toml` file. And depending on what tools you use, it might be declared multiple times in that file. Again, another topic for another post for that issue.

For the moment, I'm concerned about not re-declaring a version number in code when it's already being set and maintained (hopefully) in your `pyproject.toml` file (somewhere).

# Suggestion
## For Libraries

Assuming you still feel it's important to be able to return either `VERSION` or `__version__` from your library import (and that might could be up for debate), then I would suggest this slight modification to your library's `__init__.py` file:

```python
from importlib.metadata import version

__version__ = version(__name__)
VERSION = __version__  # if you also want to expose `VERSION` etc...
```

As a note about where this is coming from. Starting with Python 3.10, [importlib.metadata](https://docs.python.org/3/library/importlib.metadata.html) is no longer a provisional standard and is now an *official* way to fetch package metadata. The old mechanism, [pkg_resources](https://setuptools.pypa.io/en/latest/pkg_resources.html), is deprecated.

## For CLI Tools

So for a CLI tool, you can further shortcut this issue, depending on what toolkit you're using for providing the command-line parsing.

### click

If you're smart :wink:, you'll use [click](https://click.palletsprojects.com/) for your command line interface. If you then declare a `version_option` on your command, click will automatically do the right thing and fetch your package version for you without having to do anything.

{{< callout warning >}}
If you're not installing your package or project as part of the project, then this is *not* going to work. Most default setups with, `pdm`, `poetry`, and `rye` all seem biased towards importing the current project as an editable package. Also, the fact that you're exposing a `version` of some kind implies either you have an installable package/library or installable command-line script. If you just create a simple project that imports `click` for instance, without defining a package of any kind, then `click` will throw an error if you actually do a `--version` option on a `click.command()` because it can't determine the version. 
{{< /callout >}}

### argparse

If you're committed to the standard library and using `argparse` then you can define `version` like this:

```python
import argparse
from importlib.metadata import version

def main() -> int:
    parser = argparse.ArgumentParser(description="just an example")
    parser.add_argument('-V', '--version', action='version', version=version(__name__))
    args = parser.parse_args()
    print("Hello from apexample!")
    print(__name__)
    return 0
```

{{< callout note >}}
In this example, the script here is part of the `__init__.py` file. If the script name is something else, you could replace `__name__` with the name of your package.
{{< /callout >}}

So there ya go, `version` info where the source of truth is *only* in the `pyproject.toml` file (sort of).

## Other Topics

In some future post, I need to discuss [commitizen](https://commitizen-tools.github.io/commitizen/) as well as accomplishing a basic version of this using `rye`, `poetry` or `pdm`.
