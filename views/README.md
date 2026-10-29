# views/ — Trinity-held target views

A **target view** says what a fleet is meant to be: seats, agents, edges, grants (each with an expiry), the approval map, assignments, credentials by name. It holds exactly the things the fleet — this agent included — must not change about itself. Skills, schedules and prompts are not here; outcomes are not here. Schema, examples and the validator: `github:Abilityai/fleet-targets` → `schema/`, `reference/`, `tools/validate.py`.

This directory is the view's home on an install **without a git review flow**. The alternative is a git repository people merge PRs into, which this agent's token can only read (`ARCHITECT_VIEW_SOURCE=git:github:<org>/<repo>`); then nothing lives here.

## Who writes here

**Nobody by hand after the first view.** `views/<fleet>/target-view.yaml` is written only by the gated library skill **`/apply-view`**: it validates a proposal, shows the diff, raises one approval to the fleet owner, and on approval copies the proposal over the held view and runs one reconcile check. That approval is the merge. This agent never edits a view itself; it writes proposals and asks.

The **first** view is the exception: a person copies a reference view from `fleet-targets/reference/`, edits it, validates it, commits it. Or, subscribed, `/build-agent --fleet <brief>` asks the hosted Architect's `/design-fleet` and stores the answer here unedited.

## Validating

```bash
[ -d fleet-targets/.git ] || git clone --quiet https://github.com/Abilityai/fleet-targets fleet-targets
python3 fleet-targets/tools/validate.py --view views/<fleet>/target-view.yaml
```

A view that fails validation is not a source; the check stops and says so.
