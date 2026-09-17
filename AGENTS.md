# AGENTS.md

## Prerequisites

- Python 3.10.6 is the CI version; `webui.sh` falls back to the available `python3`.
- Node.js 18 is the JavaScript lint CI version.
- Linux packages are listed by distribution in `README.md`; Debian/Ubuntu:

  ```bash
  sudo apt install wget git python3 python3-venv libgl1 libglib2.0-0
  ```

## Setup and Run

- Linux automatic setup and development server (creates `venv`, installs dependencies, and launches the UI):

  ```bash
  ./webui.sh
  ```

- Put local launch settings such as `python_cmd`, `venv_dir`, and `COMMANDLINE_ARGS` in `webui-user.sh`; do not edit `webui.sh` for local configuration.
- Select an AMD backend through launch arguments:

  ```bash
  ./webui.sh --use-directml
  ./webui.sh --use-zluda
  ```

- Windows setup and server: run `webui-user.bat`.
- Install the JavaScript lint dependency:

  ```bash
  npm install
  ```

- Reproduce the CI test environment setup after activating a Python environment:

  ```bash
  python -m pip install wait-for-it -r requirements-test.txt
  TORCH_INDEX_URL=https://download.pytorch.org/whl/cpu python launch.py --skip-torch-cuda-test --exit
  ```

- There is no separate build command.

## Tests

- Most tests call the API at `http://127.0.0.1:7860`; start the CI-style test server first:

  ```bash
  python -m coverage run --data-file=.coverage.server launch.py --skip-prepare-environment --skip-torch-cuda-test --test-server --do-not-download-clip --no-half --disable-opt-split-attention --use-cpu all --api-server-stop
  ```

- Run the full suite in another shell:

  ```bash
  wait-for-it --service 127.0.0.1:7860 -t 20
  python -m pytest -vv --junitxml=test/results.xml --cov . --cov-report=xml --verify-base-url test
  ```

- Run one test file:

  ```bash
  python -m pytest -vv --verify-base-url test/test_txt2img.py
  ```

- Run one test:

  ```bash
  python -m pytest -vv --verify-base-url test/test_txt2img.py::test_txt2img_simple_performed
  ```

- `test/test_torch_utils.py` does not require the API server:

  ```bash
  python -m pytest -vv test/test_torch_utils.py
  ```

- Stop the test server:

  ```bash
  curl -X POST http://127.0.0.1:7860/sdapi/v1/server-stop
  ```

## Lint and Fix

- Run the same Python lint version as CI:

  ```bash
  python -m pip install ruff==0.3.3
  ruff check .
  ```

- Apply safe Ruff fixes:

  ```bash
  ruff check --fix .
  ```

- Run JavaScript lint and auto-fix commands from `package.json`:

  ```bash
  npm run lint
  npm run fix
  ```

- Run `ruff check .` and `npm run lint` before committing. No repository-wide formatter is configured.

## Pull Requests

- Target `dev`. The `Pull requests can't target master branch` workflow fails PRs opened directly against `master`.
- Source branch naming: no enforced naming pattern is defined in tracked repository configuration.
- Commit messages: no enforced format is defined in tracked repository configuration; keep subjects concise and imperative.
- Complete `.github/pull_request_template.md`: description, code summary, fixed issues, screenshots/videos when relevant, self-review, style, and tests.
- Required repository CI checks:
  - `ruff`
  - `eslint`
  - `tests on CPU with empty model`
- The `Pull requests can't target master branch` guard intentionally fails when a PR targets `master`; target `dev` so it is not triggered.

## Key Directories

- `modules/`: Python application, API, model/backend, processing, and UI implementation.
- `test/`: pytest suite, fixtures, input assets, and generated test outputs.
- `javascript/`: browser-side UI behavior; linted by ESLint with root `script.js`.
- `scripts/`: built-in generation and post-processing scripts loaded by the WebUI.
- `extensions-builtin/`: bundled extensions maintained with the application.
- `extensions/`: installation location for user extensions; excluded from Ruff.
- `configs/`: Stable Diffusion and backend model configuration files.
- `models/`: runtime model/checkpoint locations; large model files should remain untracked.
- `html/`: reusable UI fragments and license content.
- `localizations/`: translation files.
- `textual_inversion_templates/`: prompt templates for textual inversion and hypernetwork training.
- `.github/workflows/`: lint, test, and PR-target CI definitions.
