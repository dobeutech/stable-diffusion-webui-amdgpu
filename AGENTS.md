# AGENTS.md

## Project snapshot

- Python/Gradio Stable Diffusion Web UI fork with AMD GPU support.
- Supported backends include ROCm, DirectML, ZLUDA, ONNX Runtime, and Olive.
- Python 3.10, 3.11, and 3.12 are accepted on Linux; CI uses Python 3.10.6 for tests and Python 3.11 for linting.
- This is a single application, not a monorepo.
- Run commands from the repository root.

## Setup and development

### Linux prerequisites

Use the command for the host distribution:

```bash
# Debian/Ubuntu
sudo apt install wget git python3 python3-venv libgl1 libglib2.0-0

# Fedora/Red Hat
sudo dnf install wget git python3 gperftools-libs libglvnd-glx

# openSUSE
sudo zypper install wget git python3 libtcmalloc4 libglvnd

# Arch
sudo pacman -S wget git python3
```

### Install and run

```bash
# Creates venv/, installs Python/runtime dependencies, and starts the UI.
./webui.sh

# Install JavaScript lint tooling. This is the command used in CI.
npm i --ci
```

- Windows entry point: `webui-user.bat`, which delegates to `webui.bat`.
- Put local launcher configuration in `webui-user.sh`; `webui.sh` explicitly says not to edit it.
- Pass backend flags through the launcher, for example `./webui.sh --use-rocm`, `./webui.sh --use-directml`, or `./webui.sh --use-zluda`.
- There is no separate build command; the Python launcher prepares the environment and starts the app.

### CPU test environment (matches CI)

```bash
python -m pip install wait-for-it -r requirements-test.txt
TORCH_INDEX_URL=https://download.pytorch.org/whl/cpu \
  WEBUI_LAUNCH_LIVE_OUTPUT=1 \
  python launch.py --skip-torch-cuda-test --exit
```

## Tests

Most tests exercise the HTTP API and require the test server. Start it in one terminal:

```bash
python -m coverage run \
  --data-file=.coverage.server \
  launch.py \
  --skip-prepare-environment \
  --skip-torch-cuda-test \
  --test-server \
  --do-not-download-clip \
  --no-half \
  --disable-opt-split-attention \
  --use-cpu all \
  --api-server-stop
```

Then run tests in another terminal:

```bash
# Full suite, as run in CI
wait-for-it --service 127.0.0.1:7860 -t 20
python -m pytest -vv --junitxml=test/results.xml --cov . --cov-report=xml --verify-base-url test

# One test file
python -m pytest -vv --verify-base-url test/test_txt2img.py

# One API test
python -m pytest -vv --verify-base-url test/test_txt2img.py::test_txt2img_simple_performed

# Focused unit test that does not need the API server
python -m pytest -vv test/test_torch_utils.py

# One parameterized unit-test case
python -m pytest -vv 'test/test_torch_utils.py::test_get_param[True]'
```

Stop the CI-style server when finished:

```bash
curl -XPOST http://127.0.0.1:7860/sdapi/v1/server-stop
```

## Lint and formatting

Run both CI linters before committing:

```bash
ruff check .
npm run lint
```

Available automatic lint fixes:

```bash
ruff check --fix .
npm run fix
```

- CI installs Ruff with `python -m pip install ruff==0.3.3`.
- No standalone formatter is configured in this repository.
- Ruff excludes `extensions/` and `extensions-disabled/`; ESLint exclusions are in `.eslintignore`.

## Pull requests

- Base branch: `master` is the remote's default and only published branch.
- Known CI inconsistency: the inherited `Pull requests can't target master branch` workflow fails PRs against `master` and recommends `dev`, but this remote does not publish a `dev` branch.
- Source branch naming: no naming convention is enforced in repository configuration.
- Commit messages: no required format is enforced in repository configuration.
- Required-check/branch-protection settings are not checked into the repository; the workflows below are the checks defined in source.
- PR description must summarize the change and list fixed issues; include screenshots or videos for visual changes.
- Complete the PR-template checklist: read the contributing guide, self-review, follow style guidance, and run tests.
- All paths are covered by `CODEOWNERS` entry `* @AUTOMATIC1111`.
- CI checks expected to pass:
  - `Linter / ruff`: `ruff check .`
  - `Linter / eslint`: `npm run lint`
  - `Tests / tests on CPU with empty model`: full pytest/API suite
  - `Pull requests can't target master branch / check`: applies when a PR targets `master` and fails by design

## Key directories

| Path | Purpose |
| --- | --- |
| `launch.py` | Application entry point; delegates environment preparation and startup to `modules/launch_utils.py`. |
| `webui.py` | Gradio UI/API initialization and server lifecycle. |
| `modules/` | Core Python application, launch/runtime logic, processing, model integrations, and UI code. |
| `modules/api/` | HTTP API implementation and API models. |
| `modules/dml/` | DirectML-specific runtime support. |
| `modules/onnx_impl/` | ONNX Runtime and Olive integration. |
| `modules/flash_attn_triton_amd/` | AMD Triton flash-attention implementation. |
| `test/` | Pytest suite, fixtures, input assets, and generated test outputs. |
| `javascript/` | Browser-side UI behavior. |
| `html/` | HTML fragments and license content used by the UI. |
| `scripts/` | Built-in generation and post-processing scripts loaded by the app. |
| `extensions-builtin/` | Extensions shipped with the application. |
| `extensions/` | User-installed extensions; excluded from Ruff checks. |
| `models/` | Runtime model storage organized by model type. |
| `configs/` | Stable Diffusion and Olive model configuration files. |
| `localizations/` | Drop-in UI localization files. |
| `textual_inversion_templates/` | Prompt templates for textual inversion and hypernetwork training. |
| `requirements*.txt` | Runtime, pinned, NPU, and test Python dependency sets. |
| `.github/workflows/` | Ruff, ESLint, pytest, and PR-target CI definitions. |

## Generated and local-only files

- Do not commit virtual environments, `node_modules/`, models/checkpoints, `outputs/`, coverage data, local configuration, or test outputs; these paths are ignored in `.gitignore`.
- `webui-user.sh` and `webui-user.bat` are ignored local override files.
