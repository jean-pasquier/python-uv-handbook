# Test uv


## Prerequisites

```shell
brew install uv
```

## How to

```shell
uv python install 3.13
uv venv -p 3.13
uv sync
uv run hello
```

## Tools


### Option 1: tool as app dependency

```shell
uv add ruff --dev

```

ruff is installed in application venv


### Option 2: tool as uv tool

The uv tools are installed in a separate venv from our application. This works because ruff does not need application installed (only source code).

```shell
# Equivalent
uvx ruff check src/
uv tool run ruff check src/

# Optionally install the tool
uv tool install ruff  # installed in $HOME/.local/...
ruff check src/  # because ruff available in PATH var 
```

But this fails because pytest requires to import our application modules:


```shell
uvx pytest tests/

# pytest must be installed as app dependency
uv add pytest --dev
uv run pytest tests/  # runs in our application venv
```


## pip

`uv pip` is a drop-in replacement of standard `pip` command. 

```shell
uv pip install pydantic  # installs dep in our app venv

uv pip list  # should print all "uv installed" dep and pydantic (+ its deps) 
```

pip installed libraries are not managed by uv. Running a uv sync would uninstall the pip installed libs.

```shell
# Considering a blank app (no dependency)
uv add ruff  # first dep
uv pip list
# Package Version 
# ------- -------
# ruff    0.14.0

uv pip install pydantic
uv pip list
# Package       Version 
# ------------- -------
# ruff          0.14.0
# pydantic      2.12.0
# pydantic-core 2.41.4
# ...

uv sync  # uninstall "pip installed" pydantic, pydantic-core
uv pip list
# Package Version 
# ------- -------
# ruff    0.14.0
```



## Clean cache

Some libraries are heavy

```shell
uv cache clean  # all lib
uv cache clean ruff torch  # specific libs
```
