# AGENTS.md

## Repository overview

- Python 3.10 is the CI runtime; `webui.sh` prefers `python3.10` and falls back to `python3`.
- The application is a Gradio Stable Diffusion Web UI with AMD DirectML, ZLUDA, ROCm, and ONNX support.
- Run commands from the repository root unless noted otherwise.

## Setup and run

### Linux

Install the system prerequisites listed in `README.md` for your distribution. Debian/Ubuntu:

```bash
sudo apt install wget git python3 python3-venv libgl1 libglib2.0-0
```

Create the virtual environment, install runtime dependencies, and launch the development server:

```bash
./webui.sh
```

- `webui.sh` creates `venv/` and lets `launch.py` prepare missing dependencies on first launch.
- Put local overrides such as `python_cmd`, `venv_dir`, `TORCH_COMMAND`, and `COMMANDLINE_ARGS` in `webui-user.sh`; do not edit `webui.sh` for local configuration.
- Backend flags documented by the repository include `--use-directml` and `--use-zluda`, passed through `COMMANDLINE_ARGS` or directly to the launcher.

CI-compatible dependency/setup commands:

```bash
pip install wait-for-it -r requirements-test.txt
python launch.py --skip-torch-cuda-test --exit
npm i --ci
```

### Windows

```bat
webui-user.bat
```

- `webui.bat` creates `venv\` and runs `launch.py`.
- Put local `PYTHON`, `GIT`, `VENV_DIR`, and `COMMANDLINE_ARGS` values in `webui-user.bat`.

### Build

- No separate build command or compiled artifact is defined. `launch.py` prepares the Python environment, then starts the application.

## Tests

Install test dependencies and prepare the CPU test environment:

```bash
pip install wait-for-it -r requirements-test.txt
TORCH_INDEX_URL=https://download.pytorch.org/whl/cpu python launch.py --skip-torch-cuda-test --exit
```

Most tests call the API at `http://127.0.0.1:7860`. Start the CI-equivalent test server in one terminal:

```bash
python launch.py --skip-prepare-environment --skip-torch-cuda-test --test-server --do-not-download-clip --no-half --disable-opt-split-attention --use-cpu all --api-server-stop
```

Run the full suite in another terminal after the server is ready:

```bash
python -m pytest -vv --verify-base-url test
```

Run a test file:

```bash
python -m pytest -vv --verify-base-url test/test_txt2img.py
```

Run one test:

```bash
python -m pytest -vv --verify-base-url test/test_txt2img.py::test_txt2img_simple_performed
```

Run the self-contained Torch utility test without the API server:

```bash
python -m pytest -vv test/test_torch_utils.py
```

CI adds JUnit and coverage output with:

```bash
python -m pytest -vv --junitxml=test/results.xml --cov . --cov-report=xml --verify-base-url test
```

## Lint and formatting

Install the Python linter version used by CI:

```bash
pip install ruff==0.3.3
```

Run these before committing:

```bash
ruff check .
npm run lint
```

Apply the repository's ESLint fixes:

```bash
npm run fix
```

- Ruff configuration is in `pyproject.toml`.
- ESLint configuration and exclusions are in `.eslintrc.js` and `.eslintignore`.
- No repository-wide formatter command is defined.

## Pull requests

- Target `dev`. `.github/workflows/warns_merge_master.yml` fails pull requests opened directly against `master` and states that development normally happens on `dev`.
- Branch naming: no prefix or pattern is defined in the repository.
- Commit format: no required commit-message format is defined in the repository.
- Complete `.github/pull_request_template.md`: describe the goal and code changes, link fixed issues, add screenshots/videos when relevant, self-review, follow the linked style guide, and run tests.
- Ensure the repository CI checks pass:
  - `ruff` from `.github/workflows/on_pull_request.yaml`
  - `eslint` from `.github/workflows/on_pull_request.yaml`
  - `tests on CPU with empty model` from `.github/workflows/run_tests.yaml`
  - `Pull requests can't target master branch` from `.github/workflows/warns_merge_master.yml`

## Key directories

- `modules/` — core Python application, processing, model, UI, and shared runtime code.
- `modules/api/` — HTTP API implementation and API models.
- `modules/dml/` — DirectML-specific runtime support.
- `modules/onnx_impl/` — ONNX Runtime and Olive integration.
- `javascript/` — browser-side UI behavior; root `script.js` is the main shared script.
- `scripts/` — built-in selectable processing and UI scripts.
- `extensions-builtin/` — bundled extensions shipped with the application.
- `extensions/` — locally installed third-party extensions; excluded from Ruff checks.
- `configs/` — model and backend configuration, including `configs/olive/`.
- `models/` — runtime model storage organized by model type.
- `test/` — pytest suite, fixtures in `test/test_files/`, and generated output in `test/test_outputs/`.
- `html/` — static HTML fragments and license content.
- `localizations/` — UI translation JSON files.
- `embeddings/` and `textual_inversion_templates/` — textual inversion data and training templates.
