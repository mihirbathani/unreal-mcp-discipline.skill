# unreal-mcp-discipline

A skill for Claude that keeps agent-built Unreal Blueprint graphs small, documented, and
reviewable.

Driving the editor over MCP makes it cheap to add nodes and expensive to review them. The
usual result is a graph that compiles, works, and is unmaintainable: 100 nodes for 10 nodes of
logic, the same subgraph pasted four times, no comments, and no record of what changed or why.

This skill sets rules the agent follows before it touches the editor:

- Read the graph and the project's existing function libraries before creating any node.
- State the node plan and a count estimate first.
- Extract to a function on the third repeat of a block.
- Stop and refactor above roughly 30 nodes in one graph.
- Delete anything left disconnected.
- Document where MCP actually can: names, asset descriptions, and a dated entry in the repo
  saying what changed, why, and what is still open. MCP cannot place comment boxes in a graph,
  so the skill hands you that text to paste instead of pretending the graph is commented.

Built for the official Unreal MCP in UE 5.8 (ModelContextProtocol plugin, port 8000, the
`list_toolsets` / `describe_toolset` / `call_tool` meta-tools). If you run one of the
community MCP servers, edit the tool names in `references/mcp-operations.md`. The other two
reference files are server-agnostic.

## Quick start

In Claude Code:

```
/plugin marketplace add mihirbathani/unreal-mcp-discipline.skill
/plugin install unreal-mcp-discipline@unreal-mcp-discipline
```

Restart Claude Code. There is nothing to invoke after that; the skill loads itself when a task
mentions Unreal.

The other install routes are in [INSTALL.md](INSTALL.md), written step by step: a plain skill
folder, a project-level copy that travels with your Unreal project, and a zip upload for
claude.ai and the desktop app.

## Documentation

| File | What's in it |
| --- | --- |
| [INSTALL.md](INSTALL.md) | Every install route, step by step, plus how to check it loaded |
| [USAGE.md](USAGE.md) | What to type, what to expect back, how to clean up an existing graph |
| [SKILL.md](SKILL.md) | The skill itself: the loop and the node budget |
| [references/node-reduction.md](references/node-reduction.md) | Substitution catalogue: Branch ladders to Switch, cast chains to interfaces, copy-paste to functions, Tick polling to dispatchers |
| [references/documentation.md](references/documentation.md) | Comment and ledger format |
| [references/mcp-operations.md](references/mcp-operations.md) | UE 5.8 MCP mechanics: batching, serial calls, timeouts, verification |

Claude reads the references only when a task needs them, so the skill stays cheap to have
installed.

## Use

There is nothing to invoke. Claude loads the skill when the task matches the description, so
an ordinary request works:

> Add a scoring path to BP_Scoreboard that ignores events during halftime.

To force it, name it: "use unreal-mcp-discipline for this".

It also applies to cleanup work. Point it at a graph a previous agent built and it counts
nodes, lists duplicated subgraphs and orphans, and proposes a refactor before touching
anything.

## Requirements

Unreal Engine 5.8 with the ModelContextProtocol plugin enabled, or a community Unreal MCP
server. If you use one of those, edit the tool names in `references/mcp-operations.md` to
match it. On the Claude side: Claude Code, the desktop app, or claude.ai with Skills enabled.

The skill is plain Markdown. It has no dependencies and runs no code. Your MCP server is what
talks to the editor.

## Status

Early. Nothing here has been run against a live production build. Bug reports and graph
before/afters are welcome as issues.

## License

MIT. See [LICENSE](LICENSE).
