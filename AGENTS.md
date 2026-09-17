# AGENTS.md

## Repository overview

- Python 3.10 is the CI runtime; `README.md` specifically recommends Python 3.10.6 on Windows.
- `launch.py` prepares dependencies and starts the application.
- `webui.sh` / `webui-user.sh` are the Linux launch and local-configuration scripts.
- `webui.bat` / `webui-user.bat` are the Windows equivalents.
- There is no separate build command in this repository.

## Setup and run

### Linux

The launcher creates `venv/`, installs missing application dependencies, and starts the Web UI:

```bash
./webui.sh
```

Set local launch variables such as `python_cmd`, `COMMANDLINE_ARGS`, or `TORCH_COMMAND` in `webui-user.sh`; do not edit `webui.sh` for local configuration.

AMD launch modes documented by the repository:

```bash
./webui.sh --use-directml
./webui.sh --use-zluda
```

### Windows

Run:

```bat
webui-user.bat
```

Put local launch options in `webui-user.bat`.

### CI-compatible test environment

The test workflow installs and prepares dependencies with:

```bash
python3.10 -m venv venv
source venv/bin/activate
python -m pip install wait-for-it -r requirements-test.txt
TORCH_INDEX_URL=https://download.pytorch.org/whl/cpu python launch.py --skip-torch-cuda-test --exit
```

## Tests

Most tests call the API at `http://127.0.0.1:7860`. Start the CPU test server in one terminal:

```bash
python launch.py --skip-prepare-environment --skip-torch-cuda-test --test-server --do-not-download-clip --no-half --disable-opt-split-attention --use-cpu all --api-server-stop
```

Then run the full suite in another terminal:

```bash
wait-for-it --service 127.0.0.1:7860 -t 20
python -m pytest -vv --junitxml=test/results.xml --cov . --cov-report=xml --verify-base-url test
```

Run one test file:

```bash
python -m pytest -vv --verify-base-url test/test_txt2img.py
```

Run one test function:

```bash
python -m pytest -vv --verify-base-url test/test_txt2img.py::test_txt2img_simple_performed
```

Run the standalone Torch utility tests without an API server:

```bash
python -m pytest -vv test/test_torch_utils.py
```

## Lint and format

Install the exact lint dependencies used by CI:

```bash
python -m pip install ruff==0.3.3
npm i --ci
```

Run before committing:

```bash
ruff check .
npm run lint
```

Apply the repository's available JavaScript auto-fix command:

```bash
npm run fix
```

No Python formatter command is configured.

## Pull requests

- Target `dev`; `.github/workflows/warns_merge_master.yml` fails pull requests targeting `master`.
- No head-branch naming convention is defined in repository files.
- No commit-message format or commit-lint check is defined in repository files.
- Complete `.github/pull_request_template.md`: describe the goal and changes, link fixed issues, add screenshots/videos when relevant, self-review, follow the linked style guide, and run tests.
- CI workflow checks: `ruff`, `eslint`, and `tests on CPU with empty model`.

## Key directories

- `modules/` — core Python application, launch support, processing, model integration, UI, and API code.
- `modules/api/` — API implementation.
- `modules/dml/`, `modules/onnx_impl/` — DirectML and ONNX support.
- `modules/flash_attn_triton_amd/` — AMD Triton flash-attention implementation.
- `scripts/` — built-in selectable processing and post-processing scripts.
- `extensions-builtin/` — extensions shipped with the repository.
- `extensions/` — locally installed third-party extensions; excluded from Ruff checks.
- `javascript/`, `script.js`, `style.css` — browser-side behavior and global styling.
- `html/` — UI fragments and static HTML assets.
- `test/` — pytest suite, fixtures, input files, and generated test outputs.
- `configs/` — model inference configuration files.
- `models/` — local model weights organized by model type.
- `embeddings/` — textual-inversion embeddings.
- `localizations/` — translation files.
- `textual_inversion_templates/` — prompt templates for training.
