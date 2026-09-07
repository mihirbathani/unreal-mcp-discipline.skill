---
name: unreal-mcp-discipline
description: Build small, documented, reviewable Blueprint graphs when driving Unreal Editor over MCP, instead of hundred-node spaghetti with no comments. Use this skill whenever the task touches Unreal Engine through MCP or the Unreal Editor at all - creating or editing Blueprints, adding or connecting nodes, building UMG widgets, Niagara, Sequencer, materials, Control Rigs, spawning actors, or running editor Python - even when the user only says "add this to the Blueprint", "wire this up in UE", "make an actor that does X", or names an asset path like /Game/Blueprints/BP_Foo. Also use it when reviewing, auditing, cleaning up or documenting graphs an earlier agent built, and when the user complains about node bloat, duplicated logic, missing comments, or an agent burning tool calls in the editor.
---

# Unreal MCP discipline

Driving the editor over MCP makes it cheap to add nodes and expensive to review them. The
default failure is not a broken graph - it is a graph that compiles, works, and is
unmaintainable: 100 nodes for 10 nodes of logic, the same subgraph pasted four times, no
comments, and no record of what changed or why. A human opens it a week later, cannot tell
what is load-bearing, and rewrites it. The MCP session was then a net loss.

Treat every node as something a human will have to read on a 1080p monitor at 11pm before a
show. That is the standard.

## The loop

**1. Read before writing.** No node is created before the current state is known.
- `list_toolsets` -> `describe_toolset` -> `call_tool`. Never guess a tool name or an
  argument schema; the schema is authoritative and versions drift.
- Read the target graph, the Blueprint's variables and functions, its parent class, and any
  Blueprint Function Library or interface the project already has for this area. Most
  "write a new function" tasks are actually "call the one that exists".
- If the asset is under source control, confirm it is saved and committed before a bulk edit.
  MCP mutates live UObject state; an unreviewable diff is the usual regret.

**2. Plan the graph in text, then count.** Before any write call, state the node plan in
chat: entry point, the blocks, the node count estimate, and which existing functions get
reused. If the estimate is over ~30 nodes in one graph, the plan is wrong - split it into
functions before building, not after. It costs one paragraph to fix on paper and twenty tool
calls to fix in the editor.

**3. Build in batches, not per-node round trips.** Once the plan is settled, create the nodes
in as few calls as possible - the Scripting toolset's script execution path exists for this.
Fifty individual add-node and connect-pin calls for one function is a tooling failure, not
thoroughness. Keep each batch idempotent: check for an existing node or asset before
creating it, so a timeout retry does not leave duplicates behind. A timeout is not a
failure - re-read state before retrying.

**4. Compile and verify each unit.** Compile after each function or event, read the actual
result, and fix errors before adding more. Do not stack three unverified units and compile
once at the end; the error list is then unattributable.

**5. Document as you build, not as a final pass.** See `references/documentation.md`. The
comment header, the function description, and the ledger entry are part of the deliverable,
not decoration.

**6. Audit before declaring done.** Run the checklist below and report it. If something
fails, fix it and re-report rather than noting it as a caveat.

## Node budget

Hard rules, because soft guidance loses to momentum mid-build:

- **Same pattern twice, extract on the third.** Two occurrences of an identical subgraph is a
  warning; three is a defect. Collapse to a function (pure if it has no side effects), a
  macro if it needs multiple exec outputs or a latent node, or a Blueprint Function Library
  entry if two Blueprints need it.
- **Over ~30 nodes in a graph, stop and refactor.** Event graphs that only route to functions
  can stay small; the work belongs inside the functions.
- **Over ~8 nodes in a straight chain with no branch, look for a single node that does it.**
  Engine nodes exist for most of it.
- **No node exists twice on both sides of a Branch.** Move it past the merge.
- **Zero orphan nodes.** Anything disconnected that the session created gets deleted before
  finishing. Half-finished attempts left floating are worse than nothing - the next agent
  cannot tell them from intentional work.
- **Zero unrequested extras.** No debug Print String left in, no "helpful" error handling, no
  sample content nobody asked for. Suggest extras in chat; build only what was asked.

`references/node-reduction.md` has the substitution catalogue - the specific 100-node
patterns and their 10-node equivalents. Read it before planning any graph with branching,
casting, repeated logic, or per-frame work.

## Documentation contract

Minimum for any graph this session touches:

- A comment box at the graph origin: purpose, when it runs, what it reads and writes, side
  effects, and the fact that an agent last edited it with the date.
- Comment boxes around each logical block. Comment the *why*, never restate the node name.
- A Description on every function and macro created - that is the hover tooltip other graphs
  and other agents see.
- Category and tooltip on every variable created, especially instance-editable ones.
- A ledger entry in the project's agent notes file recording asset path, what changed, why,
  and anything left open.

Format and templates in `references/documentation.md`. The ledger is the part agents skip and
the part that matters most for the next session - including your own next session, which
starts with no memory of this one.

## When Blueprint is the wrong answer

Push back instead of building:
- Per-frame maths, tight loops, or anything hot - C++ or an existing engine node.
- Tunable numbers scattered as literals - a Data Asset, Data Table, or curve.
- One-off bulk editor changes across many assets - an editor Python script, not a Blueprint.
- Type-checking cast chains - a Blueprint Interface.
- Polling on Tick - event dispatchers, timers, or a timeline.

## Operating rules

- **Serial only.** Tool calls execute on the game thread. Parallel calls into the editor
  deadlock it. One call at a time, always.
- **Save before and after** any bulk change.
- **Arrange nodes** after building. An unarranged graph is an undocumented graph.
- **Verify with a read**, not an assumption. Re-read the graph, or take a viewport or graph
  screenshot, before saying it works.
- **Arbitrary Python execution inside the editor is privileged.** It can move, mutate, or
  delete project content with no second confirmation. Use it for batching known-good
  operations, not for exploration, and never for anything destructive without asking first.

`references/mcp-operations.md` has the UE 5.8 specifics - server, meta-tools, batching
patterns, retry and timeout handling.

## Closing report

End every session that touched the editor with this, and nothing longer:

```
Assets touched:   /Game/... (created | modified)
Graphs:           <graph name> - <N> nodes, <M> functions extracted
Reused:           <existing functions/interfaces called instead of rebuilt>
Compile:          clean | <errors and what was done>
Docs:             header + block comments + descriptions | ledger updated at <path>
Orphans removed:  <N>
Not built:        <things suggested but deliberately left out>
```

`Not built` is not padding. It is how the next person knows a gap was a decision rather than
an oversight.

## Auditing someone else's graph

When the task is cleaning up what a previous agent left:

1. Read the whole graph and count nodes per function before changing anything.
2. Identify duplicated subgraphs, dead branches, orphans, and Tick polling. List them with
   node counts.
3. Propose the refactor and the expected node count after, then wait for a go-ahead. Do not
   silently rewrite working production logic.
4. Refactor one function at a time, compiling between each. A single big rewrite that fails
   to compile is unbisectable.
5. Document what the graph does *before* changing it - if nobody can say what it does, that
   is the first deliverable, not the refactor.
