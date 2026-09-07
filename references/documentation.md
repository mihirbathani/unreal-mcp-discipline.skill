# Documentation contract

Two audiences, both of whom have zero context: a human opening the graph in a month, and the
next agent session - which includes you, with no memory of this one. Documentation written
only inside the chat response is lost the moment the conversation ends. It has to live in the
asset and in the repo.

## 1. Graph header comment

One comment box at the top-left of every graph touched, before any logic. Keep it to what
cannot be read off the nodes.

```
[BP_Scoreboard :: UpdateScore]
Purpose : Applies a scoring event to the live score state and pushes it to the overlay widget.
Runs on : Score event dispatcher from GM_Match. Not on Tick.
Reads   : MatchState (enum), CurrentScore (struct)
Writes  : CurrentScore, fires OnScoreChanged
Gotcha  : Ignores events while MatchState is Halftime - deliberate, see MatchState enum.
Edited  : agent, 2026-09-07
```

`Gotcha` is the highest-value line and the one most often missing. Anything that would look
like a bug to a reader but is intentional goes there. If there is nothing, drop the line.

## 2. Block comments

A comment box around each logical block - typically 3 to 10 nodes. The title says why the
block exists.

Good: `Debounce - overlay refreshes at 30Hz max, broadcaster drops frames above that`
Useless: `Branch on IsValid` (restates the node), `Main logic` (says nothing)

Suggested colour convention, stated once in the header so it is discoverable:
setup and init, per-frame or timed, network or replication, debug-only. Any consistent scheme
works; consistency across the project is what matters, so check what the project already uses
before inventing one.

## 3. Function and macro descriptions

Every function and macro created gets a Description in its Details panel. That string is the
hover tooltip everywhere the function is called, so it is the only documentation most readers
will ever see.

One or two sentences, ending with the failure mode:
`Returns the clamped score delta for a scoring event. Returns 0 and logs a warning if the
event type is not in the scoring table.`

## 4. Variables

Every variable created gets:
- A **Category** - grouping is what makes a 40-variable Blueprint navigable.
- A **Tooltip** - required for anything instance-editable or exposed on spawn. A designer
  changing it in the Details panel has no other source of truth.
- A name that carries type intent: `b` prefix for booleans, plural for arrays, no
  abbreviations that are not already project convention.

## 5. The agent ledger

The part that survives the session. Append to the project's agent notes file - check for
`CLAUDE.md`, `AGENTS.md`, or a `Docs/` convention first and follow it; only create
`Docs/Agents/blueprint-log.md` if nothing exists. Ask before creating a new file in a repo
you did not set up.

```markdown
## 2026-09-07 - Scoreboard overlay scoring path

**Assets**
- /Game/Blueprints/BP_Scoreboard (modified)
- /Game/Blueprints/Lib/BPFL_ScoreMath (created)

**What changed**
Moved score clamping out of three copies in BP_Scoreboard into BPFL_ScoreMath::ClampDelta.
UpdateScore now dispatches on the MatchState enum instead of a four-Branch ladder.

**Why**
Clamping rules were drifting between the copies; the halftime case was only handled in one.

**Reused, not rebuilt**
GM_Match already exposed OnScoreChanged - bound to it instead of polling.

**Open / not done**
- Overtime is not in the MatchState enum yet. UpdateScore treats it as Live. Confirm intended.
- No automation test on ClampDelta.

**Verify**
Compile clean. PIE: scored during halftime, score correctly unchanged.
```

`Open / not done` is what makes the next session cheap. Without it the next agent re-derives
the same open questions, or worse, silently answers one wrong.

## 6. What not to write

- Node-by-node narration. If the comment count approaches the node count, the graph is
  under-named and over-commented; fix the naming.
- Comments restating the node name.
- Aspirational comments describing what the graph should do rather than what it does.
- Changelog-in-the-graph. History belongs in source control and the ledger, not in comment
  boxes that grow forever.
- A ledger entry for a read-only session. Nothing changed; there is nothing to record.
