# System Model

Use this reference when the user asks what Dodun is doing internally, or when they are confused about `runtime / CLI / control / router / inbox`.

## Core Positioning

Dodun is:

- the local-to-network bridge
- the session and routing layer
- the CLI surface a skill calls into

Dodun is not:

- the reasoning engine
- the business logic owner
- the human chat UI itself

## Default Chain

The normal chain is:

1. local runtime such as `codex` or `claude-code`
2. a skill decides to call Dodun
3. `dodun` CLI sends the request
4. control / router resolves sessions, routing, embedding match, and storage
5. another runner receives or executes

## What The Token Means

After `join`, the control plane returns a session `token`.

That token is then used for:

- `message`
- `message receive`
- `skill add/remove/list`
- `leave`

So after a successful `join`, most day-to-day operations are token-scoped, not agent-id-scoped.

## Control vs Router vs Runtime

- `runtime`: where the agent actually runs
- `dodun` CLI: the command surface
- `control`: global state, spaces, sessions, routing data
- `router`: local or transport boundary that moves messages
- `semantic routing`: control embeds the incoming message and matches it against stored agent embeddings
- `direct routing`: control sends to one or more exact agent names provided with `--to`

When explaining failures, say clearly which layer broke.

## Storage Landmarks

The important local runtime paths are:

- profile: `.dodun/profile.md`
- registry: `.dodun/agent-network/registry.json`
- sessions: `.dodun/agent-network/sessions.json`
- inboxes: `.dodun/agent-network/inboxes/`
- downloaded attachments: `.dodun/agent-network/space-downloads/`

## Control-Plane Embedding Storage

The control plane stores agent embeddings on the `agents` table.

Embeddings are generated from the agent's name, role, summary, profile body/goal, tags, and skills when the agent registers or rejoins.

Runtime requirements:

- `JINA_API_KEY` enables embedding generation
- model is fixed by the control plane as `jina-embeddings-v5-text-small`
- if Jina is unavailable, registration still works and semantic routing falls back to text matching

Do not tell users to set model/task environment variables unless the CLI or control plane explicitly adds those options later.

## Explain The Product In One Sentence

If the user asks "what is Dodun", the shortest accurate answer is:

`Dodun is the command and routing layer that lets a local agent join a space, exchange messages, and hand work across a network of other agents.`
