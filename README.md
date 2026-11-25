*This project has been created as part of the 42 curriculum by gamado-x.*

# Call Me Maybe

## Description

Teaching a small AI model to turn a question into a function call instead of an answer.

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

## Keeping the output valid

Small models like Qwen3 0.6B often get JSON wrong. They add extra text, forget quotes or invent fields. To stop this, the program checks every word the model is about to write and blocks anything that would break the format. The model still chooses the function and the values, but the output always comes out as valid JSON.

For the technical details, see [DEV.md](DEV.md).
