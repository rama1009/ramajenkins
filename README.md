name: lint_python
on: [pull_request, push]
jobs:
  lint_python:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-python@v2
      - run: pip install --upgrade pip wheel
      - run: pip install bandit black codespell flake8 flake8-2020 flake8-bugbear
                         flake8-comprehensions isort mypy pytest pyupgrade safety
      - run: bandit --recursive --skip B101 . || true  # B101 is assert statements
      - run: black --check . || true
      - run: codespell || true  # --ignore-words-list="" --skip="*.css,*.js,*.lock"
      - run: flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
      - run: flake8 . --count --exit-zero --max-complexity=10 --max-line-length=88
                      --show-source --statistics
      - run: isort --check-only --profile black . || true
      - run: pip install -r requirements.txt || pip install --editable . || true
      - run: mkdir --parents --verbose .mypy_cache
      - run: mypy --ignore-missing-imports --install-types --non-interactive . || true
      - run: pytest . || pytest --doctest-modules .
      - run: shopt -s globstar && pyupgrade --py36-plus **/*.py || true
      - run: safety check
