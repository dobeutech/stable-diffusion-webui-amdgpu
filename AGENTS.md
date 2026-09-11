# AGENTS.md

## Repository scope

- Single Python/Gradio application; this is not a monorepo.
- Supported launcher Python is 3.10 by default (`webui.sh` falls back to `python3`); CI tests use Python 3.10.6 and lint uses Python 3.11.
- Run commands from the repository root.

## Setup and run

### Linux

```bash
# Debian/Ubuntu prerequisites (other distributions are listed in README.md)
sudo apt install wget git python3 python3-venv libgl1 libglib2.0-0

# Create the venv, install application dependencies, and start the dev server
./webui.sh
```

- Put local launcher overrides such as `python_cmd`, `venv_dir`, and `COMMANDLINE_ARGS` in `webui-user.sh`; do not edit `webui.sh` for local configuration.
- AMD backend examples: `COMMANDLINE_ARGS="--use-directml" ./webui.sh` or `COMMANDLINE_ARGS="--use-zluda" ./webui.sh`.
- Windows entry point: `webui-user.bat` (delegates to `webui.bat`).
- There is no separate build command; the launcher prepares the Python environment before starting the app.

### CI-style dependency setup

```bash
# Test tooling
pip install wait-for-it -r requirements-test.txt

# Resolve/install app dependencies without starting the server
TORCH_INDEX_URL=https://download.pytorch.org/whl/cpu \
  python launch.py --skip-torch-cuda-test --exit

# JavaScript lint tooling (command used by CI)
npm i --ci
```

## Tests

### Fast or targeted tests

```bash
# Standalone unit-test file (does not require the API server)
python -m pytest -vv test/test_torch_utils.py

# One test function
python -m pytest -vv test/test_torch_utils.py::test_get_param

# Collect tests without running them
python -m pytest --collect-only -q test
```

Most tests under `test/` call the live API at `http://127.0.0.1:7860` (configured in `pyproject.toml`). Start the CI test server before running the full suite:

```bash
python -m coverage run --data-file=.coverage.server launch.py \
  --skip-prepare-environment \
  --skip-torch-cuda-test \
  --test-server \
  --do-not-download-clip \
  --no-half \
  --disable-opt-split-attention \
  --use-cpu all \
  --api-server-stop
```

In another shell:

```bash
# Full CI test command
python -m pytest -vv --junitxml=test/results.xml \
  --cov . --cov-report=xml --verify-base-url test

# One API test while the server is running
python -m pytest -vv --verify-base-url \
  test/test_txt2img.py::test_txt2img_simple_performed
```

Stop the test server when finished:

```bash
curl -XPOST http://127.0.0.1:7860/sdapi/v1/server-stop
```

## Lint and format

Run both CI linters before committing:

```bash
ruff check .
npm run lint
```

Available automatic JavaScript fix command:

```bash
npm run fix
```

- Ruff configuration lives in `pyproject.toml`; CI pins `ruff==0.3.3`.
- ESLint configuration lives in `.eslintrc.js`; `npm run fix` is the only configured formatter/fixer.

## Pull requests

- Target `dev`. The `Pull requests can't target master branch` workflow deliberately fails PRs whose base is `master`.
- No branch-name pattern is configured in the repository; use a short descriptive branch name.
- No commit-message format is configured in the repository; keep commit subjects concise and descriptive.
- Complete `.github/pull_request_template.md`: description, change summary, linked issues, screenshots/video when applicable, self-review, style, and tests.
- Repository-defined PR CI checks (branch-protection requirements are not stored in this repository):
  - `Linter / ruff`: `ruff check .`
  - `Linter / eslint`: `npm run lint`
  - `Tests / tests on CPU with empty model`: live-server pytest suite
  - `Pull requests can't target master branch / check`: base-branch guard

## Key paths

| Path | Purpose |
| --- | --- |
| `launch.py` | Thin application entry point; delegates environment preparation and startup to `modules/launch_utils.py`. |
| `webui.py` | Gradio UI/API initialization and server lifecycle. |
| `modules/` | Core Python implementation, including API, model backends, processing, UI, and AMD/DirectML/ONNX support. |
| `javascript/` | Browser-side UI behavior; covered by ESLint. |
| `scripts/` | Built-in generation and post-processing scripts exposed by the UI. |
| `extensions-builtin/` | Extensions shipped with the application. Ruff excludes extension directories. |
| `test/` | Pytest unit and live-API tests, fixtures, input assets, and generated test outputs. |
| `configs/` | Stable Diffusion and backend inference configuration files. |
| `html/` | UI HTML fragments and bundled license page. |
| `localizations/` | Drop-in localization files. |
| `models/` | Local runtime model files; ignored by Git. Do not commit checkpoints or generated weights. |
| `requirements*.txt` | Runtime, pinned, NPU, and test Python dependency sets. |
| `.github/workflows/` | Lint, test, and PR-base CI definitions. |

## Generated and local-only files

- Do not commit virtual environments, `node_modules/`, models/checkpoints, `outputs/`, coverage data, local config, or test outputs; these are ignored in `.gitignore`.
- `webui-user.sh` and `webui-user.bat` are local override files and are ignored.
