# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Qlib is Microsoft's AI-oriented quantitative investment platform covering the full workflow from data processing, model training, backtesting to strategy execution.

- **Package name**: `pyqlib`
- **Python versions**: 3.8, 3.9, 3.10, 3.11, 3.12

## Common Commands

### Installation

```bash
# Standard installation (includes Cython compilation)
make install

# Full development environment (all optional dependencies)
make dev

# Specific dependency groups
make dev      # pytest, statsmodels
make lint     # black, pylint, mypy, flake8, nbqa
make rl       # tianshou, torch (numpy<2.0.0 required)
```

### Testing

```bash
# Run all tests
pytest tests/

# Run specific test file
pytest tests/test_workflow.py

# Run tests excluding slow tests
pytest -m "not slow" tests/

# Run specific test class/method
pytest tests/test_workflow.py::TestClass::test_method
```

**Note**: RL tests are automatically skipped on non-Linux platforms.

### Linting

```bash
# Run all lint checks
make lint

# Individual checks
make black    # Code formatting (120 char line length)
make pylint   # Code quality
make flake8   # Style check
make mypy     # Type checking
make nbqa     # Jupyter notebook check
```

### Build & Package

```bash
make build    # Build wheel package
make clean    # Remove build artifacts
```

## Code Architecture

### Core Modules

| Module | Location | Purpose |
|--------|----------|---------|
| `qlib.data` | `qlib/data/` | High-performance data access, storage, caching. Core API: `D.features()`, `D.calendar()` |
| `qlib.model` | `qlib/model/` | Model base classes, trainers, ensemble learning, risk models |
| `qlib.backtest` | `qlib/backtest/` | Trading simulation, account management, executors, reporting |
| `qlib.workflow` | `qlib/workflow/` | Experiment management via MLflow, Recorder pattern for tracking |
| `qlib.strategy` | `qlib/strategy/` | Trading strategy base classes |
| `qlib.rl` | `qlib/rl/` | Reinforcement learning framework for order execution |
| `qlib.contrib` | `qlib/contrib/` | Contributed models, strategies, data handlers, reports |
| `qlib.utils` | `qlib/utils/` | Utilities: time handling, parallelization, serialization |

### Key Patterns

- **Initialization**: Always call `qlib.init(provider_uri='...')` before using any qlib functionality
- **Workflow execution**: Use `qrun <config.yaml>` to run end-to-end pipelines defined in YAML
- **Experiment tracking**: Results stored in `mlruns/` directory via MLflow integration
- **Recorder pattern**: Use `R` (from `qlib.workflow`) to access experiment recorder for logging parameters, metrics, and artifacts

### Performance-Critical Code

- `qlib/data/_libs/rolling.pyx` and `qlib/data/_libs/expanding.pyx` are Cython modules for rolling calculations
- These are compiled during `make install` or when shared objects don't exist

## Code Style

- **Line length**: 120 characters (black format)
- **Pre-commit hooks**: black and flake8 configured in `.pre-commit-config.yaml`
- Type hints checked with mypy

## CI/CD

Tests run on: Windows, Ubuntu, macOS with Python 3.8-3.12 matrix.
