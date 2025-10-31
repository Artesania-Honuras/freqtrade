## Quick orientation

Short: Freqtrade is a Python 3.11+ CLI trading bot. The codebase is organized as a Python package (`freqtrade/`) with a small client package (`ft_client/`), a `user_data/` area for runtime config, and `tests/` for unit/online tests.

Key places to look right away:
- `freqtrade/` — main package (CLI entrypoints, trading/backtesting/hyperopt modules)
- `ft_client/` — separate pip-installable client used by CI and packaging
- `freqtrade/plugins/` — plugin extension points (pairlists, protections, etc.)
- `freqtrade/exchange/` — exchange implementations (see `exchange/binance.py`)
- `tests/` and `tests/conftest.py` — test helpers (e.g. `log_has`, `log_has_re`)
- `docs/developer.md` and top-level `README.md` — developer setup and commands

## Big-picture architecture (short)
- CLI -> `freqtrade.__main__` selects a subcommand (trade, backtesting, hyperopt, webserver, etc.).
- Core modules call well-scoped subsystems:
  - Exchange adapter (wrapping CCXT and exchange-specific logic) in `freqtrade/exchange/`.
  - Strategy logic runs as classes that accept candle DataFrames and return signals.
  - Plugins extend behavior (pairlists, protections) under `freqtrade/plugins/`.
  - Persistence: sqlite is used for runtime/trade state; user-controlled via `user_data/`.

Why this matters to an AI coder: changes affecting CLI, config schema, or command docs will require running repository scripts and CI helpers (`build_helpers/*`) — CI enforces the repository is not modified by these scripts.

## Concrete developer workflows (use these exact commands)
- Local dev install (preferred):
  - `./setup.sh --install` (interactive script) — creates `.venv` and installs either `requirements.txt` or `requirements-dev.txt`.
  - Or manual: `pip install -r requirements-dev.txt` then `pip install -e .[all]`
- Run tests locally: from repo root run `pytest` (CI runs `pytest --random-order --durations 20 -n auto`).
- Linters & typechecks used in CI:
  - `isort --check .`
  - `ruff check --output-format=github`
  - `ruff format --check`
  - `mypy freqtrade scripts tests` (CI only on selected runners)
- Docs: `pip install -r docs/requirements-docs.txt && mkdocs serve` (dev) or `mkdocs build` (CI).

CI-specific notes an AI must respect:
- CI runs scripts that update docs and the config schema: `build_helpers/create_command_partials.py` and `build_helpers/extract_config_json_schema.py`. If your change touches CLI or config, run these scripts locally and ensure `git status` remains clean.
- CI will fail if running those scripts modifies tracked files.

## Project-specific coding patterns (examples and where to look)
- Pairlists: implement handlers under `freqtrade/plugins/pairlist/` — copy `VolumePairList.py` and follow `gen_pairlist` / `filter_pairlist` conventions.
- Protections: must inherit `IProtection` and use `date_now` (NOT direct `datetime.now`) to preserve backtest determinism. See `freqtrade/plugins/protections/*` for examples.
- Exchanges: add classes under `freqtrade/exchange/` and register them in `freqtrade/exchange/__init__.py`. See `exchange/binance.py` for stoploss-on-exchange handling.
- Tests: use helpers from `tests/conftest.py` (e.g. `log_has`, `log_has_re`) to assert log messages.

## Integration & external dependencies
- Exchanges: CCXT is used for most exchange interactions.
- Optional/large features: FreqAI (freqai, freqai-rl) and PyTorch — these are behind extra requirement files (`requirements-freqai.txt`, `requirements-freqai-rl.txt`).
- Native optional: TA-Lib, see documentation for platform-specific install notes.

## Quick checklist for PRs that an AI-assisted change should follow
1. Run unit tests: `pytest` (match CI flags for parallelization if needed).
2. Run `python build_helpers/extract_config_json_schema.py` and `python build_helpers/create_command_partials.py` if you touched CLI or config.
3. Ensure linters/typechecks pass: `ruff`, `isort`, `mypy` where applicable.
4. Run `pre-commit install` locally if working in a dev environment; CI runs pre-commit checks.
5. If you modify docs, run `mkdocs build` to ensure no doc generation diffs.

## Helpful file references to cite in code changes
- `docs/developer.md` — developer setup and debug launcher configs
- `setup.sh` — canonical local install flow used by contributors
- `.github/workflows/ci.yml` — exact CI steps and commands (tests, linters, mypy, docs)
- `build_helpers/extract_config_json_schema.py` and `build_helpers/create_command_partials.py` — scripts CI runs that must not change repo state after execution
- `freqtrade/plugins/pairlist/VolumePairList.py` and `freqtrade/exchange/binance.py` — concrete examples of plugin and exchange patterns
- `tests/conftest.py` — test helpers and logging assertions

If any of the above sections are unclear or you want more examples (e.g. step-by-step for adding a new exchange or pairlist), tell me which area to expand and I will iterate. 
