# Django (+DRF) Boilerplate
## Requirements
1. clone project. 
2. create an `env` environment and activate it.
3. install requirements
```bash
pip install -r requirements.txt
```

Code Quality Part
======
Here are my personal perference.
### 1. Black
With black you can format Python code, and my preference is using [PEP 8](https://www.python.org/dev/peps/pep-0008/) as my style guide, and so, 79-characters per line of code is what I use.

In case of `black` configuration we can setup in `pyproject.toml` file. Mine look like this:
```yaml
[tool.black]
line-length = 79
multi_line_output = 3
exclude = '''
/( <br>
    | env
)/
'''
```

Let’s explain each option.

* `-l` or `--line-length`: How many characters per line to allow. [default: 88]
* `-t` or `--target-version`: Python versions that should be supported by Black’s output. [default: per-file auto-detection]

### 2. isort
> A Python utility / library to sort imports.
And just as their slogan states: “isort your imports, so you don’t have to.”

And the same for `isort` inside of `setup.cfg`.

```yaml
[isort]
line_length = 88
skip = env/
multi_line_output = 3
skip_glob = **/migrations/*.py
include_trailing_comma = true
force_grid_wrap = 0
use_parentheses = true
```

The options used are mainly to be compatible with black [(see here)](https://pycqa.github.io/isort/docs/configuration/black_compatibility/):

* `--multi-line`: Multi line output (0-grid, 1-vertical, 2-hanging, 3-vert-hanging, 4-vert-grid, 5-vert-grid-grouped, 6-vert-grid-grouped-no-comma, 7-noqa, 8-vertical-hanging-indent-bracket, 9-vertical-prefix-from-module-import, 10-hanging-indent-with-parentheses).
    * 3-vert-hanging
* `--profile`: Base profile type to use for configuration. Profiles include: black, django, pycharm, google, open_stack, plone, attrs, hug. As well as any shared profiles.
    * black

But there’s still something missing. black does not care about comments or [docstrings](https://www.python.org/dev/peps/pep-0257/), and isort cares even less, for obvious reasons; enter flake8.

### 3. flake8
> flake8 is a python tool that glues together pep8, pyflakes, mccabe, and third-party plugins to check the style and quality of some python code.

[PEP 8](https://www.python.org/dev/peps/pep-0008/) recommends limiting docstrings or comments to 72 characters, which is exactly what I’m using for flake8.

So in order to use `flake8` you’ll have to create a `.flake8` file or `setup.cfg` file, and i prefer `setup.cfg`. Mine looks like this:

```yaml
[flake8]
exclude = .git,*/migrations/*,*env*,*venv*,__pycache__,*/staticfiles/*,*/mediafiles/*
max-line-length = 120
max-doc-length = 72
max-complexity = 10
count = true
statistics = true
```

So let’s explain each option used.

* `--max-doc-length`: Maximum allowed doc line length for the entirety of this run. (Default: None)
* `--ignore`: Comma-separated list of errors and warnings to ignore (or skip). For example, --ignore=E4,E51,W234. (Default: [‘E226’, ‘E123’, ‘W504’, ‘E121’, ‘W503’, ‘E126’, ‘E704’, ‘E24’])

In my case I am using `72` as the maximum allowed characters for my docstrings, in accordance to PEP 8, and ignoring the following errors:

* `E211`: whitespace before ‘(‘
* `E999`: SyntaxError: invalid syntax
    * In my case this occurs again with the `print` statement where I am printing just one arguments like this `print arg`
* `F401`: module imported but unused
    * I do import some modules in order to get “Intellisense” when I peek into the details in PyCharm
* `F821`: undefined name `name`
    * In one of my libraries I am checking if an argument is a string, and in order to cover my bases with plain strings (`str`) and unicode, I found that using `basestring` would work for all characters, including non-Latin characters
* `W503`: line break before binary operator
    * It doesn’t like when binary operators are broken into multi-line statements

### 4. pre-commit
> A framework for managing and maintaining multi-language pre-commit hooks.
Finally, let’s put it all together with [pre-commit](https://pre-commit.com/).

Here is my final pre-commit file:
```yaml
repos:
  - repo: https://github.com/psf/black
    rev: 22.6.0
    hooks:
      - id: black
        exclude: ^.*\b(migrations)\b.*$
  - repo: https://github.com/PyCQA/isort
    rev: 5.10.1
    hooks:
      - id: isort
  - repo: https://github.com/PyCQA/flake8
    rev: 4.0.1
    hooks:
      - id: flake8
```


### 5. .vscode/settings.json
To work smoothly during development let's add `black`, and `isort`:

```json
{
    "editor.formatOnSave": true,
    "python.formatting.provider": "black",
    "python.formatting.blackArgs": [
        "--line-length=119",
    ],
    "python.sortImports.args": [
        "--profile=black",
    ],
    "[python]": {
        "editor.codeActionsOnSave": {
            "source.organizeImports": true
        }
    }
}
```
