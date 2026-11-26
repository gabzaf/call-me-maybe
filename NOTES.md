# Notes

What I learned while building this project. Terms are defined in [CONCEPTS.md](CONCEPTS.md).

## Finding the next step

Start from the end and work backwards.

1. **How will the project be run?** Find the run command in the subject: `uv sync` then `uv run python -m src`.
2. **What does that command need?** `uv sync` needs `pyproject.toml`. The code needs `torch` and `transformers` installed.
3. **What depends on nothing?** That's the first step. Here: `pyproject.toml`.

Rule: environment first, then code.

## pyproject.toml

- **The name must be exact.** Tools look for `pyproject.toml`, so `uv sync` won't find `config.toml`. The `.toml` only says what format is inside.
- **`uv sync` is the only command the reviewer runs.** So [pyproject.toml](CONCEPTS.md#pyprojecttoml) must list everything the project needs.
- **Local packages need a source.** `llm-sdk` isn't published online, so without this uv searches the internet and `uv sync` fails:

  ```toml
  [tool.uv.sources]
  llm-sdk = { path = "llm_sdk" }
  ```

- **Let `uv` write it.** Writing it by hand, I forgot `[tool.uv.sources]` and typed `[dependencies-groups]` instead of `[dependency-groups]`. `uv init` + `uv add` would have got both right.
