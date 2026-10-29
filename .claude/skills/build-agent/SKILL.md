---
name: build-agent
description: Build a new agent on this Trinity instance from a brief — a design from the hosted Architect first when subscribed (ARCHITECT_ADVISOR, or --from), then the create-agent wizard headlessly, one gaps ask + one approval ask, the agent created with all schedules off; --fleet designs a whole fleet into one target view instead
argument-hint: <brief> [--from <advisor>] [--no-deploy] [--fleet]
allowed-tools: [Bash, Read, Write, Skill, mcp__trinity__call_a2a_agent, mcp__trinity__get_a2a_task, mcp__trinity__chat_with_agent, mcp__trinity__get_execution_result, mcp__trinity__ask_operator, mcp__trinity__get_my_ask]
user-invocable: true
automation: autonomous
metadata:
  version: "0.2.0"
  changelog:
    - "0.2.0: the resident half of the Architect (trinity-enterprise#996, 2026-10-29): --from defaults to the hosted Architect endpoint named by ARCHITECT_ADVISOR when set; --fleet sends the brief to /design-fleet and stores the one target view it answers with (views/<fleet>/target-view.yaml), handing over to the library /compile-view; absorbs the retired Abilityai/architect template"
    - "0.1.0: first version — advisor via local agent or A2A endpoint (/design-agent → agent-spec/v1), spec passed to /create-agent:custom unedited, wizard-only fallback, resumable across the wizard's two asks"
---

# Build Agent

> ℹ️ `build-agent v0.2.0` — the hosted Architect is the default advisor when `ARCHITECT_ADVISOR` is set; `--fleet` designs a whole fleet into a target view. Unsubscribed: wizard alone, as before.

## Arguments

$ARGUMENTS

- **brief** — everything except the flags.
- `--from <advisor>` — where to get a design: `agent:<name>` (an agent on this instance, called with `chat_with_agent`) or `<endpoint>` (a registered outbound A2A endpoint, called with `call_a2a_agent`). Omitted → `${ARCHITECT_ADVISOR}` when set (the hosted Architect, see `/subscribe`); otherwise no advisor.
- `--fleet` — the brief describes a whole fleet, not one agent. Requires an advisor: send `/design-fleet <brief>` the same way as Step 1, accept only a reply whose first YAML block validates against `fleet-targets/schema/target-view.yaml`, store it unedited as `views/<fleet>/target-view.yaml` (fleet = the view's `fleet:` key), print the path, and stop: building the fleet is the library `/compile-view` → `/apply-view` → `/reconcile-fleet` path, each gated where the view says. Without an advisor, say so and stop — the wizard designs agents, not fleets.
- `--no-deploy` — scaffold and stop before the creation step.

## Step 0: Resume or start

Derive `<slug>` from the brief (kebab-case, ≤ 30 chars). If `~/builds/.build-<slug>.json` exists, this is a resumed run (the wizard is waiting on one of its asks): read it and go straight to **Step 3** with the stored brief — the wizard reads its own pending file and `get_my_ask` and continues.

Otherwise write `~/builds/.build-<slug>.json` = `{brief, from, no_deploy, started_at, design: null}`.

## Step 1: Get a design (only with `--from`)

Send exactly one call — `/design-agent <brief>` — never more than once per build.

- **`agent:<name>`** → `mcp__trinity__chat_with_agent(agent_name: <name>, message: "/design-agent <brief>", parallel: true, async: true)`, then poll `get_execution_result` until terminal (every ~30 s, at most 10 min).
- **`<endpoint>`** → `mcp__trinity__call_a2a_agent(agent_name: <this agent>, endpoint: <endpoint>, message: "/design-agent <brief>", dedup_label: "design-<slug>")`. State `working`/`submitted` → poll `get_a2a_task(task_id)` at most 10 min. A payment-required outcome means this instance has no payment set up for that advisor.

**Accept** the reply only if its first line starts with `# agent-spec/v1:`. Store it in the build file as `design`, and write it to `~/builds/<slug>.spec.md`.

**Fall back** (no advisor, wizard-only) on: unreachable endpoint, refusal, payment required, timeout, or a reply that is not an `agent-spec/v1` document. Record the reason in the build file. Never re-send.

## Step 2: Compose the wizard brief

- With a design → the brief **is** the spec, unedited, followed by one line: `Requested as: <original brief>`.
- Without → the original brief.
- Append `--no-deploy` intent as a line (`Do not create the agent on the instance; stop after scaffolding.`) when the flag was given.

## Step 3: Run the wizard

Invoke `/create-agent:custom` with the composed brief (Skill tool). The wizard runs headless: it may end the turn waiting on its **gaps ask** or its **approval ask** — that is normal. Report which ask is pending and stop; the next `/build-agent` call with the same brief resumes here.

**Archive packing (local path).** When the wizard takes the no-token path, build the archive with `scripts/pack-agent.sh <destination>` — it writes the `.trinity-manifest.json` the platform verifies and prints the base64 — rather than packing by hand.

## Step 4: Close

When the wizard reports the agent created (or scaffolded with `--no-deploy`):

1. Verify on evidence: `get_agent_info(<name>)` returns its template, and `list_agent_schedules(<name>)` shows every schedule `enabled: false`.
2. Delete `~/builds/.build-<slug>.json`.
3. Reply with: the agent's name, source mode (repo URL, or "uploaded — no reproducible source"), the advisor used or the fallback reason, the assumptions the wizard recorded, and the one next step for the person (review it, add credentials, arm schedules).
4. Guarded `mcp__trinity__report` (skip silently if absent or refused): `report_type: agent_builder.build`, `display_hint: markdown`, the same content.

## Errors

| Situation | Do |
|---|---|
| `create-agent` plugin missing | Stop: "the create-agent plugin is not installed — check template.yaml plugins and restart the agent" |
| Advisor fails in any way | Wizard-only, reason recorded and reported — never block the build on advice |
| Wizard's ask cancelled / declined | Keep the scaffold in `~/builds/<slug>`, report where it is, delete the build file |
| `deploy_local_agent` refused (`MANIFEST_DRIFT`, size) | Re-pack with the script once from a fresh copy; still refused → report the refusal verbatim |
