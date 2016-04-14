# Setting up Python for Neovim

Neovim requires a package to be installed for Python plugins to work.  You *really* should read `:h nvim-python` to supplement the info on this page.

## Requirements

`deoplete.nvim` requires Python 3 to be installed.  By extension, `deoplete-jedi` does, too.  It is recommended that Python 3.3+ is used, but any Python version past 3.1 should work.

## Using Virtual Environments

If you do heavy Python development, you will most likely prefer using a virtual environment.  `deoplete-jedi` will display completions for your current shell's Python interpreter (run: `which python` to determine this).  This includes the Python interpreter that's made active using a virtualenv.

If you feel this is too much work for your needs, read `:h nvim-python` for the basic install instructions.  Add the symlink `~/bin/python` pointing to the `python3` interpreter if your prefer to use Python 3, but your system's `python` binary links to Python 2.

If you are already using virtualenv for all of your work, it is recommended that you use separate virtual environments for Neovim, and only Neovim.  This will remove the need to install the `neovim` package in each virtual environment.  The following examples uses [pyenv][].  There are [tips](#tips-for-using-pyenv) below for installing and using `pyenv` without much effort.  `TODO: add instructions for pyvenv and virtualenv`

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

Now that you've noted the interpreter paths, add the following to your `init.vim` file:

```vim
let g:python_host_prog = '/full/path/to/neovim2/bin/python'
let g:python3_host_prog = '/full/path/to/neovim3/bin/python'
```

## Tips for using pyenv

- [Ensure you have the prerequisites installed](https://github.com/yyuu/pyenv/wiki/Common-build-problems).
- Installing pyenv with homebrew is unreliable.  Use [pyenv-installer][] instead.
- There is a final step that's printed to the terminal after installing `pyenv` and it's important!
- To confirm you have `pyenv` correctly installed, run `which pyenv`.  It should print a shell function, not a file path.
- Run `pyenv doctor` to avoid surprises.
- `pyenv global` can be thought of as altering the `$PATH` to include the specified versions' `bin` directory.  This only works while `pyenv` is active.
- `pyenv shell` is the same as above, but for the current session.
- `pyenv local` is the same as above, but it writes a `.python-version` file in the current directory.  It allows the specified versions to be automatically set when you enter the directory, and unset when you leave it.  Very convenient for projects.
- You will want to add `.python-version` to your global `.gitignore` file.
- `pyenv shell --unset` will reset the session's Python versions.
- `pyenv activate venvname` differs from `pyenv shell venvname` in that only one `$VIRTUAL_ENV` can be active at a time.
- `pyenv deactivate` deactivates the virtual environment.
- `pyenv versions` lists the versions you have installed.  `system` is a special case pointing to Python versions that were originally found in `$PATH`.  Virtual environment names are listed as versions.


## Why you shouldn't use sudo

Unless you know what you're doing, the system installed Python interpreters should remain pristine and only contain packages that were installed via the OS's package manager (read: *not pip*).  `sudo` in front of `pip install` should make you cringe any time it's recommended.

Some distros use separate locations for package manager packages and user packages, but the potential to mess things up is increased when you use `sudo`.  On distros that don't make a distinction between system and user packages, it's possible to upgrade system packages with `sudo pip install -U`.  Or the reverse could happen and you'll downgrade packages installed with `pip`.  There is also the issue of non-Python system packages that may require the specific versions of Python packages that were built for the distro.  The point is, you will have two package managers competing to decide what's new or old.

If you can't honestly say that you know how Python packages are laid out, and enjoy having a stable, error-free Python installation, leave the system Python installations alone.

[pyenv]: https://github.com/yyuu/pyenv
[pyenv-installer]: https://github.com/yyuu/pyenv-installer