# deja: retrieval over prior coding-agent sessions

Status: design approved, not implemented
Date: 2026-08-30
Author: pjdurden

## Summary

deja is an MCP server that lets a coding agent consult what other agents
already did in the same codebase, on the same error, or on the same kind of
task, before it starts guessing.

A developer sits down to fix a bug in vLLM. Before the agent begins, it calls
`find_prior_sessions("paged attention OOM")`. It gets back three condensed
accounts of other agents that worked in that repo: which files they opened
first, what they tried, what failed, and which approach actually landed a
commit. That goes into context. The agent skips wrong turns somebody else
already took.

The corpus is not ours. It already exists.

## Problem

Coding agents rediscover the same dead ends independently, thousands of times,
in the same repositories. The knowledge of "how do you actually navigate this
codebase" is generated in every session and destroyed at the end of it.

Meanwhile roughly 6,000 real agent sessions on real open-source repos sit in a
published research dataset, shaped for offline analysis by researchers, and are
consumed by nobody at agent runtime.

## Prior art, and why this shape

The obvious versions of this idea are taken, and the evidence is worth
recording so we do not re-derive it later.

Session viewers and sharing tools (measured 2026-08-29):

| Repo | Stars | Last push |
| --- | --- | --- |
| jhlee0409/claude-code-history-viewer | 2,117 | active |
| simonw/claude-code-transcripts | 1,674 | Feb 2026 |
| d-kimuson/claude-code-viewer | 1,277 | active |
| daaain/claude-code-log | 1,202 | active |
| yoavf/ai-sessions | 10 | Dec 2025 |
| terryso/cc-live | 6 | Apr 2026 |
| moredip/session-share | 2 | Feb 2026 |

The distribution is bimodal and the split is not random. Every project above
1,000 stars is a personal-utility tool: read my own history, publish my own
session. Every project in the single digits is a social or broadcast tool:
watch someone else, share to an audience, live viewing. Four independent teams
built the spectating version and all four failed. The demand is for reading
your own transcripts, not for watching strangers.

Public corpora:

- Trace Commons (HuggingFace `trace-commons/agent-traces`), CC-BY-4.0. Correct
  design (public repos only, local anonymization, contributor review) but 33
  session files, created 2026-06-11, untouched since 2026-06-18. Voluntary
  donation does not fill a corpus.
- SWE-chat (HuggingFace `SALT-NLP/SWE-chat`), ODC-BY, gated with automatic
  approval. About 6,000 sessions, 63k user prompts, 355k tool calls, 2.7M
  logged events, roughly 200 repos. Harvested from Entire.io checkpoint
  branches in public GitHub repos, not donated. Ships prebuilt tables:
  `sessions`, `conversations`, `checkpoints`, `commits`, `repositories`,
  `session_logs`, plus ~5,850 raw transcripts.

Every agent-memory MCP server found is scoped to the user's own history or
their team's. None retrieve from a public cross-user corpus. That is the gap
deja occupies.

Two properties of SWE-chat make it the right substrate. Transcripts are Claude
Code JSONL, byte-identical in shape to what Claude Code writes locally, so one
parser serves both the corpus and any future local ingest. And each checkpoint
links to a commit with line-level authorship, which gives a ground-truth
did-this-work signal for free.

## Non-goals

- Not a session viewer. That market is saturated and won.
- Not a broadcast or spectating product. Four corpses.
- Not a new corpus. We consume existing ones; filling a corpus is somebody
  else's unsolved problem and we should not inherit it.
- Not agent memory. deja is cross-user and read-only; it does not store the
  user's own sessions.

## Architecture

Hosted retrieval (chosen over local-first): the operator authenticates to
HuggingFace once and serves derived results, so users never hit the dataset
gate. That gate would otherwise be an install-killing first step.

Five components, two of them always-on.

### 1. Ingest (offline batch)

Runs on a workstation or in CI, on demand, roughly monthly. Authenticates to
HF, pulls SWE-chat parquet tables and transcripts, segments each transcript
into episodes, distills episode cards, embeds them, writes to the index.

Slow and unglamorous by design. Nobody watches it run. This is also where
additional sources (Trace Commons, direct GitHub harvesting of Entire.io
checkpoint branches) are added later without touching anything downstream.

All of the intellectually interesting code lives here, in the open-source repo,
readable and runnable by anyone.

### 2. Index store (Cloudflare)

- Vectorize: episode-card embeddings, with repo as a filter field so queries
  are scoped before they are ranked.
- D1: filter and ranking metadata (repo, language, outcome, landed-a-commit,
  model, timestamps).
- R2: full episode bodies, fetched only for results actually returned.

One vendor, three primitives, nothing to patch or restart.

### 3. Retrieval API (single Cloudflare Worker)

`POST /search` with `{repo, query, k, filters}`, returns ranked episode cards.

Pipeline: embed query on Workers AI (`bge-base-en-v1.5`, avoiding a second
vendor and an unpredictable per-call bill) -> Vectorize search filtered by repo
-> lexical match on files, symbols, error strings -> rank fusion -> hydrate
bodies from R2 -> condense -> return.

### 4. MCP server (thin client)

Published to npm as `deja-mcp`, run via `npx`. Holds no data and no logic
beyond shaping tool calls into HTTP requests.

### 5. Cost containment (v1, not bolted on later)

Free API key via GitHub OAuth, per-key daily quota, aggressive caching of
repeated `(repo, query)` pairs. The failure mode of a hosted architecture is a
surprise bill, so the throttle ships with v1.

## Sizing

Measured across 60 real Claude Code transcripts (244MB local corpus):

```
human turns / session     median 2      mean 6.3    p90 13     max 62
tool calls / session      median 11     mean 42     p90 132    max 303
session size              median 254KB  mean 645KB  p90 1.7MB  max 5.0MB
episode size              median 39KB   mean 102KB  p90 229KB  max 1.6MB
tool calls / episode      median 3      mean 6.7    p90 17     max 102
```

6,000 sessions at ~6.3 human turns each is about 38k episodes, and after
quality filtering closer to 20k. At 768 dimensions the whole index sits inside
Cloudflare free tiers. Steady-state cost is therefore not a design constraint.
The throttle in component 5 exists for abuse and runaway traffic, which is a
different problem from index size.

## Segmentation and the episode card

### Boundaries

An episode runs from one human turn to the next. The boundary is cleanly
detectable: user entries come in two kinds, and only one is a real human. A
genuine turn carries a `text` content block. The other ~65 of 66 user entries
in a typical session are `tool_result` payloads fed back to the model.

Dropped line types: `queue-operation`, `attachment`, `last-prompt`,
`atis-latch`, `file-history-snapshot`.

### Why cards, not episodes

A median episode is 39KB (roughly 10k tokens) and p90 is 229KB. It cannot be
returned to a querying agent and cannot be usefully embedded. Nearly all of
that mass is `tool_result` bodies: file contents, bash output, grep dumps,
which are simultaneously the largest and least useful part for retrieval.

The indexed unit is a card of roughly 1-3KB distilled from the episode:

- `intent`: the human's ask, verbatim, truncated.
- `trajectory`: ordered action sequence, names and arguments only, no payloads.
  For example `Read(csrc/attention/paged_attn.cu)` -> `Bash(pytest -k paged)`
  -> `Edit(vllm/core/block_manager.py)`.
- `files`: files touched. The strongest retrieval key.
- `errors`: extracted traceback lines and error strings. The second strongest,
  and arguably the highest-value single field in the system.
- `outcome`: landed a commit / tests passed / abandoned, from SWE-chat's
  `checkpoints` and `commits` tables.

Full episodes are stored in R2 addressable by id. The card is what gets
searched; the episode is what `get_episode` returns.

### Quality filter

With a median of 3 tool calls per episode, a large fraction are junk ("yes",
"continue", "thanks"). Drop episodes with fewer than 2 tool calls or no file
touched. This likely halves the index and raises precision more than any
ranking change would. Rank landed-a-commit episodes above ones that went
nowhere, using ground truth rather than a guessed quality score.

Card fields are hard-capped. The 1.6MB max episode is real and will otherwise
break ingest.

### Known risk

Distillation is lossy. If the card drops the thing that made the episode
useful, retrieval returns plausible-looking junk. This is precisely what the
held-out eval exists to catch, which is why the eval is built before ingest.

## Retrieval

Two different query types need two different indexes, fused rather than
blended:

1. "I am trying to do X": semantic, wants embedding similarity over intent and
   trajectory.
2. "I hit this exact traceback" or "I am editing this exact file": lexical,
   wants exact match over files, symbols, error strings.

Embedding similarity is poor at the second and exact match is poor at the
first. Exact match on an error string is dramatically more precise than any
embedding, and "another agent hit this identical error in this identical repo"
is the highest-value result deja can produce.

### Coverage tiers

SWE-chat spans roughly 200 repos. Most users will work on a repo with zero
sessions in the corpus. If repo-anchored retrieval is the only mode, deja is
empty for nearly everyone and gets uninstalled on day one.

Retrieval degrades in labeled tiers:

1. Same repo. Highest value, rare.
2. Same error, any repo. Often just as useful, because errors are more portable
   than codebases.
3. Same task shape, similar stack. Weakest but not nothing.

The card header states its tier so the model can weigh it. `repo_coverage`
returning zero must say so honestly rather than silently falling back, so the
agent does not over-trust a tier-3 result.

## MCP surface

Three tools. Every additional tool degrades selection quality for the whole
session.

```
find_prior_sessions(task, error?, files?, repo?, k=3)
get_episode(episode_id, part?)
repo_coverage(repo?)
```

`repo` defaults to the git remote auto-detected in the client's cwd, so the
model cannot supply it wrongly. Same for `files` when there is an obvious
current file.

Results are compact markdown, not JSON. Cards cost fewer tokens as prose and
models read them better:

```
[ep_8f21] vllm-project/vllm - landed (commit a3f9c21) - tier 1 (same repo)
intent: "paged attention kernel OOMs on long sequences"
path:   Read paged_attn.cu -> Grep "max_seq_len" -> Edit block_manager.py
        -> Bash "pytest -k paged" (failed) -> Edit paged_attn.cu -> passed
errors: CUDA error: out of memory (block_manager.py:214)
files:  csrc/attention/paged_attn.cu, vllm/core/block_manager.py
```

Three cards is roughly 1-1.5k tokens. That is the budget ceiling: `k=3`
default, hard cap 5.

### Firing policy

Models systematically under-call unfamiliar tools, so a tool nobody invokes is
a dead project. But auto-injecting retrieved context on every prompt burns
tokens constantly and poisons the session on a bad match.

A `SessionStart` hook performs one cheap coverage lookup and injects a single
line: "deja holds 47 prior sessions for this repo; call find_prior_sessions
before non-trivial work." One line of context; the model keeps control.

That line is also the entire discovery mechanism. It tells the model both that
the tool exists and that it is relevant here, which a static tool description
can never do.

## Privacy and abuse

A hosted architecture means the query, containing the user's task description
and possibly error text from a private codebase, leaves their machine.

- A client-side secret scrubber runs over the query before it is sent:
  known-pattern detectors (provider API keys, JWTs, PEM blocks, DB connection
  strings), high-entropy string detection, path anonymization. Seed the pattern
  corpus from cc-live's `BASE_SENSITIVE_PATTERNS` (MIT, attributed), then do
  the three things it did not: run at capture rather than display, add entropy
  detection, and emit a redaction manifest.
- No query logging by default, stated in the README. Opt-in telemetry only.
- An offline mode flag for users who will not send anything, falling back to a
  locally built index. Post-v1: it duplicates the index build on the client and
  is not required for launch.

## Eval

Built first, before ingest. It costs nothing to run and it is the only thing
that can tell us the project works.

### Tier 1: held-out continuation (offline, no model calls)

Remove episode E from the corpus. Build a query from its prefix only (human
intent, plus first error if present). Query with E's entire session excluded.
Score returned episodes against what E actually did:

- File hit@k: did any returned episode touch a file E ended up touching?
- Error hit@k: did retrieval surface an episode with the same error string?
- Path overlap: Jaccard between returned file sets and E's file set.

Deterministic, free, runs in CI, and doubles as the regression suite for every
future ranking change.

### Baselines

A hit@3 of 0.4 means nothing alone. Five arms, identical queries:

1. Random episodes from the corpus
2. Random episodes from the same repo
3. BM25 lexical only
4. Vector only
5. Hybrid fusion (the system)

Arm 2 is the honest killer. In a 200-repo corpus, returning any episode from
the same repo scores well on file overlap because repos have hot files. If
hybrid retrieval cannot clearly beat random-same-repo, the embedding work is
decoration, and we need to know that in week two rather than month three.

### Splits

Episodes within a session are near-duplicates. Split by session for the
same-repo tier. Run a second split with entire repos held out, which is the
only honest measure of tier-3 cross-repo generalization. Report both; they will
differ substantially.

### Negative controls

Query with nonsense, and with tasks about repos absent from the corpus. The
system must return nothing rather than confidently returning its best bad
match. A retrieval tool that always returns something is worse than useless
inside an agent loop, because the agent will believe it.

### Tier 2: end-to-end (gated behind tier 1)

Run an agent on real tasks with retrieval on and off; measure resolve rate and
turns-to-solution. This is the headline number and the basis for any writeup,
but it costs real money and is meaningless if tier 1 says retrieval finds
nothing.

## Testing

Distinct from eval:

- Golden-file tests for segmentation against fixture transcripts.
- One test per scrubber pattern.
- Snapshot tests on card distillation.
- Contract tests on the Worker API.
- Schema tests on the MCP tool definitions.

## Build order

1. Eval harness and fixtures
2. Ingest and segmentation
3. Index build
4. Retrieval API
5. MCP client
6. Auth, quota, and query caching
7. SessionStart hook

This yields a defensible number before anything hosted exists, which puts the
abandonment risk on the cheap end of the project.

## Open questions

- HF gate terms for `SALT-NLP/SWE-chat` sit on top of the ODC-BY license and
  have not been read. Serving derived content must be confirmed permissible
  before launch. Not a design blocker.
- SWE-chat's HF snapshot was last modified 2026-04-29 though the paper calls it
  a living dataset. Freshness needs checking.
- Entire.io checkpoint branches live in public GitHub repos, so direct
  harvesting may bypass the gate entirely and solve freshness. Unverified.
- Embedding model choice (`bge-base-en-v1.5`) is a default, not a measured
  decision. The tier-1 eval should settle it.

## Risks

- Distillation loses the useful signal and retrieval returns confident junk.
  Mitigated by building the eval first.
- Coverage is too narrow (200 repos) for tier-1 matches to be common, and tiers
  2 and 3 prove not useful enough. This is the central product risk and the
  eval measures it directly.
- Hybrid retrieval fails to beat random-same-repo, meaning the system adds
  nothing over a trivial baseline.
- Hosted architecture carries ongoing cost and uptime obligations that outlast
  motivation.

## Attribution

- SWE-chat: SALT-NLP, ODC-BY. Attribution required.
- Trace Commons: CC-BY-4.0. Attribution required.
- Secret pattern corpus seeded from terryso/cc-live, MIT. Attribution in
  README.
