# Changelog

## 1.1.0, 2026-10-11

Reworked around a capability the first release assumed wrongly.

- MCP cannot place comment boxes in a Blueprint graph. Every rule that depended on them is
  gone. Documentation now rests on naming, asset Descriptions and Categories where the toolset
  exposes them, the repo ledger, and comment text handed back for the user to paste.
- Added a session scope note: the skill loads once and stays active, so it does not need
  re-invoking or restating per message.
- Added a "questions worth asking" section, with an equal list of questions not worth asking.
- Added one undo transaction per logical unit, so a single Ctrl+Z reverses a function's worth
  of work.
- Node positions are now set explicitly on creation rather than assuming an auto-arrange tool.
- Setup corrected: `ToolsetRegistry` and `AllToolsets` are needed alongside
  `ModelContextProtocol`. Documented `RefreshTools` and `GenerateClientConfig`.

## 1.0.0, 2026-09-07

First public release.

- `SKILL.md`: the six-step loop (read, plan and count, batch, compile and verify, document,
  audit), the node budget, the documentation contract, the list of cases where Blueprint is
  the wrong answer, and the fixed closing report.
- `references/node-reduction.md`: the substitution catalogue. Branch ladders to Switch, cast
  chains to interfaces, copy-paste to functions, Tick polling to dispatchers.
- `references/documentation.md`: naming rules, asset metadata, the handoff comment text, and
  the agent ledger format.
- `references/mcp-operations.md`: UE 5.8 MCP mechanics. The three meta-tools, serial
  execution, batching, timeouts and retries, verification, cleanup.
- Installable as a Claude Code plugin, a plain skill folder, or a zip upload to claude.ai.

Not yet run against a live production build.
