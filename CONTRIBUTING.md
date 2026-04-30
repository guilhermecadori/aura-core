# Contributing to AURA

Thanks for your interest in contributing. This document covers commit conventions, branch and PR workflow, local dev setup, and code quality expectations.

## Commit Messages

This repo uses [Conventional Commits](https://www.conventionalcommits.org/), enforced via [commitlint](./commitlint.config.mjs) (config: `@commitlint/config-conventional`) on every commit and PR title.

Format:

```
<type>(<scope>): <subject>
```

Common types: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `ci`, `build`, `perf`, `style`.

Examples:

```
feat(agent): add tool-use loop with retry
fix(rag): handle empty corpus on first ingest
docs(adr): record vLLM-over-Triton decision
chore(repo): bump ruff to 0.4
```

PR titles must also follow this format — enforced by the [semantic PR title](.github/workflows/semantic-pr-title.yml) workflow.

## Branch and PR Workflow

1. Branch off `main`. Use a descriptive prefix matching the commit type, e.g. `feat/agent-tool-use`, `fix/rag-empty-corpus`, `docs/contributing`, `chore/repo-hygiene`.
2. Keep PRs small and focused. One logical change per PR.
3. Open a PR against `main`. The PR title follows Conventional Commits (see above).
4. CI must pass: commitlint, semantic PR title, gitleaks, and python-quality (when Python files change).
5. Squash-merge is the default — keep the squash commit message Conventional-Commits-compliant.

## Local Dev Setup

### Prerequisites

- **Node.js 20+** — for commitlint
- **Python 3.12+** — for backend, agent, and tooling
- **Git** — with hooks enabled

### Install

```bash
# Node deps (commitlint)
npm install

# Python: create a venv and install quality tools
python -m venv .venv
source .venv/bin/activate            # Windows: .venv\Scripts\activate
pip install ruff mypy pytest pytest-cov

# When pyproject.toml declares dependencies, also:
pip install -e .
```

### Run Quality Checks

These match what CI runs in [`.github/workflows/python-quality.yml`](.github/workflows/python-quality.yml):

```bash
ruff check .              # lint
ruff format --check .     # format check (use `ruff format .` to apply)
mypy src/                 # type check (once src/ exists)
pytest                    # tests with coverage (once tests/ exists)
```

Lint and format settings live in [`pyproject.toml`](./pyproject.toml).

## Code Quality Expectations

- **Ruff** — pycodestyle, pyflakes, isort, bugbear, pyupgrade, simplify, comprehensions, ruff-specific. Line length 100.
- **Mypy** — strict mode (`strict = true`), Python 3.12 target.
- **Pytest** — branch coverage on `src/`, strict markers and config, term-missing + xml reports.
- **Secrets** — never commit credentials. Gitleaks runs on every push and PR.

## Questions

Open an issue or start a discussion on the repo.
