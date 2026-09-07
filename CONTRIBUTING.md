# Contributing

The skill is plain Markdown. There is no build step and nothing to compile.

## Layout

```
SKILL.md                          the loop, the node budget, the closing report
references/node-reduction.md      100-node patterns and their 10-node equivalents
references/documentation.md       comment and ledger format
references/mcp-operations.md      UE 5.8 MCP mechanics, the only server-specific file
.claude-plugin/                   plugin and marketplace manifests
```

`SKILL.md` is loaded into context every time the skill triggers, so it stays short. The
`references/` files are read only when a task needs them, so that is where detail belongs.

## What's useful to send

A before and after graph is the most valuable kind of issue: a real 100-node graph, the
version the skill produced, and the node counts for both.

Also useful:

- A substitution for `node-reduction.md`. A bloat pattern you keep seeing, and the node that
  replaces it.
- Corrections to `mcp-operations.md`. Unreal MCP is experimental in 5.8 and the API drifts. If
  a tool name, port, or console command in there is wrong on your build, say which build.
- Tool-name mappings for the non-Epic community servers.

## House style

Keep it to what an agent can act on. No motivational text, no restating engine docs.

Rules are hard numbers where possible, as in "over ~30 nodes, stop". Soft guidance loses to
momentum mid-build.

Wrap prose at about 95 columns, which is what the existing files do. Long table rows and URLs
are exempt. If you add a rule to `SKILL.md`, cut something: length costs context on every
trigger.

## Testing a change

Install your branch through [route B in INSTALL.md](INSTALL.md#route-b-plain-skill-folder-personal),
pointing the clone at your fork. Restart Claude Code and confirm with "what skills do I have?".
Then run a real task against a live editor and paste the closing report into the pull request.

Untested changes are still welcome. Just say so in the pull request rather than implying
otherwise.

## Filing an issue

Include the Unreal version, which MCP server you are running, and the graph or transcript that
went wrong. "It built 90 nodes" without the graph is hard to act on.
