# AGENTS.md

## Repository overview

- Python Stable Diffusion WebUI with AMD GPU support through DirectML, ZLUDA, and ROCm-related launch paths.
- Python 3.10.6 is the CI baseline; the Linux launcher prefers `python3.10` and falls back to `python3`.
- Run commands from the repository root unless noted otherwise.

## Setup and run

### Linux

Install the platform prerequisites listed in `README.md`, then let the launcher create `venv/` and install runtime dependencies:

```bash
./webui.sh
```

Configure Python, the virtual environment, Torch installation, or launch arguments in `webui-user.sh`. Examples:

```bash
export COMMANDLINE_ARGS="--use-zluda"
./webui.sh
```

```bash
export COMMANDLINE_ARGS="--use-directml"
./webui.sh
```

Prepare dependencies without keeping the server running, as CI does:

```bash
python launch.py --skip-torch-cuda-test --exit
```

### Windows

Run the checked-in launcher; it performs first-run setup and starts the server:

```bat
webui-user.bat
```

### Test and lint tooling

```bash
python -m pip install wait-for-it -r requirements-test.txt
python -m pip install ruff==0.3.3
npm i --ci
```

- There is no separate build command in this repository.

## Tests

Most tests call a live WebUI at `http://127.0.0.1:7860`. Start a test server in one terminal:

```bash
python launch.py --test-server
```

For a CPU-only baseline, use the CI launch options:

```bash
python launch.py --skip-prepare-environment --skip-torch-cuda-test --test-server --do-not-download-clip --no-half --disable-opt-split-attention --use-cpu all --api-server-stop
```

Run tests in another terminal:

```bash
# Full suite
python -m pytest -vv --verify-base-url test

# One file
python -m pytest -vv --verify-base-url test/test_txt2img.py

# One test
python -m pytest -vv --verify-base-url test/test_txt2img.py::test_txt2img_simple_performed

# Standalone utility test that does not call the live API
python -m pytest -vv test/test_torch_utils.py
```

CI runs the suite with coverage and JUnit output:

```bash
python -m pytest -vv --junitxml=test/results.xml --cov . --cov-report=xml --verify-base-url test
```

## Lint and format

Run before committing:

```bash
# Python; matches the `ruff` CI check
ruff check .

# JavaScript; matches the `eslint` CI check
npm run lint
```

The repository provides an ESLint autofix command:

```bash
npm run fix
```

- No repository-wide Python formatter command is configured.

## Pull requests

- Create a topic branch; do not submit from a clone's `master` or `main` branch.
- Target `dev`. PRs targeting `master` fail the `Pull requests can't target master branch` workflow.
- No branch-prefix convention is documented; use a short, descriptive branch name.
- No commit-message format is documented; keep commits scoped to the change.
- Keep unrelated changes in separate PRs. Do not submit reformat-only changes.
- Bug fixes must include reproduction steps.
- Complete the PR template: description, summary, fixed issues, relevant screenshots/videos, self-review, style, and test checklist.
- Installation-script or dependency changes must be verified from a fresh default Windows installation unless the changed path is explicitly non-Windows.
- Pass the applicable repository checks defined in `.github/workflows/`:
  - `Linter / ruff`
  - `Linter / eslint`
  - `Tests / tests on CPU with empty model`
  - `Pull requests can't target master branch / check` enforces the target-branch rule.

## Key directories

- `modules/`: core Python application, launch, API, processing, UI, and model integration code.
- `javascript/`: browser-side WebUI behavior; linted by ESLint with `script.js`.
- `extensions-builtin/`: extensions shipped with the application.
- `extensions/`: locally installed third-party extensions; excluded from Ruff checks.
- `test/`: pytest suite, fixtures, test images, and generated test output location.
- `models/`: runtime model storage organized by model type.
- `configs/`: Stable Diffusion model configuration files.
- `scripts/`: built-in user-facing processing scripts.
- `html/`: reusable HTML fragments used by the UI.
- `localizations/`: UI translation JSON files.
- `embeddings/`: textual-inversion embedding storage.
- `textual_inversion_templates/`: prompt templates for textual-inversion training.
- `.github/workflows/`: lint, test, and pull-request target CI definitions.

## Important entry points and configuration

- `launch.py`: environment preparation and application launcher used by CI.
- `webui.py`: WebUI initialization and server startup.
- `webui.sh` / `webui-user.bat`: supported Linux and Windows setup/start entry points.
- `webui-user.sh`: user-overridable Linux launcher variables; edit this instead of `webui.sh`.
- `requirements_versions.txt`: pinned Python 3.10.6 runtime dependencies.
- `requirements-test.txt`: pytest and coverage-related dependencies.
- `pyproject.toml`: Ruff and pytest configuration.
- `package.json` / `.eslintrc.js`: JavaScript lint and autofix commands/rules.
