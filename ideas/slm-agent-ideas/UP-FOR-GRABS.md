# Up for grabs: SLM agent project ideas

I researched these and never built any of them. I'm not going to work on
them, so if you have the time, take one. It's MIT-licensed: use it, rename it,
ship it, no need to ask.

## The idea in one line

Four ranked open-source projects in the gaps of small-language-model research for agents. The best is a grammar compiler that cuts the "constraint tax" small models pay under structured output.

## Why it might matter

NVIDIA's "Small Language Models are the Future of Agentic AI" (arXiv:2506.02153)
argues that most agent calls could run on SLMs, and asks for empirical work
that doesn't exist yet. Three 2026 papers measure how much accuracy small
models lose when forced into a JSON schema, but none reduces that loss at
compile time. The cost case for SLMs is also usually stated per token, never
per usable output.

## What exists

Research only: [`SPEC.md`](SPEC.md) has the literature map (which areas are
crowded and which are open) and the ranked ideas:
- **A.** Constraint-tax-aware grammar compiler ("schema linting for small models")
- **B.** Cost-per-usable-output harness for SLM fleets
- **C.** Open reproduction of NVIDIA's LLM-to-SLM conversion algorithm
- **D.** Grammar-aware speculative decoding with SLM drafters

## What's left

All of it. A and B share one harness: run SLMs over a task suite and measure
schema validity and correctness. Build that harness once and you get two
results.

## Where to start

Idea A. Take a handful of real JSON Schemas, generate equivalent grammar
variants (field order, where the reasoning field goes, enum vs string, flat vs
nested), and measure validity, accuracy and latency on 0.5B to 2B models. A
rented GPU for a few days is enough.

## Notes

The literature map is from Aug 2026. Re-check the "open seam" sections before
you build, because this area moves fast.
