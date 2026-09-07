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
- Write a graph header comment, a description on every function, and a dated entry in the
  repo saying what changed, why, and what is still open.

Built for the official Unreal MCP in UE 5.8 (ModelContextProtocol plugin, port 8000, the
`list_toolsets` / `describe_toolset` / `call_tool` meta-tools). If you run one of the
community MCP servers, edit the tool names in `references/mcp-operations.md`. The other two
reference files are server-agnostic.

## Quick start

Claude Code, one command:

```bash
/plugin marketplace add mihirbathani/unreal-mcp-discipline.skill
```

then:

```bash
/plugin install unreal-mcp-discipline@unreal-mcp-discipline
```

Restart Claude Code. That's it — there is nothing to invoke, the skill loads itself when a
task mentions Unreal.

Other install routes (plain skill folder, project-level, claude.ai / desktop upload) are in
**[INSTALL.md](INSTALL.md)**, written step by step.

## Documentation

| File | What's in it |
| --- | --- |
| **[INSTALL.md](INSTALL.md)** | Every install route, step by step, plus how to verify it loaded |
| **[USAGE.md](USAGE.md)** | How to use it day to day, example prompts, what output to expect |
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

- Unreal Engine 5.8 with the **ModelContextProtocol** plugin enabled, or a community Unreal
  MCP server (edit the tool names in `references/mcp-operations.md` if so).
- Claude Code, the Claude desktop app, or claude.ai with Skills enabled.

The skill is plain Markdown. It has no dependencies, runs no code, and does not talk to the
network itself — your MCP server does that.

## Status

Early. Nothing here has been run against a live production build. Bug reports and graph
before/afters are welcome as issues.

## License

MIT — see [LICENSE](LICENSE).
