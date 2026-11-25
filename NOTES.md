# Notes

What I learned while building this project.

## Finding the next step

Start from the end and work backwards.

1. **How will the project be run?** Find the run command in the subject: `uv sync` then `uv run python -m src`.
2. **What does that command need?** `uv sync` needs `pyproject.toml`. The code needs `torch` and `transformers` installed.
3. **What depends on nothing?** That's the first step. Here: `pyproject.toml`.

Rule: environment first, then code.
