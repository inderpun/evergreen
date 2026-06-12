---
last_verified: 2026-06-12
gardener_sources:
  - https://huggingface.co/docs/tokenizers/index
  - https://github.com/openai/tiktoken
  - https://github.com/google/sentencepiece
---

# Tokens & Tokenization

## What & why

A language model cannot read text. It reads integers. **Tokenization** is the process
that converts a string of characters into a sequence of integers (tokens), and back.
Every LLM call starts with it and your bill is calculated in it.

Why not split on spaces (words)? Words give you a vocabulary of millions with long
tails of rare forms — "running", "runner", "ran" look unrelated. Why not split on
characters? You get a tiny vocabulary but sequences become very long, and the model
must spend capacity learning basic spelling from scratch.

**Subword tokenization** (the dominant approach) is the compromise: common words stay
whole, rare words are split into meaningful pieces. "tokenization" might encode as
`["token", "ization"]`. This balances vocabulary size (~30–100K tokens) against
sequence length, and lets the model share learned representations across morphological
variants.

## How it actually works

The dominant algorithm is **Byte-Pair Encoding (BPE)**, originally a compression
algorithm repurposed for NLP:

1. Start with a vocabulary of individual bytes (256 entries — handles any Unicode).
2. Count the most frequent adjacent pair in the training corpus.
3. Merge that pair into a new token. Add it to the vocabulary.
4. Repeat until the vocabulary reaches the target size.

The result is a vocabulary that reflects corpus statistics: common English subwords
get dedicated tokens, while rare or foreign-language subwords fragment into bytes.

**SentencePiece** is a related approach used by many open-weight models. It operates
on raw Unicode rather than bytes, treats the input as a continuous character stream
(no pre-tokenization on spaces), and is language-agnostic by design.

One critical detail: tokenizers have a **normalization** step before encoding —
lowercasing, Unicode normalization, whitespace handling — and two tokenizers that use
the same BPE algorithm but different normalization will produce different token
sequences from the same string.

## Practical implications for builders

**Token counts = costs.** Every provider bills per input and output token. The
cost of a prompt is not proportional to its character count; it is proportional to
how the model's specific tokenizer encodes it. Code and structured formats (JSON,
XML, Markdown) are often more token-dense than prose because punctuation and
whitespace tokenize poorly.

| Input type | Rough observation |
|---|---|
| Dense English prose | ~1 token per ~4 characters |
| Code (Python, JSON) | Higher token-per-character ratio due to indentation, brackets |
| Non-Latin scripts | Often 2–4× the English token rate for equivalent meaning |
| Whitespace-heavy formats | Every repeated space/indent is a token |

**Multilingual quirks.** BPE training corpora are English-heavy. A concept that takes
5 tokens in English may take 15–25 tokens in Thai, Arabic, or Chinese, because those
scripts fragment more aggressively. This has three practical consequences: (1) costs
more, (2) leaves less room in a context window for non-English content, (3) models
may reason less reliably in languages that were underrepresented during training.

**Tokenizer mismatch pitfalls.** When you switch models — say, from one family to
another — the tokenizer changes. A prompt that fit in the context window of one model
may exceed the limit of another even if both are described as "128K context." Always
measure token counts with the tokenizer belonging to the model you are actually
calling. Counting with the wrong tokenizer is a silent bug that shows up as
unexplained context-limit errors in production.

**Counting tokens before calling the API** is inexpensive and prevents surprises.
Most inference libraries expose a `count_tokens` or `encode` method. Use it on large
prompts during development, and in production for any prompt that could grow
unboundedly (e.g., when including retrieved documents).

## Interview questions

After this page you should be able to answer:

1. Why did subword tokenization win over word-level and character-level approaches? What does each trade away?
2. Walk through the BPE algorithm. What determines which pairs get merged first?
3. A customer reports that their Spanish-language prompts cost 3× more than equivalent English ones. What is the most likely explanation, and is there a fix?
4. You switch from one LLM provider to another with the same advertised context length. Some prompts start failing with "context too long". Why?
5. Your RAG pipeline inserts retrieved documents into a prompt template. What can go wrong with token counting here, and how do you defend against it?

---

*Last verified 2026-06-12 · maintained by the Gardener*
