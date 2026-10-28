# What is Jev?

## How a normal model call works

Every model call has two stages:

| Stage | What happens | Bound by |
|---|---|---|
| **1. Prefill** | The model reads the whole prompt in **one forward pass**. | Compute |
| **2. Decode** | The model writes the answer **one token at a time**, feeding each new token back in. | Memory |

Decode in practice, for a 500-token prompt:

```
pass 1:  500 tokens      -> token 1
pass 2:  500 + 1 tokens  -> token 2
pass 3:  500 + 2 tokens  -> token 3
...      until a stop token is produced
```

## What Jev does

Jev **skips the decode stage entirely** and answers in a single prefill pass.

You state up front what you want back, as a set of options or a yes/no, and Jev returns exactly one of them.

> **One forward pass → no decode → predictable output**

## What you can ask it

| API | Type | Example |
|---|---|---|
| `choice` | Nominal | north / east / west / south |
| `score` | Ordinal | low / medium / high |
| `bool` | Binary | true / false |

## Packing it into one sequence

- A decoder-only model generates its response token by token.
- Jev uses an **encoder model** such as BERT instead.
- Attention gives a lot of weight to the first tokens, so the **question goes before the context**, not after it.
- Each new question means feeding the whole sequence in again:

```
question + [MASK] + option A + [MASK] + option B + ... + context
```

## Glossary

| Term | Meaning |
|---|---|
| **Pydantic object** | The output schema for an LLM call. |
| **Logits** | The raw, unnormalized scores from the model's final layer, before softmax. |
| **Vocabulary distribution** | Each prediction is a probability over every token in the vocabulary. |
| **Stop sequence** | If the top-scoring token is a stop sequence, generation ends; otherwise it continues. |
| **KV cache** | The saved keys and values from earlier tokens. They are reused so follow-up questions on the same context don't recompute it. |
