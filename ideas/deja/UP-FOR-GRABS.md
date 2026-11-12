# Up for grabs: deja

I designed this and never built it. I'm not going to work on it any more, so
if you have the time, take it. It's MIT-licensed: fork it, rename it, ship it,
no need to ask.

## The idea in one line

An MCP server that lets a coding agent look up what other agents already did
in the same repo, on the same error or on the same kind of task, before it
starts guessing.

## Why it might matter

Coding agents hit the same dead ends independently, thousands of times, in the
same repositories. What a session learns about "how you actually navigate
this codebase" is thrown away when it ends. Meanwhile about 6,000 real agent
sessions on real open-source repos sit in a published research dataset that
nobody uses at agent runtime. The corpus already exists. deja would make it
queryable from inside an agent: `find_prior_sessions("paged attention OOM")`
returns condensed accounts of which files earlier agents opened, what failed
and which approach actually landed a commit.

## What exists

The spec only:
[`docs/superpowers/specs/2026-08-30-deja-design.md`](docs/superpowers/specs/2026-08-30-deja-design.md).
It includes a prior-art survey measured in Aug 2026. That survey is why the
broadcast/session-viewer version of this idea was dropped: that space is
crowded. Retrieval at runtime is not.

## What's left

Everything: ingest the dataset, condense sessions into short structured
accounts, index them by repo, error and task, and expose them as MCP tools.
Then measure whether agents with deja solve tasks in fewer steps than without.

## Where to start

Read the spec's prior-art section first, then the corpus section. Build the
condenser and an offline eval before the MCP server. If the condensed
accounts don't make agents measurably faster, the server doesn't matter.
