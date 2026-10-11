# Usage

There is nothing to invoke and no syntax to learn. The skill declares when it applies, and
Claude loads it as soon as a task touches Unreal. It loads once and stays active for the rest
of the session, so there is no need to name it again on every message.

## 1. Ask for the work

> Add a scoring path to BP_Scoreboard that ignores events during halftime.

> Make an actor that opens a door when the player overlaps it and closes it after 3 seconds.

> Wire the health bar in WBP_HUD to the player's health variable.

> Build a Niagara emitter for the muzzle flash and hook it to the fire event.

Naming an asset path like `/Game/Blueprints/BP_Foo`, or just saying "in UE", is enough to
trigger it. To force it, say "use unreal-mcp-discipline for this".

## 2. What changes

Without the skill, an agent starts creating nodes immediately. With it, four things happen
first.

1. It reads the target graph, the Blueprint's variables and functions, its parent class, and
   any existing Blueprint Function Library covering the same area. Most "write a new function"
   requests turn out to be "call the one that already exists".
2. It plans in text and counts. You get the node plan in chat before a single write call:
   entry point, blocks, estimated node count, and which existing functions get reused. Over
   roughly 30 nodes it splits the plan into functions on paper, which costs a paragraph
   instead of twenty tool calls.
3. It batches. One script execution per logical unit, not fifty add-node round trips.
4. It documents where MCP can. Names first, since they are the only thing that stays attached
   to the graph, then asset Descriptions and Categories if the toolset exposes them, then a
   dated ledger entry in your repo. MCP cannot place comment boxes in a graph, so the header
   and block comment text comes back in the report for you to paste, or ignore.

## 3. Read the plan before saying go

The plan step is the cheap place to intervene. If the count looks wrong, or it missed a
function you already have, say so there:

> That's 40 nodes. We already have BPFL_ScoreMath, so use ClampDelta instead of rebuilding it.

Correcting a plan is one message. Correcting a built graph is a refactor.

## 4. What you get at the end

Every session that touched the editor closes with a fixed report:

```
Assets touched:   /Game/... (created | modified)
Graphs:           <graph name> - <N> nodes, <M> functions extracted
Reused:           <existing functions/interfaces called instead of rebuilt>
Compile:          clean | <errors and what was done>
Docs:             descriptions set | <what the toolset refused> | ledger at <path>
Orphans removed:  <N>
Not built:        <things suggested but deliberately left out>
```

`Not built` is the line worth reading. It is how you tell a deliberate gap from an oversight.

## 5. Cleaning up a graph someone else built

Point it at the mess:

> Audit BP_PlayerController. A previous agent built it and I can't read it.

> This event graph is 140 nodes. Clean it up.

> Nobody knows what BP_MatchFlow does. Document it.

It will not silently rewrite working logic. The sequence is: count nodes per function, then
list duplicated subgraphs, dead branches, orphans and Tick polling with their node counts,
then propose the refactor and the expected count afterwards, then wait for your go-ahead.
Refactors land one function at a time, compiling between each, so a failure is bisectable.

If nobody can say what the graph does, documenting it is the first deliverable, before any
refactor.

## 6. When it pushes back

The skill tells Claude to argue instead of building in these cases:

| You asked for | It will suggest |
| --- | --- |
| Per-frame maths or a tight loop in Blueprint | C++, or an existing engine node |
| Tunable numbers as scattered literals | A Data Asset, Data Table, or curve |
| A one-off bulk change across many assets | An editor Python script, not a Blueprint |
| A cast chain doing type checks | A Blueprint Interface |
| Polling on Tick | Event dispatchers, timers, or a timeline |

It is a suggestion, not a veto. Say "do it in Blueprint anyway" and it will.

## 7. The ledger

The skill appends a dated entry to your project's agent notes. It looks for `CLAUDE.md`,
`AGENTS.md`, or a `Docs/` convention first and follows whatever you already use, and it asks
before creating a new file in a repo it did not set up.

Each entry records the assets touched, what changed, why, what was reused instead of rebuilt,
what is still open, and how it was verified. The "Open / not done" section is what makes the
next session cheap, including Claude's own next session, which starts with no memory of this
one.

Format and templates are in [references/documentation.md](references/documentation.md).

## 8. Habits worth keeping

Commit before a long MCP session. MCP mutates live UObject state, and an unreviewable diff is
the usual regret. Review the Blueprint diff in the editor's asset diff tool rather than
trusting that the change was small.

A timeout is not a failure. It means the response never arrived, not that the operation did
not happen. The skill re-reads state before retrying so you do not end up with duplicate
actors, and if you are driving the editor by hand, do the same.

Treat editor Python as privileged. It can move, mutate, or delete project content with no
second confirmation.

---

Back to [README.md](README.md). Install steps are in [INSTALL.md](INSTALL.md).
