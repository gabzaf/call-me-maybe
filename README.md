*This project has been created as part of the 42 curriculum by gamado-x.*

# Call Me Maybe

## Description

Teaching a small [AI model](CONCEPTS.md#llm) to turn a question into a [function call](CONCEPTS.md#function-calling) instead of an answer.

Ask *"What is the sum of 40 and 2?"* and the program won't answer `42`. It picks the function that solves the problem and fills in its values:

```json
{
  "prompt": "What is the sum of 40 and 2?",
  "name": "fn_add_numbers",
  "parameters": {"a": 40.0, "b": 2.0}
}
```

This is how AI assistants use tools: the model chooses the tool, and the program uses it.

## How tool calling works

Tool calling happens in five steps. **This project does step 2 only.**

1. **Ask.** The model gets the question and a list of tools it can use.
2. **Choose.** The model picks a tool and fills in the values (the JSON above).
3. **Run.** The program, not the model, runs the tool: `40 + 2 = 42`.
4. **Report.** The result `42` goes back to the model.
5. **Answer.** The model replies: *"The sum of 40 and 2 is 42."*

The model only decides what to do. The program decides whether to actually do it. This split makes it safer: the model can never run anything on its own.

## Constrained decoding

Small models like [Qwen3 0.6B](CONCEPTS.md#qwen3) often get [JSON](CONCEPTS.md#json) wrong. They add extra text, forget quotes or [invent fields](CONCEPTS.md#hallucination).

**[Constrained decoding](CONCEPTS.md#constrained-decoding)** fixes this. The model writes its answer one piece at a time. Before each piece, the program blocks every option that would break the format. The model picks only from what's left.

So the model still chooses the function and the values, but the output always comes out as valid JSON.

For the full algorithm, see [DEV.md](DEV.md#constrained-decoding). For terms, see [CONCEPTS.md](CONCEPTS.md).

## Documentation

- [DEV.md](DEV.md): technical details
- [CONCEPTS.md](CONCEPTS.md): terms explained
- [NOTES.md](NOTES.md): what I learned
