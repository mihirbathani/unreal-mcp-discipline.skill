# Usage

Nothing to invoke, no syntax to learn. The skill declares when it applies, and Claude loads it
on its own the moment a task touches Unreal.

## 1. Just ask for the work

> Add a scoring path to BP_Scoreboard that ignores events during halftime.

> Make an actor that opens a door when the player overlaps it and closes it after 3 seconds.

> Wire the health bar in WBP_HUD to the player's health variable.

> Build a Niagara emitter for the muzzle flash and hook it to the fire event.

Naming an asset path (`/Game/Blueprints/BP_Foo`) or saying "in UE" is enough to trigger it.

**To force it**, say so: *"use unreal-mcp-discipline for this"*.

## 2. What changes in Claude's behaviour

Without the skill, an agent starts creating nodes immediately. With it, four things happen
first:

1. **It reads.** The target graph, the Blueprint's variables and functions, its parent class,
   and any existing Blueprint Function Library covering the same area. Most "write a new
   function" requests turn out to be "call the one that already exists".
2. **It plans in text and counts.** You get the node plan in chat before a single write call —
   entry point, blocks, estimated node count, and which existing functions get reused. Over
   ~30 nodes, it splits into functions *on paper*, which costs a paragraph instead of twenty
   tool calls.
3. **It batches.** One script execution per logical unit, not fifty add-node round trips.
4. **It documents while building.** Graph header comment, block comments, a Description on
   every function, and a dated ledger entry in your repo.

## 3. Read the plan before saying go

The plan step is the cheap place to intervene. If the count looks wrong, or it missed a
function you already have, say so there:

> That's 40 nodes. We already have BPFL_ScoreMath — use ClampDelta instead of rebuilding it.

Correcting a plan is one message. Correcting a built graph is a refactor.

## 4. What you get at the end

Every session that touched the editor closes with a fixed report:

```
Assets touched:   /Game/... (created | modified)
Graphs:           <graph name> - <N> nodes, <M> functions extracted
Reused:           <existing functions/interfaces called instead of rebuilt>
Compile:          clean | <errors and what was done>
Docs:             header + block comments + descriptions | ledger updated at <path>
Orphans removed:  <N>
Not built:        <things suggested but deliberately left out>
```

`Not built` is the line to read. It is how you tell a deliberate gap from an oversight.

## 5. Cleaning up a graph someone else (or some other agent) built

Point it at the mess:

> Audit BP_PlayerController — a previous agent built it and I can't read it.

> This event graph is 140 nodes. Clean it up.

> Nobody knows what BP_MatchFlow does. Document it.

It will **not** silently rewrite working logic. The sequence is: count nodes per function,
list duplicated subgraphs / dead branches / orphans / Tick polling with counts, propose the
refactor and the expected node count after, then **wait for your go-ahead**. Refactors land
one function at a time, compiling between each, so a failure is bisectable.

If nobody can say what the graph does, documenting it is the first deliverable — before any
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

It's a suggestion, not a veto. Say "do it in Blueprint anyway" and it will.

## 7. The ledger

The skill appends a dated entry to your project's agent notes — it looks for `CLAUDE.md`,
`AGENTS.md`, or a `Docs/` convention first and follows whatever you already use. It asks
before creating a new file in a repo it didn't set up.

Each entry records assets touched, what changed, why, what was reused instead of rebuilt,
what's still open, and how it was verified. The **Open / not done** section is what makes the
next session cheap — including Claude's own next session, which starts with no memory of this
one.

Format and templates: [references/documentation.md](references/documentation.md).

## 8. Habits worth keeping

- **Commit before a long MCP session.** MCP mutates live UObject state. An unreviewable diff
  is the usual regret.
- **Review the Blueprint diff** in the editor's asset diff tool rather than trusting "the
  change was small".
- **A timeout is not a failure.** The skill re-reads state before retrying so you don't get
  duplicate actors — but if you're driving manually, do the same.
- **Treat editor Python as privileged.** It can move, mutate, or delete project content with
  no second confirmation.

---

Back to [README.md](README.md) · Install steps in [INSTALL.md](INSTALL.md)
