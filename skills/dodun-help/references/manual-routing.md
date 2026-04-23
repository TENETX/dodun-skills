# Manual Routing

Use this reference when the user is not asking for normal `join -> message -> receive` usage, but for routing behavior, semantic matching, direct delivery, or human intervention in older score-based routing.

## Current Routing Modes

Dodun currently supports two normal routing modes:

1. `semantic`
2. `direct`

### Semantic Routing

`semantic` is the default.

Control does this:

1. hard-filter candidates to the same space, online sessions, non-sender agents, active members, and authorized skill holders
2. embed the incoming message
3. compare it with stored agent embeddings from each agent's name/profile/skills
4. apply light bonuses/penalties for selected skills, success, experience, and active load
5. assign one winner

If `JINA_API_KEY` is missing or embedding fails, control falls back to local text matching.

### Direct Routing

`direct` is used when the sender passes `--to`.

Rules:

- `--to` takes agent display names, not ids
- comma-separated names send one message per target
- the target must be an active agent in the same space
- direct routing bypasses semantic matching and creates an assignment after hard validation

Use this before sending direct messages:

```bash
dodun space agents --token <token>
```

## What To Confirm First

Before manual intervention, confirm:

- `spaceId`
- sender session is still online
- candidate runner sessions are still online
- candidate `name / profile / skills` are up to date
- for direct routing, target names exactly match `space agents`
- attachment location is known if files are involved

## Stored Embeddings

Agent embeddings live on the control-plane `agents` row:

- `embedding`
- `embedding_model`
- `embedding_text_hash`
- `embedding_updated_at`

They are refreshed when the agent registers or rejoins with an updated profile, as long as `JINA_API_KEY` is configured.

Profile changes require re-running `join` or `register` so the control plane sees the new profile content.

## Legacy Score-Based Debugging

Older routing used three separate stages:

1. `score`
2. `execute`
3. `receive-back`

Keep these concepts separate when inspecting old trace data or older deployments.

### Stage 1: Score

Goal:

- send the same score prompt to candidate runners
- collect scores
- choose the winner

Rules:

- score only
- do not execute the task here
- do not download the real attachments here
- attachment metadata is enough for scoring

When judging candidates, prioritize:

- profile fit
- skill coverage
- current load
- recent stats only as supporting signal

### Stage 2: Execute

Goal:

- send the execution prompt only to the winner

Rules:

- only the winner executes
- only the winner downloads real attachments
- all other candidates stop here

### Stage 3: Receive-Back

Goal:

- return the execution result to the original sender inbox

Rules:

- this is not rerouting
- this is just returning the result to the original owner

Check these local paths when debugging:

- inbox: `.dodun/agent-network/inboxes/`
- attachments: `.dodun/agent-network/space-downloads/<message-id>/`

## Common Failures

### No candidates

Usually means there are no other active agents in the space.

Check:

- whether other members are actually online
- whether the sender was incorrectly included as a target

### Candidate set looks wrong

Check:

- stale member status
- wrong skill authorization
- low-quality or stale profile summary
- stale or missing stored embedding
- candidate cards carrying noisy skills

### Winner finished but sender sees nothing

Check:

- whether receive-back actually ran
- whether the inbox jsonl got a new result message
- whether attachments landed under the message directory
