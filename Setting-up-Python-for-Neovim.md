# Setting up Python for Neovim

Neovim requires a package to be installed for Python plugins to work.  You
*really* should read `:help provider-python` to supplement the info on this page.

## Table of Contents

* [A brief overview of Neovim + Python](#a-brief-overview-of-neovim--python)
* [Requirements](#requirements)
* [Simple Setup](#simple-setup)
* [Using Virtual Environments](#using-virtual-environments)
* [Tips for using pyenv](#tips-for-using-pyenv)
* [Why you shouldn't use sudo](#why-you-shouldnt-use-sudo)

## A brief overview of Neovim + Python

The main advantages of using Python in plugins is that it enables plugins to
have access to network sockets, and perform long-running or expensive
operations in the background without freezing the Vim UI.  This is the reason
it is commonly used with completion plugins.

Vim plugins are able to execute Python code when Vim is compiled with Python
support.  However, default system installations of Vim that are compiled with
Python versions that are considered to be the stable default for the OS, which
usually meant Python 2.6 or 2.7 until recently.  Additionally, Vim could only
be compiled with Python 2 or Python 3, not both.  Re-compiling Vim to include
newer versions is a non-trivial task and it would put an undue burden on users
if plugins required Python 3.

Neovim resolves these limitations by providing a bridge to your system's Python
installations.  It requires a [Python package][python-client] to be installed,
which still puts a burdern on the user, but it is arguably easier than
re-compiling Vim.


## Requirements

Each Python interpreter that is used with Neovim will require the
[neovim][python-client] package.

`deoplete.nvim` requires Python 3 to be installed.  By extension,
`deoplete-jedi` does, too.  It is recommended that Python 3.3+ is used, but any
Python version greater than 3.1 should work.


## Simple Setup

If you do light Python development, or the extent of your Python use is limited
to Neovim, the basic installation instructions will be sufficient.

`:h nvim-python` suggests the use of `sudo pip install neovim` for
system-wide availability.  It's an acceptable solution if you are certain that
your Python environment will be unaffected.  Read the following section before
you consider using `sudo`: [Why you shouldn't use sudo](#why-you-shouldnt-use-sudo)

It is recommended to use the `--user` flag when installing the `neovim`
package:

```shell
pip2 install --user neovim
pip3 install --user neovim
```

The `--user` flag installs `neovim` in a directory within your home directory.
The limitation with this is that every user account on the system will need to
perform this step.  You can confirm the location and that your environment
permits user packages by running: `python -m site`


## Using Virtual Environments

If you do heavy Python development, you will most likely prefer using a virtual
environment.  `deoplete-jedi` will display completions for your current shell's
Python interpreter (run: `which python` to determine this).  This includes the
Python interpreter that are made active using a virtualenv.

If you are already using virtualenv for all of your work, it is recommended
that you use separate virtual environments for Neovim, and only Neovim.  This
will remove the need to install the `neovim` package in each virtual
environment.  The following examples uses [pyenv][].  There are
[tips](#tips-for-using-pyenv) below for installing and using `pyenv` without
much effort.  `TODO: add instructions for pyvenv and virtualenv`

```shell
pyenv install 2.7.11
pyenv install 3.4.4

pyenv virtualenv 2.7.11 neovim2
pyenv virtualenv 3.4.4 neovim3

pyenv activate neovim2
pip install neovim
pyenv which python  # Note the path

pyenv activate neovim3
pip install neovim
pyenv which python  # Note the path

# The following is optional, and the neovim3 env is still active
# This allows flake8 to be available to linter plugins regardless
# of what env is currently active.  Repeat this pattern for other
# packages that provide cli programs that are used in Neovim.
pip install flake8
ln -s `pyenv which flake8` ~/bin/flake8  # Assumes that $HOME/bin is in $PATH
```

Now that you've noted the interpreter paths, add the following to your
`init.vim` file:

```vim
let g:python_host_prog = '/full/path/to/neovim2/bin/python'
let g:python3_host_prog = '/full/path/to/neovim3/bin/python'
```

## Tips for using pyenv

- [Ensure you have the prerequisites installed][pyenv-prereq].
- Installing pyenv with homebrew is unreliable.  Use [pyenv-installer][]
  instead.
- There is a final step that's printed to the terminal after installing `pyenv`
  and it's important!
- To confirm you have `pyenv` correctly installed, run `which pyenv`.  It
  should print a shell function, not a file path.
- Run `pyenv doctor` to avoid surprises.
- `pyenv global` can be thought of as altering the `$PATH` to include the
  specified versions' `bin` directory.  This only works while `pyenv` is
  active.
- `pyenv shell` is the same as above, but for the current session.
- `pyenv local` is the same as above, but it writes a `.python-version` file in
  the current directory.  It allows the specified versions to be automatically
  set when you enter the directory, and unset when you leave it.  Very
  convenient for projects.
- You will want to add `.python-version` to your global `.gitignore` file.
- `pyenv shell --unset` will reset the session's Python versions.
- `pyenv activate venvname` differs from `pyenv shell venvname` in that only
  one `$VIRTUAL_ENV` can be active at a time.
- `pyenv deactivate` deactivates the virtual environment.
- `pyenv versions` lists the versions you have installed.  `system` is a
  special case pointing to Python versions that were originally found in
  `$PATH`.  Virtual environment names are listed as versions.


## Why you shouldn't use sudo

Unless you know what you're doing, the system installed Python interpreters
should remain pristine and only contain packages that were installed via the
OS's package manager (read: *not pip*).  `sudo` in front of `pip install`
should make you cringe any time it's recommended.

Some distros use separate locations for system packages and user packages, but
the potential to mess things up is increased when you use `sudo`.  On distros
that don't make a distinction between system and user packages, it's possible
to upgrade system packages with `sudo pip install -U`.  Or the reverse could
happen and you'll downgrade packages installed with `pip`.  There is also the
issue of non-Python system packages that may require the specific versions of
Python packages that were built for the distro.  The point is, you will have
two package managers competing to decide what's new or old.

A very common example of what can go wrong: `sudo pip install --upgrade pip`.
[It is a problem many Python developers encounter][broken-pip].  This problem
occurs more than it should and renders all Python programs installed in the
system bin directories to be broken.

If you can't honestly say that you know how Python packages are laid out or
what your system's Python dependencies are, and enjoy having a stable,
error-free Python installation, leave the system Python installations alone.

[python-client]: https://pypi.python.org/pypi/neovim
[pyenv]: https://github.com/yyuu/pyenv
[pyenv-prereq]: https://github.com/yyuu/pyenv/wiki/Common-build-problems
[pyenv-installer]: https://github.com/yyuu/pyenv-installer
[broken-pip]: https://www.google.com/search?q=sudo+pip+broken+after+upgrade
