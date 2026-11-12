# SLM project ideas + literature map
Researched 2026-08-25. A literature map of small language models (SLMs) for agentic work, and four ranked project ideas in the open gaps.

## 1. Literature map (what already exists)

### The thesis paper
- NVIDIA, "Small Language Models are the Future of Agentic AI" (arXiv:2506.02153)
  - 6-step LLM-to-SLM conversion algorithm: trace collection -> PII scrub -> task clustering
    -> SLM selection -> LoRA/QLoRA fine-tune -> iterate.
  - Case study replacement estimates: MetaGPT 60%, Open Operator 40%, Cradle 70%.
  - Stated barriers: infra inertia, benchmark misalignment, marketing deficit.
  - Stated open items: cost dynamics as infra matures; agentic-specific SLM benchmarks;
    EMPIRICAL VALIDATION OF THE CONVERSION ALGORITHM (no artifact released).

### Distillation / agentic SLM
- Distilling LLM Agent into Small Models with Retrieval and Code Tools (NeurIPS 2025, arXiv:2505.17612)
  - repo: github.com/Nardien/agent-distillation
- Student-Centered Distillation / SCoRe (arXiv:2509.14257) - 7B student closes gap w/ 72B teacher
- SLMs for Efficient Agentic Tool Calling (arXiv:2512.15943) - 77.55% ToolBench pass rate
- Search Do Not Guess (arXiv:2604.04651); Privileged Info Distillation (arXiv:2602.04942)
- SLMs for Agentic Systems survey (arXiv:2510.03847)
- AgentFloor: how far up the tool-use ladder can small open-weight models go (arXiv:2605.00334)

### Routing / cascade  [CROWDED - avoid]
- Cluster Route Escalate (arXiv:2606.27457) - 97-99% of strongest model accuracy, lower TPOT
- UCCI calibrated uncertainty cascade routing (arXiv:2605.18796)
- Switchcraft: router for agentic tool calling (arXiv:2605.07112)
- ReLope multimodal routing (arXiv:2603.24787); Privacy-preserving routing (arXiv:2604.15728)
- Second Guess: abstention + answer stability in SLMs (arXiv:2605.25394)

### Constrained decoding + SLMs  [THE OPEN SEAM]
- The Constraint Tax (arXiv:2605.26128): on Qwen2.5-0.5B/1.5B + SmolLM2-1.7B, hard schema
  decoding takes validity 61.5% -> 100.0% but accuracy 19.7% -> 11.0%. 3.6x-8.2x latency overhead.
- When Correct Isn't Usable (arXiv:2605.02363): 7-9B models, naive prompting = 85% task accuracy
  but 0% OUTPUT accuracy (correct AND parsable).
- Structure Snowballing / alignment tax of constrained decoding (arXiv:2604.06066)
- JSONSchemaBench (arXiv:2501.10868)
- All of these MEASURE the tax. None MITIGATE it at the grammar-compiler level.

### Speculative decoding  [crowded but SLM angle partly open]
- Future Validity is the Missing Statistic / Phi-estimation for grammar-faithful spec decoding (arXiv:2605.07698)
- Constrained Decoding with Speculative Lookaheads (arXiv:2412.10418)
- PyramidSD, DSpark (arXiv:2607.05147), BanditSpec (arXiv:2505.15141),
  Learning to Draft (arXiv:2603.01639), 3-Model Spec Decoding (arXiv:2510.12966)

### Serving infra for SLM fleets
- InfiniLoRA disaggregated multi-LoRA serving (arXiv:2604.07173)
- ForkKV: copy-on-write disaggregated KV cache for multi-LoRA agents (arXiv:2604.06370)
- Kairos multi-agent serving (arXiv:2508.06948)

### Edge / on-device  [CROWDED - avoid]
- Cloud to Edge: benchmarking LLM inference on hardware-accelerated SBCs (arXiv:2604.24785)
- Edge Deployment of SLMs (arXiv:2511.22334); SLM Survey Measurements Insights (arXiv:2409.15790)

### Cost / TCO
- Only blog-tier calculators exist. Consensus break-even ~100M-500M tokens/mo, or >2M tokens/day.
  All of it is PER-TOKEN. Nobody composes in retry rate, schema-failure rate, or escalation rate.


## 2. Ranked project ideas

### TIER 1

#### Idea A - Constraint-tax-aware grammar compiler ("schema linting for small models")
WHAT: given a JSON Schema, emit semantically-equivalent grammar variants (reasoning-field
placement, field ordering, enum vs free string, flat vs nested, free-then-constrain / lazy
masking), measure validity x accuracy x latency for each across an SLM set, and ship the
minimal-tax variant. Library + public leaderboard.
NOVELTY: strong. Three 2026 papers measure the constraint tax; none reduce it at compile time.
FIT: extends an existing grammar compiler (github.com/pjdurden/genui-grammar) rather than starting from scratch.
COST: 4-6 weeks. Needs a rented GPU, but 0.5B-1.7B eval is cheap.
UPSTREAM PATH: llguidance / outlines-core / XGrammar.

#### Idea B - Cost-per-usable-output (effective cost model for SLM fleets)
WHAT: harness that measures true cost = raw token cost / (P(schema valid) x P(correct)), plus
measured escalation rate under a router. Publish the composed curve and the real crossover point.
NOVELTY: strong, and it produces a FINDING, not just a tool: "SLMs are Nx cheaper on paper,
only Mx cheaper once you pay for retries and escalation."
FIT: extends client-side cost-reduction work (github.com/pjdurden/cache-aware-request-planning) to the SLM case, and can reuse kv-exec-bench's execution-validity checks.
CONTRARIAN VALUE: a credible "the SLM cost case is overstated" result is more useful than another "SLMs are great" post.
COST: 3-4 weeks, and it SHARES INFRASTRUCTURE WITH IDEA A (both need: run SLMs over a task
suite, measure validity + correctness). Build the harness once, get two artifacts.

### TIER 2

#### Idea C - Open reproduction of NVIDIA's LLM-to-SLM conversion algorithm
WHAT: `slm-convert` - trace instrumentation, PII scrub, task clustering, SLM candidate
selection, LoRA distillation, replacement-rate report. Reproduce the MetaGPT 60% / Cradle 70% numbers.
NOVELTY: medium on ideas, HIGH on artifact - the reference implementation of a well-cited
paper whose authors explicitly asked for it, with a listed contact (agents@nvidia.com).
COST: highest. Needs real fine-tuning infra.

#### Idea D - Grammar-aware speculative decoding with SLM drafters
Draft proposals get killed by the GRAMMAR, not the target model. Partly covered by
arXiv:2605.07698 and arXiv:2412.10418. Natural upstream home is vLLM; a smaller engine (e.g. github.com/pjdurden/nanoserve) is an easier place to prototype it.

### SKIP (too crowded)
- Routing / cascade / escalation - 5+ 2026 papers, saturated
- SLM abstention / uncertainty - Second Guess + UCCI cover it
- Edge / NPU SLM benchmarking - Edge-LLM-Bench just did it
- Yet another SLM survey

## 3. Recommendation
Build A, then publish B off the same harness. Both need the same machinery (run SLMs over a task suite and measure validity and correctness), so one harness yields two artifacts. Upstream homes are llguidance and outlines-core.
