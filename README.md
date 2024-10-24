# Oceandata Notebooks

The code repository for the collection of notebooks distributed as
[oceandata tutorials and data recipes][tutorials].

## For Contributors

> [!IMPORTANT]
> - Edit notebooks in JupyterLab so Jupytext can do its magic.
> - Create new notebooks under the `notebooks` folder, starting from a copy of `COPYME.ipynb`.

Keeping notebooks in a code repository is tough for collaboration and curation
because notebooks contain large, binary outputs and metadata that messes up `git merge`.
This repository uses [Jupytext][jupytext] to keep notebooks (the ".ipynb" files) paired
with plain text files (the ".py" files). Contributors should work on the ".ipynb" files
in the `notebooks` folder. These files are ignored by git, but the paired ".py" files are
not. So, save your changes in a notebook, commit the paired ".py" file, and push.

> [!Note]
> When you first clone this repository, the `notebooks` folder will be empty. Open a
> terminal at the project root and sync from the `src` using `jupytext`.

```shell
$ shopt -s globstar  # enables `**` in Bash (enabled by default in Zsh)
$ jupytext --sync src/**/*.py
```
You can run this any time to ensure there is an ".ipynb" file in your `notebooks` folder
for every ".py" file in the `src` folder.

## For Maintainers

The subsections below provides information that repository maintainers will want to know.
If you merge a pull request to `main`, consider yourself a maintainer.

<!-- TODO: pre-commit shows warning when run in virtual env -->

### Development Environment: the `uv.lock` file

We use the `uv` command line tool to maintain a virtual environment for maintainer activities,
which you should install separately (a `pip install --user` or `pipx install` both work).
The `uv sync` command creates the development environment from the `uv.lock` file.
```shell
$ uv sync --extra dev
$ source .venv/bin/activate
(oceandata-notebooks) $
```

### Automation and Checks: the `.pre-commit-config.yaml` file

We use several automations to get standard code formatting, run lint checks, and ensure
consistency between ".py" and ".ipynb" files. These are implemented using the [pre-commit]
tool from the development environment. You can setup git hooks to run these automations,
as needed, at every commit.
```shell
(oceandata-notebooks) $ pre-commit install
```
You can also run checks over all files chaged on a feature branch or the currently
checked out git ref. For the latter:
```shell
(oceandata-notebooks) $ pre-commit run --from-ref main --to-ref HEAD
```

### Dependencies: the `pyproject.toml` file

For every `import` statement, or if a notebook requires a package to be installed
for a backend (e.g. h5netcdf), make sure the package is listed in the `project.dependencies`
array in `pyproject.toml`. You can add entries manually or using `uv`, as in:
```shell
$ uv add scipy
```
The `project.optional-dependencies` tables list additional dependencies that are needed
either for a Jupyter kernel, for a Docker image with JupyterLab, or by maintainers.

### Container Image: the `docker` folder

The `docker` folder has configuration files that `repo2docker` uses to build an image suitable
for use with a JupyterHub platform. The following command builds and runs the image locally,
while the `jupyterhub/repo2docker-action` pushes built images to GitHub packages. You
must have `docker` available to use `repo2docker`.

```shell
(oceandata-notebooks) $ repo2docker --appendix="$(< docker/appendix)" -p 8889:8888 docker jupyter lab --no-browser --ip 0.0.0.0
```

The configuration files are a bit complicated, but updated automatically by `pre-commit`
hooks following changes to `pyproject.toml` and `docker/environment.yml`. No `requirements`
file in this repository should be manually edited. The `docker/environment.yml` file is there
for non-Python packages available on conda-forge; we use PyPI for Python packages.
1. `requirements.txt` are the (locked) dependencies needed in `book/setup.py`
1. `docker/requirements.in` are the (locked) packages from repo2docker and `docker/environment.yml`
1. `docker/requirements.txt` are a merge of our (locked) dependencies with `docker/requirements.in`

> [!Note]
> The top-level `requirements.txt` file is located there to also provide dependencies for
> [Binder](https://mybinder.org/).

### Rendering to HTML: the `book` folder

In addition to the ".py" files paired to notebooks, the `book` folder contains configuration
for a [Jupyter Book][jb]. Only notebooks listed in `book/_toc.yml` are included. Building
the notebooks as one book provides smaller files for tutorial content, a single source of
JavaScript and CSS, and a test that all notebook run without errors.

To build the book:
```shell
(oceandata-notebooks) $ jb build book
```
That populates the `book/_build` folder. The folder is ignored by git, but its contents
can be provided to the web team. The `_templates` make the website look very plain on
purpose, so that only the notebook content is included in the HTML files.

When you create a brand new notebook under the `notebooks` folder, it won't be rendered
to HTML locally or visible as a notebook on GitHub until you create, commit, and push the
copy under `book`. To create the `book` version:
```shell
$ jupytext --sync book/src/path/to/new/tutorial.py
$ git add book/notebooks/path/to/new/tutorial.ipynb
```
Opening the notebook this creates under `book/notebooks` in JupyterLab will add unwanted
metadata. Do not commit any changes introduced by opening the new notebook.

## How to Cite

## Acknowledgements
This repository has greatly benefited from works of multiple open-science projects,
notably [Learn OLCI][learn-olci] and the [NASA EarthData Cloud Cookbook][cookbook].

[tutorials]: https://oceancolor.gsfc.nasa.gov/resources/docs/tutorials
[jupytext]: https://jupytext.readthedocs.io/
[jupyterlab]: https://jupyter.org
[jb]: https://jupyterbook.org
[learn-olci]: https://github.com/wekeo/learn-olci/blob/main/README.md
[cookbook]: https://nasa-openscapes.github.io/earthdata-cloud-cookbook
[pre-commit]: https://pre-commit.com/
