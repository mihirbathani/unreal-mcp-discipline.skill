# Documentation contract

Two audiences, both of whom have zero context: a human opening the graph in a month, and the
next agent session - which includes you, with no memory of this one. Documentation written
only inside the chat response is lost the moment the conversation ends.

**MCP cannot place comment boxes in a Blueprint graph.** That is the constraint this whole
file is built around. Check it against the installed build if the toolset changes, but until
then: no header comment, no block comments, no colour-coded regions. Everything below is where
the documentation goes instead.

## 1. Names, because they are all that stays in the graph

With comment boxes unavailable, the name is the comment. This moves naming from a style
preference to the primary deliverable.

- Functions are named for intent and outcome, not mechanism. `CanFireWeapon`, not
  `CheckWeaponBool`. `ApplyScoreDelta`, not `DoScoreStuff`.
- A name that needs "And" is two functions. `ValidateAndApplyScore` splits.
- Booleans take the `b` prefix and read as a claim that is true or false: `bIsHalftime`, not
  `bHalftimeCheck`.
- Arrays are plural. Maps are named for the lookup they do: `ScoreByPlayerId`.
- No abbreviations that are not already project convention. The next reader does not know
  that `SCMgr` was obvious at the time.
- Enum values carry the meaning that a comment would have: `Halftime`, not `State2`.

A graph with good names and no comments is readable. A graph with vague names and no comments
is a rewrite. There is no comment box available to rescue the second case.

## 2. Descriptions, Categories and tooltips, if the toolset allows

These live on the asset rather than in the graph, so they may be settable even though comments
are not. Call `describe_toolset` and find out rather than assuming either way.

Every function and macro gets a Description if one can be set. That string is the hover
tooltip everywhere the function is called, so it is the only in-editor documentation most
readers will ever see. One or two sentences, ending with the failure mode:

```
Returns the clamped score delta for a scoring event. Returns 0 and logs a warning if the
event type is not in the scoring table.
```

Every variable gets a Category, which is what makes a 40-variable Blueprint navigable, and a
tooltip if it is instance-editable or exposed on spawn. A designer changing a value in the
Details panel has no other source of truth.

Report which of these the toolset refused. "Descriptions set, variable tooltips not supported
by the toolset" is useful. Silence reads as "done".

## 3. Handoff comment text

Since the comment boxes cannot be placed by MCP, write them out so a human can paste them in.
Put this below the closing report. Keep it to what cannot be read off the nodes.

```
[BP_Scoreboard :: UpdateScore]
Purpose : Applies a scoring event to the live score state and pushes it to the overlay widget.
Runs on : Score event dispatcher from GM_Match. Not on Tick.
Reads   : MatchState (enum), CurrentScore (struct)
Writes  : CurrentScore, fires OnScoreChanged
Gotcha  : Ignores events while MatchState is Halftime - deliberate, see MatchState enum.
Edited  : agent, 2026-10-11
```

`Gotcha` is the highest-value line and the one most often missing. Anything that would look
like a bug to a reader but is intentional goes there. If there is nothing, drop the line.

For block comments, name the node range and the reason the block exists:

```
Around the Set Timer / Compare pair:  "Debounce - overlay refreshes at 30Hz max, the
                                       broadcaster drops frames above that"
```

Good block text says why. Useless block text restates the node: `Branch on IsValid` tells the
reader nothing they cannot see, and `Main logic` tells them less.

The user may not paste any of it. That is fine. It costs a few lines to offer and the ledger
still holds the same information.

## 4. Content/Docs/agent-log.md

The running record of what agents changed, and the part that survives the session. One dated
entry per session that changed something. Create the file if it is missing. If the project
already keeps this somewhere else, a `CLAUDE.md`, an `AGENTS.md`, an existing `Docs/`
convention, append there instead of starting a competing file.

```markdown
## 2026-10-11 - Scoreboard overlay scoring path

**Assets**
- /Game/Blueprints/BP_Scoreboard (modified)
- /Game/Blueprints/Lib/BPFL_ScoreMath (created)

**Backup**
Saved/AgentBackups/2026-10-11-1432/ - pre-session commit a3f91c2

**What changed**
Moved score clamping out of three copies in BP_Scoreboard into BPFL_ScoreMath::ClampDelta.
UpdateScore now dispatches on the MatchState enum instead of a four-Branch ladder.

**Why**
Clamping rules were drifting between the copies; the halftime case was only handled in one.

**Reused, not rebuilt**
GM_Match already exposed OnScoreChanged - bound to it instead of polling.

**Not in the graph**
No comment boxes - MCP cannot place them. Header text is in the session report if anyone
wants to paste it. Function Descriptions were set; variable tooltips were not supported.

**Open / not done**
- Overtime is not in the MatchState enum yet. UpdateScore treats it as Live. Confirm intended.
- No automation test on ClampDelta.

**Verify**
Compile clean. PIE: scored during halftime, score correctly unchanged.
```

`Open / not done` is what makes the next session cheap. Without it the next agent re-derives
the same open questions, or worse, silently answers one wrong.

## 5. Content/Docs/blueprint-map.md

The log says what changed. The map says what exists. It is the file a new person, or a new
agent session, reads first to find out what the project is made of without opening fifty
assets.

Update the entry for any Blueprint touched this session. Add an entry if there is none. Do
not try to document every asset in the project; a map of everything is a map of nothing. An
entry earns its place if someone could break something by not knowing about it.

```markdown
# Blueprint map

Last updated: 2026-10-11

## Systems

- **Match flow** - GM_Match drives state, BP_Scoreboard and WBP_Overlay react to it.
- **Scoring** - BPFL_ScoreMath holds the rules, nothing else is allowed to clamp scores.

## /Game/Blueprints/BP_Scoreboard

Type      : Actor
Purpose   : Holds live score state and pushes it to the overlay widget.
Entry     : OnScoreChanged dispatcher from GM_Match. Not on Tick.
Reads     : MatchState (enum), CurrentScore (struct)
Writes    : CurrentScore, fires OnScoreChanged
Functions : UpdateScore (24 nodes), ApplyPenalty (9 nodes)
Talks to  : GM_Match (binds to its dispatcher), WBP_Overlay (owns the widget)
Gotcha    : Ignores scoring events while MatchState is Halftime. Deliberate.
Risk      : Sole writer of CurrentScore. Anything else writing it is a bug.

## /Game/Blueprints/Lib/BPFL_ScoreMath

Type      : Blueprint Function Library (static)
Purpose   : Scoring arithmetic shared by the scoreboard and the match rules.
Entry     : Called, never fired.
Functions : ClampDelta (pure, 6 nodes)
Talks to  : Nothing. No self reference, no world context. Keep it that way.
```

The `Gotcha` and `Risk` lines carry the most weight, because they are the two things a reader
cannot work out from the graph in under ten minutes. `Entry` is next: knowing what fires a
Blueprint is most of knowing what it is for.

Keep entries short enough that the file stays readable at fifty Blueprints. If an entry wants
to become a paragraph, the Blueprint probably wants to become two Blueprints.

## 6. What not to write

- Node-by-node narration in the handoff text. If it approaches one line per node, the graph is
  under-named; fix the naming instead.
- Handoff text restating node names.
- Aspirational text describing what the graph should do rather than what it does.
- Changelog prose in the map. The map is present tense: what is there now. History lives in
  `agent-log.md` and in source control.
- An `agent-log.md` entry for a read-only session. Nothing changed; there is nothing to record.
- A claim that the graph is commented. It is not, and it cannot be.
