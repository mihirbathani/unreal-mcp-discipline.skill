# Contributing

The skill is plain Markdown. There is no build step and nothing to compile.

## Layout

```
SKILL.md                          the loop, the node budget, the closing report
references/node-reduction.md      100-node patterns and their 10-node equivalents
references/documentation.md       comment and ledger format
references/mcp-operations.md      UE 5.8 MCP mechanics (the only server-specific file)
.claude-plugin/                   plugin + marketplace manifests
```

`SKILL.md` is loaded into context every time the skill triggers, so it stays short. The
`references/` files are read only when a task needs them — that's where detail belongs.

## What's useful to send

- **A before/after graph.** A real 100-node graph and the version the skill produced, with
  node counts. This is the most valuable kind of issue.
- **A substitution for `node-reduction.md`.** A bloat pattern you keep seeing, and the node
  that replaces it.
- **Corrections to `mcp-operations.md`.** Unreal MCP is experimental in 5.8 and the API drifts.
  If a tool name, port, or console command here is wrong on your build, say which build.
- **Community server notes.** Tool-name mappings for the non-Epic servers.

## House style

- Keep it to what an agent can act on. No motivational text, no restating engine docs.
- Rules are hard numbers where possible ("over ~30 nodes, stop") — soft guidance loses to
  momentum mid-build.
- Wrap at 100 columns.
- If you add a rule to `SKILL.md`, cut something. Length costs context on every trigger.

## Testing a change

1. Install your branch via [Route B in INSTALL.md](INSTALL.md#route-b--plain-skill-folder-personal),
   pointing the clone at your fork.
2. Restart Claude Code and confirm with "what skills do I have?".
3. Run a real task against a live editor and paste the closing report into the PR.

Untested changes are still welcome — just say so in the PR, don't imply otherwise.

## Filing an issue

Include the Unreal version, which MCP server you're running, and the graph or transcript that
went wrong. "It built 90 nodes" without the graph is hard to act on.
