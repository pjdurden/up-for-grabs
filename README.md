# Up for grabs

Projects and ideas I started or thought through, then stopped working on. None of them is maintained. Everything is MIT-licensed: fork it, rename it, ship it, no need to ask.

Each entry has an `UP-FOR-GRABS.md` that says what the idea is, what exists, what's left and where to start.

| Name | What it is | Kind | Handed over |
|---|---|---|---|
| [gh-agentify](https://github.com/pjdurden/gh-agentify/tree/up-for-grabs) | gh CLI extension that scores a repo's readiness for AI coding agents (doctor works) and was meant to wire up Spec Kit, AI review, gh-aw, zizmor and rulesets in one command | code | 2026-11-12 |
| [deja](ideas/deja) | An MCP server that lets a coding agent look up what other agents already did in the same repo, on the same error or on the same kind of task, before it starts guessing. | idea (spec) | 2026-11-12 |
| [semmap](https://github.com/pjdurden/semmap/tree/up-for-grabs) | Auto-maintained per-directory map files (`_map.md`) that tell a coding agent where things live in a repo, regenerated only for the directories a merged PR touched, so they never rot the way hand-writt | code | 2026-11-12 |
| [pickui](https://github.com/pjdurden/pickui/tree/up-for-grabs) | Personalized UI with no generated UI at all: every button, panel and layout has a few hand-built variants, one batched call to a fast classification model picks which variant this shopper sees, and pl | code | 2026-11-12 |
| [why-engine](https://github.com/pjdurden/why-engine/tree/up-for-grabs) | An autonomous pipeline that researches, scripts, illustrates, narrates and assembles one 15 to 20 minute "why is America built this way" explainer video per day, with every claim grounded in primary s | code | 2026-11-12 |
| [slm-agent-ideas](ideas/slm-agent-ideas) | Four ranked open-source projects in the gaps of small-language-model research for agents. The best is a grammar compiler that cuts the "constraint tax" small models pay under structured output. | idea (spec) | 2026-11-12 |
| [kobon-duel](https://github.com/pjdurden/kobon-duel) | Two adversarial Claude sessions arguing about the Kobon triangle problem | code | 2026-11-12 |
| [planner-bench](https://github.com/pjdurden/planner-bench) | Benchmark for LLM-serving autoscaling forecasters, scored on GPU-hours and SLO violations rather than forecast error. Tests TimesFM 3.0, Chronos and NVIDIA Dynamo's four shipped predictors on a real Azure trace. | code | 2026-11-12 |
| [kv-canary](https://github.com/pjdurden/kv-canary) | A canary for silent KV-cache-compression failures: functional accuracy (code/tool-calls) vs perplexity under quantization & token eviction, scored by a Silent Degradation Score. | code | 2026-11-12 |
| [kv-exec-bench](https://github.com/pjdurden/kv-exec-bench) | Executable-correctness benchmark for KV-cache compression: code pass@1 and tool/JSON-schema validity instead of perplexity | code | 2026-11-12 |
| [ai-accelerator-radar](https://github.com/pjdurden/ai-accelerator-radar) | Investment-committee dashboard on AI companies funded by Y Combinator and peer accelerators: category mix, outcome track record, crowding vs breakout | code | 2026-11-12 |
| [airlock](https://github.com/pjdurden/airlock) | Scheduled Claude Code agents that do real work unattended, and cannot reach the outside world until a human opens the door. | code | 2026-11-12 |
| [churn](https://github.com/pjdurden/churn) | Deterministic, zero-GPU simulator of elastic-membership DiLoCo training + a straggler-aware membership policy that beats the wait-for-everyone baseline (~4.6x in-sim, validated on real torch/gloo). | code | 2026-11-12 |
| [cache-aware-request-planning](https://github.com/pjdurden/cache-aware-request-planning) | Cache-aware client-side request planning for black-box LLM APIs: lossless billed-cost reduction by aligning requests to the provider prompt cache. Research artifact + paper draft. | code | 2026-11-12 |
| [genui-grammar](https://github.com/pjdurden/genui-grammar) | Constrained-decoding grammar compiler for generative UI: compiles a component schema into a grammar so an LLM can only emit valid UI trees | code | 2026-11-12 |
| [decided-ui](https://github.com/pjdurden/decided-ui) | The model chooses among interfaces you wrote. Decided UI, not generative UI. | code | 2026-11-12 |
