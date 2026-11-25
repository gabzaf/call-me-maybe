# Call Me Maybe: Developer Notes

Terms are defined in [CONCEPTS.md](CONCEPTS.md).

## Overview

The program maps a natural-language prompt to a function call: a function name from a fixed set, plus typed arguments that match that function's [schema](CONCEPTS.md#schema). The model is [Qwen3-0.6B](CONCEPTS.md#qwen3), loaded through the provided `llm_sdk` (`Small_LLM_Model`, a wrapper around a [Hugging Face](CONCEPTS.md#hugging-face) [causal LM](CONCEPTS.md#causal-lm)).

Input:

- `data/input/functions_definition.json`: the available functions, each with `name`, `description`, `parameters` (name → `{"type": ...}`) and `returns`.
- `data/input/function_calling_tests.json`: the prompts to process.

Output: `data/output/function_calling_results.json`, an array with one object per prompt:

```json
{
  "prompt": "What is the sum of 40 and 2?",
  "name": "fn_add_numbers",
  "parameters": {"a": 40.0, "b": 2.0}
}
```

## Setup

The project is run with:

```bash
uv sync
uv run python -m src
```

- `uv sync` reads `pyproject.toml` and installs the dependencies into a local environment (`.venv`). This includes `torch` and `transformers`, which `llm_sdk` needs to load the model.
- `uv run python -m src` runs the code inside that environment.

So `pyproject.toml` must exist before any code can run.

## Scope in the tool-calling pipeline

| Step | Responsibility | In this project |
|------|----------------|-----------------|
| 1. Context | Prompt + tool schemas are fed to the model | Yes ([prompt construction](CONCEPTS.md#prompt)) |
| 2. Call generation | Model emits `{name, parameters}` | **Yes, core of the project** |
| 3. Execution | Host code dispatches the call | No |
| 4. Result injection | Tool output appended to the context | No |
| 5. Final response | Model generates the user-facing answer | No |

Nothing is executed. The output is the call, not its result. Keeping generation and execution separate means the model can only propose a call; the host decides whether to run it, after validating name and argument types against the schema.

## Constrained decoding

Small models are unreliable at structured output when asked through the prompt alone: they emit prose around the JSON, drop quotes, [hallucinate keys](CONCEPTS.md#hallucination) or pick nonexistent functions. Prompting is not used to enforce the format. Instead, [decoding is constrained](CONCEPTS.md#constrained-decoding) at the [token level](CONCEPTS.md#token).

At each generation step:

1. Get the next-token [logits](CONCEPTS.md#logits) with `get_logits_from_input_ids(input_ids)`.
2. Compute the set of tokens that keep the output a valid prefix of a schema-conforming JSON object.
3. Set the logit of every other token to [`-inf`](CONCEPTS.md#masking).
4. Pick the next token from the remaining logits ([greedy argmax](CONCEPTS.md#greedy-argmax)), append it and repeat until the object is closed.

What the [grammar](CONCEPTS.md#grammar) constrains at each position:

- **Structure:** fixed keys and punctuation (`{"name": "`, `", "parameters": {`, ...) are forced, so they need no model choice.
- **Function name:** only tokens that extend a prefix of one of the defined function names are allowed.
- **Parameter keys:** once the name is fixed, only that function's parameter names, in schema order.
- **Values:** by declared type. `number` allows digits, `-`, `.` and exponent characters in valid order; `string` allows any token that does not prematurely close or break the string (escaped quotes handled); `boolean` allows `true` / `false`.

The model still decides *which* function and *which* values. It can no longer decide the *shape*, so every output parses and matches the schema by construction.

### Tokenizer handling

Tokens are [BPE subwords](CONCEPTS.md#bpe), not characters. A single token can span a key boundary, a quote and a value at once. The allowed-token check therefore tests the token's decoded text against the current grammar state character by character, not token by token. The [vocabulary](CONCEPTS.md#tokenizer) is read once from `get_path_to_vocab_file()` / `get_path_to_tokenizer_file()`, and decoded token strings are cached so masking stays cheap per step.

## Design decisions

*To be filled in as the implementation lands: data structures for the grammar state, precomputation strategy for allowed-token sets, prompt template, decoding strategy.*

## Performance

*To be filled in: accuracy on the test prompts, generation time per prompt, failure cases.*

## Challenges

*To be filled in.*
