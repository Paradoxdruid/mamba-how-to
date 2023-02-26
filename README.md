# Using Mamba-Forge for Python Environment Management

Management of version, dependencies, and virtual code environments in $\color{green}\textsf{Python}$ can be... [challenging](https://xkcd.com/1987/).

<a href="#"><img src="https://imgs.xkcd.com/comics/python_environment.png" width=40% height=40% alt="relevant xkcd"></a>

Typically and traditionally, Python installation relies on the `pip` tool, and a variety of virtual environment wrappers, such as `venv`, `virtualenv`, and `poetry`.

An alternative, that I espouse as a much better general solution for most folks, is ***completely abandoning*** that ecosystem and embracing the **Conda/Mamba** environment and package management ecosystem as an alternative.  This alternative works on OSX, Windows, and Linux, has an active community, and allows seamless management of non-Python dependencies as well.  It's a win!

($\color{blue}\textsf{\normalsize Note}$: **mamba** is a drop-in replacement for the default **conda** software command line interface/utility that has a much faster solver for dependency resolution.  Here, we will use `mamba` extensively--  but it is compatible with (and often simply a wrapper around) the `conda` CLI; resources you find for conda will work seamlessly and correctly in this setup as well-- and you can almost always replace `mamba` with `conda` in commands-- `mamba` will just solve installations much faster. The [conda cheatsheet](https://conda.io/projects/conda/en/latest/_downloads/843d9e0198f2a193a3484886fa28163c/conda-cheatsheet.pdf) is a quite useful one-pager reference!)

-----

## Table of Contents

1. [Environments](#environments)
2. [Installation](#installation)
3. [Getting Started](#getting-started)
4. [Installing Packages](#installing-packages)
5. [Managing Environments](#managing-environments)
6. [Publishing Packages](#publishing-packages)
7. [IDE Integration](#ide-integration)
8. [Further Contributions](#further-contributions)

## Environments

In the **conda/mamba** ecosystem, Python and other software is always organized into a virtual environment; even the base-level install is containerized and independent from the host system.  Each environment can have different python versions, with different dependencies, as well as different compiled helper modules and more (for instance, it can also install common CLI utilities such as `ncdu` or `htop`).

## Installation

My recommended way to get started is by installing the [**mamba-forge installer (click here)**](https://github.com/conda-forge/miniforge#mambaforge).  Importantly, this will set your system to download and install software packages from the community-driven [conda-forge](https://conda-forge.org/index.html) repository, which has a wide-selection of up-to-date packages.

This will install the core **mamba** CLI tool (and **conda** infrastructure), and create your first virtual environment, named `base`.

($\color{blue}\textsf{\normalsize Note}$: It's an **extremely** good habit to avoid the temptation of installing much else in this base environment, so don't go hog-wild installing dependencies just yet!)

After installation, **mamba** will automatically add to your shell configuration to activate the `base` environment on startup (modifying your `.bashrc`, `.zshrc`, or `profile.ps1` depending on your shell of choice; it's also compatible with Oh-My-Zsh, Spaceship, Starship, and many other prompt modifiers).  That means that after installation, if you restart your shell, you should see something like:

```bash
user@host\~ (base)$
```

Notice that the `base` conda environment is active.

## Getting started

Best practice is to avoid modifying the `base` environment, so let's create a typical working environment for day-to-day computing:

```bash
mamba create --name default python=3.11
```

This will create a new environment named `default` with Python version 3.11 (the latest and greatest stable version, as of this writing).

We can then modify our configuration file (`.bashrc`, `.zshrc`, etc.) by adding the following line at the very end:  `mamba activate default`.  That will cause it to be activated when we open a new shell, leaving the `base` environment untouched.

You can close your shell and reopen it, or type `mamba activate default`.  In either case, your prompt should now look like:

```bash
user@host\~ (default)$
```

## Installing packages

Now that we have a functional environment for day-to-day use, we can install commonly used Python packages.  As a scientist and data scientist, these include packages like `scipy`, `plotly`, `notebook`, and `dash` for me, along with code linting, testing, and formatting tools such as `black`, `mypy`, `pre-commit`, and `pytest`.

`mamba install scipy black mypy`

That command searches the `conda-forge` repository of packages (the default, and very up-to-date community maintained package repository), finds compatible packages, and installs them in the active environment.  It's very fast!

You can check your currently installed packages in your active environment with: `mamba list`

You can still use `pip` inside a mamba environment... but probably should use it as a last resort for packages unavailable on conda-forge.  Using pip introduces the possibility of [conflicts](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#using-pip-in-an-environment), but is generally safe.

## Managing environments

Your command prompt should always show your current active environment.  Behind the scenes, the `base` environment, which contains the `mamba` command itself is always lurking, so if you run:

`mamba deactivate`

You'll see your prompt change to contain `(base)`, not no environment at all.

You can list your current environments with:

`mamba env list`

and switch to a new environment with:

`mamba activate ENV_NAME`

Best practice is to create a new environment per project, so that dependencies can be pinned independently.  You can see more on environment management in the [conda docs](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#building-identical-conda-environments).

### Pinning dependencies and exporting dependency versions

Traditionally, python reported on a given project or environment's needed dependencies and versions with `requirements.txt` or `setup.py`.  Lately, these are being supplemented by the (much better) standards of `pyproject.toml`.  Mamba uses it's own standard, though it can be partially compatible with `pyproject.toml`.

**Mamba** specifies the packages of an environment with `environment.yml`.  It can be exported with various levels of "locks" to specify exactly the packages present or needed--  this can enable excellent cross-platform (i.e., OSX to Windows) reproducibility if used carefully.

To export a simple list of all packages and dependencies in your environment:

`mamba env export > environment.yml`

You can then recreate that environment (or share it with someone), making it on the new machine with:

`mamba env create -f environment.yml`

Note: this version of the `environment.yml` file may not work cross-platform, and still uses the dependency solver, so slightly different versions of dependencies might be installed.  To get around that, we have two paths:

#### Deterministic Reproducibility on the same Operating System

To ensure deterministic / exactly identical environments-- on the same operating system, we can use a spec file:

`mamba list --explicit > spec-file.txt`  and then to recreate it: `mamba create --name MY_ENV --file spec-file.txt`

#### Cross-platform environments

To ensure an `environment.yml` file works cross-platform, we can ensure we only include the instructions for the packages we purposefully installed, and let the dependency solver find the proper, system-appropriate dependencies:

`mamba env export --from-history > environment.yml`

### Environment Version History

One welcome feature of **conda** is that, like **git**, it stores version history for an environment, and it is possible to roll-back to prior versions.

`mamba list --revisions` and `mamba install --revision=REVNUM` can be used to restore prior states of the environment.

## Publishing Packages

If you create a Python project, odds are likely that someday you will want to share them.

The good news is that using **mamba** in no way stops you from using `pyproject.toml`, `wheel`, `build`, `twine`, `flutter`, or any other tools to create a wheel / distribution to upload to **PyPi**.  Go wild!

You also open up the option of creating **conda** packages, installable via `mamba install`.  The [official primer on conda package creation](https://docs.conda.io/projects/conda-build/en/latest/user-guide/tutorials/build-pkgs.html) is a good read, as is [The Joy of Packaging](https://python-packaging-tutorial.readthedocs.io/en/latest/conda.html).

Once you have a conda package, you can host it on your own channel / repository, or [submit it to conda-forge](https://conda-forge.org/docs/maintainer/adding_pkgs.html).

## IDE Integration

**Conda** virtual environments should be automatically detected and used by most major IDEs, such as [VS Code](https://code.visualstudio.com/docs/python/environments) and [PyCharm](https://www.jetbrains.com/help/pycharm/conda-support-creating-conda-virtual-environment.html).

## Further Contributions?

What's missing in this quick guide?  Add an [issue](https://github.com/Paradoxdruid/mamba-how-to/issues)!
