# Test uv


## Prerequisites

Install uv

```shell
curl -LsSf https://astral.sh/uv/install.sh | sh
# or
brew install uv
```

See [uv installation guide](https://docs.astral.sh/uv/getting-started/installation/)


## Comparison within Python ecosystem

| Feature                 | pyenv | pip | pipx | twine | poetry | uv |
|-------------------------|-------|-----|------|-------|--------|----|
| Managed python versions | √     |     |      |       |        | √  |
| Manage app dependencies |       | √   |      |       | √      | √  |
| Run tools               |       |     | √    |       |        | √  |
| Build app               |       |     |      |       | √      | √  |
| Publish app             |       |     |      | √     | √      | √  |


### Compare with Poetry

| Concept              | poetry                  | uv                  |
|----------------------|-------------------------|---------------------|
| config file name     | pyproject.toml          | pyproject.toml      |
| lock file name       | poetry.lock             | uv.lock             |
| manage dependencies  | poetry add/remove <dep> | uv add/remove <dep> |
| install dependencies | poetry install          | uv sync             |
| run app/script       | poetry run <script>     | uv run <script>     |

uv is quite similar to poetry commands


## How to

See all features on [uv documentation](https://docs.astral.sh/uv/getting-started/features/)

### Manage python (same as pyenv)

```shell
uv python install 3.13
```

### Install, build and run application 

```shell
uv venv -p 3.13
uv sync
uv run hello  # hello is defined in pyproject.toml as `project.scripts` to run test_uv/app.py:main function
# almost equivalent of (because main() is called in app.py's __main__ section)
uv run src/test_uv/app.py
```

## Use tools


### Option 1: tool as app dependency

```shell
uv add ruff --dev
```

ruff is installed in application venv


### Option 2: tool as uv tool

The uv tools are installed in a separate venv from our application. This works because ruff does not need application installed (only source code):

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
# Throws the error: No module named 'test_uv'

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


## Docker

uv provides several ways to use uv in docker:
1. Distroless Docker images (do not contain anything but the uv binaries)
2. Derived images including an operating system with uv pre-installed


### Using distroless

```dockerfile
FROM python:3.12-slim-trixie 
# or any safely built python image

COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/
```


### Using derived image

```shell
docker run --rm -it ghcr.io/astral-sh/uv:debian uv --help
```

### Best practices

* Compiling bytecode: `UV_COMPILE_BYTECODE` 
* Caching: `UV_CACHE_DIR`
* Link mode to "copy": `UV_LINK_MODE`
* Example of [multistage build](https://github.com/astral-sh/uv-docker-example/blob/main/multistage.Dockerfile)
