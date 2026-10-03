# Agent Builder

A Trinity agent that creates other agents. Tell it what you need in a paragraph; it runs the [Ability.ai create-agent wizard](https://github.com/Abilityai/abilities) headlessly, asks you once for anything missing and once for approval, and creates the new agent on your instance with every schedule switched off until you review it.

```
/build-agent An agent that watches our support queue and drafts first replies for a human to send
/build-agent <brief> --from agent:<advisor>               # get a design from an advisor agent first
/build-agent <brief> --no-deploy                          # scaffold only
```

- **No credentials needed.** With a GitHub token set as this agent's GitHub PAT, each new agent gets its own private repo; without one it is uploaded directly (no reproducible source — the builder says so).
- **Advisors are optional.** `--from` takes an agent on this instance (`agent:<name>`) or a registered A2A endpoint. The exchange is one call — `/design-agent <brief>` → one `agent-spec/v1` document — and any advisor failure falls back to the wizard alone.
