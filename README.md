# Agent Builder

A Trinity agent that creates other agents — and, as the resident half of the Architect, keeps your fleet to its target view. Tell it what you need in a paragraph; it runs the [Ability.ai create-agent wizard](https://github.com/Abilityai/abilities) headlessly, asks you once for anything missing and once for approval, and creates the new agent on your instance with every schedule switched off until you review it.

```
/build-agent An agent that watches our support queue and drafts first replies for a human to send
/build-agent <brief> --from agent:<advisor>               # get a design from an advisor agent first
/build-agent <brief> --no-deploy                          # scaffold only
/build-agent <brief> --fleet                              # subscribed: design a whole fleet into one target view
/reconcile-fleet --check                                  # library set `architect`: what drifted, what is unobserved
/subscribe                                                # connect to the hosted Architect (an admin registers the endpoint)
```

- **No credentials needed.** With a GitHub token set as this agent's GitHub PAT, each new agent gets its own private repo; without one it is uploaded directly (no reproducible source — the builder says so).
- **Advisors are optional.** `--from` takes an agent on this instance (`agent:<name>`) or a registered A2A endpoint. The exchange is one call — `/design-agent <brief>` → one `agent-spec/v1` document — and any advisor failure falls back to the wizard alone.

- **Fleet verbs come from the library.** Assign the skill set `architect` from [abilityai/trinity-skills](https://github.com/Abilityai/trinity-skills) in Library → Skills: `/subscribe` connects to the hosted Architect, `/reconcile-fleet` diffs the live fleet against the target view (statuses in sync / drift / unobserved / schema-gap, nothing inferred), `/compile-view` turns a view into a Trinity SystemManifest, `/apply-view` merges a proposal behind one approval. The view lives in a repo people merge into, or in `views/` here (see `views/README.md`). Schema and reference views: [Abilityai/fleet-targets](https://github.com/Abilityai/fleet-targets) `schema/`, `reference/`.
- **Subscribed, it has a brain.** `/subscribe` connects this agent to the hosted Architect over A2A (signed as this agent, no stored credential). Then `/build-agent` gets its designs from doctrine, `--fleet` designs whole fleets, and the hosted side proposes the next view when gaps age. Unsubscribed, everything above still works; only the designs come from the wizard alone.
