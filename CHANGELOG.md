# Changelog

## 1.2.0, 2026-10-11

Three additions, all about not losing work.

- Automatic backups. Before the first write to any asset, its `.uasset` is copied to
  `Saved/AgentBackups/<timestamp>/`, keeping its path under Content. No prompt, no opt-in.
  `Saved/` is outside the content tree, so the copies never reach the Content Browser or a
  cooked build, and a standard Unreal `.gitignore` already excludes them. When the project is
  in git, the pre-session commit hash goes in the log as well. The restore procedure is
  documented, including the part people get wrong: close the editor first, or it writes the
  old package straight back over the restored file.
- A capability check before anything else. Every task sorts into one of three routes before
  the plan is written: MCP can do it, the user has to do it by hand, or computer use can
  drive the editor UI. Computer use has to be offered and accepted, never used as a silent
  fallback after a failed call. "I cannot do that part" is now an explicit acceptable answer.
- Two project docs instead of one vague "agent notes file". `Content/Docs/agent-log.md` for
  dated entries on what changed and why. `Content/Docs/blueprint-map.md` for what exists:
  one entry per Blueprint with its purpose, what fires it, what it talks to, and its gotcha.
  An existing project convention still wins over both.

Smaller: the closing report gains a backup line, and drops the orphan count when there were
none.

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
