# Concepts

Short definitions of the terms used in [README.md](README.md) and [DEV.md](DEV.md).

## The model

### LLM

Large Language Model: an AI trained on lots of text to predict what comes next. Chat assistants are LLMs.

### Qwen3

A family of open LLMs made by Alibaba. This project uses its smallest version.

[Official site](https://qwenlm.github.io/) · [Qwen3-0.6B model page](https://huggingface.co/Qwen/Qwen3-0.6B)

### Parameters

The numbers inside a model that it learned during training. **0.6B** means 0.6 billion. Fewer parameters make a model faster and lighter, but less reliable.

### Hugging Face

A website and set of Python libraries for sharing and running AI models. The project downloads Qwen3 from there.

[Official site](https://huggingface.co/)

### Causal LM

A model that writes text left to right, one token at a time, each based on what came before. Qwen3 is one.

## Text and tokens

### Token

The unit a model reads and writes. Often a piece of a word, not a full word or a single letter: `"function"` might be `"func"` + `"tion"`.

### BPE

Byte Pair Encoding: the method that splits text into tokens. It merges frequent letter pairs into bigger pieces ("subwords"). One token can mix symbols and letters, like `":"` or `{"`.

### Tokenizer

The tool that turns text into tokens (encode) and back (decode). The **vocabulary** is the full list of tokens the model knows, about 150,000 for Qwen3.

## Generation

### Prompt

The text given to the model. Here: the user's question plus the list of available functions.

### Logits

A score for every token in the vocabulary, given by the model at each step. Higher score = the model thinks that token should come next.

### Masking

Setting a token's score to `-inf` (minus infinity) so it can never be picked. Used to block tokens that would break the format.

### Greedy argmax

Always picking the token with the highest score. Simple and predictable: the same input always gives the same output.

### Constrained decoding

Masking every invalid token before each pick, so the model can only produce output that follows the rules. Full algorithm in [DEV.md](DEV.md#constrained-decoding).

### Grammar

The rules that say which text is valid at each point, for example "after `{` must come `"name"`". Constrained decoding follows it.

## Output

### JSON

A text format for structured data: `{"key": "value"}`. Any programming language can read it, so another program can take the model's output and run the function safely. That is why this project outputs JSON.

### Schema

The description of what valid data looks like: which keys exist and what type each value has. Here, `functions_definition.json` is the schema for each function.

### Hallucination

When a model makes something up. A **hallucinated key** is a field that doesn't exist in the schema, like `"num1"` instead of `"a"`.

## Tool calling

### Function calling

An LLM choosing a function and its arguments instead of answering directly. Also called **tool calling**. See the five steps in [README.md](README.md#how-tool-calling-works).

### Agent

An AI system that uses tools in a loop to complete tasks. Function calling is what makes it possible.
