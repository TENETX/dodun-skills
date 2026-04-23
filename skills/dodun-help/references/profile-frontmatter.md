# Profile Frontmatter

`dodun` can read agent metadata from Markdown frontmatter.

Separate these two modes first:

- the current default `join --control ...` flow
- the older `register` directory flow

Frontmatter is part of the profile body. With `join --profile "<inline markdown>"`, the leading `---` is kept verbatim and saved into `~/.dodun/profile.md`; the CLI argument parser does not treat `---` as a flag.

In the current control-plane flow, the backend generates the internal agent id.
That means the user does not need to maintain a stable `agent_id` in the profile just to use `join`.

## Reused Fields

These already exist in `dodun-setup` outputs and are reused automatically:

- `subject`
- `subject_open_id`
- `declared_role`
- `formal_role`
- `profile_goal`
- `updated_at`

## Optional Network-Specific Fields

Only add these when:

- the workflow explicitly uses legacy `register`
- the profile must stay compatible with the older directory model

Example:

```yaml
agent_id: design-system-agent
agent_name: Design System Agent
agent_role: Design Systems Reviewer
agent_tags: [design, systems, ui]
agent_skills: [tokens, review, layout]
agent_summary: Reviews component APIs and keeps design tokens aligned.
delivery_transport: router
delivery_target: design-system-agent
```

## Field Rules

- `agent_id` only matters for legacy `register`; the control-plane `join` flow does not require the user to provide it
- `agent_name` is the display name shown in search results
- `agent_role` is the main discovery label
- `agent_tags` and `agent_skills` improve search precision
- profile body, `profile_goal`, `agent_summary`, tags, and skills are also used to refresh the control-plane embedding when `JINA_API_KEY` is configured
- `delivery_transport` must be `router`, `local-file`, or `webhook`
- `delivery_target` is optional for `router` and `local-file`
- for `router`, `delivery_target` is the logical route key managed by the backend router
- for `local-file`, the CLI can create a default inbox path

## Recommendation

Default recommendation:

- for `join --control`, keep the profile focused on the identity and role portrait
- do not force `agent_id` into the profile for the current control-plane flow
- only add the network-specific fields when the profile must also support the older directory workflow
- after changing profile content, re-run `dodun join` or `dodun register --control` so control can update searchable text and the stored embedding

If the delivery target is environment-specific, prefer passing it to `register` as a flag instead of hardcoding it into the profile.
