# MCP operations against Unreal Editor 5.8

Verify everything here against the installed build before relying on it. Unreal MCP is
experimental in 5.8; Epic has said APIs and data formats may change, and a studio may be
running a different server (Epic's official plugin, or one of the community servers, which
have different tool names entirely). Schema first, memory second.

## The three layers

```
MCP client  ->  http://127.0.0.1:8000/mcp
                Unreal Editor
                  ModelContextProtocol   (server, connection, tool search)
                  Toolset Registry       (toolset discovery and schemas)
                  AllToolsets / toolsets (actors, blueprints, materials, ...)
```

Three plugins, not one: `ModelContextProtocol`, `ToolsetRegistry` and `AllToolsets`. Enabling
only the first gives a server with nothing useful registered on it.

Useful console commands:

```
ModelContextProtocol.StartServer
ModelContextProtocol.RefreshTools              re-register after enabling a toolset
ModelContextProtocol.GenerateClientConfig <client>   writes a client config block
```

`RefreshTools` is the fix for a toolset that was enabled after the server started and is not
showing up. Restarting the editor for that is wasted time.

The server has no authentication layer and binds to loopback. Any process running as the same
user can talk to it. Do not expose the port; do not run it on a shared machine.

## Tool search: three meta-tools

With tool search on (the default), the server exposes `list_toolsets`, `describe_toolset` and
`call_tool` rather than the full catalogue of hundreds of tools. The sequence is always:

1. `list_toolsets` - find which toolset owns the operation.
2. `describe_toolset` - get the exact tool names and argument schemas.
3. `call_tool` - invoke with validated arguments.

Skipping step 2 and guessing argument names is the main cause of retry loops. The schema is
cheap; a failed call plus a retry plus a confused recovery is not. Discovered tools are
callable on the same turn, so there is no reason to defer.

If tool search is disabled in project settings, every tool is registered upfront and the
names are directly visible - the read-before-write rule is unchanged.

## Serial execution

Tool calls run on Unreal's game thread. Issuing calls in parallel deadlocks the editor. One
call at a time, wait for the result, then the next. This is not a throughput preference; it
is a correctness requirement.

## Batching

Individual node-and-pin calls are the biggest source of wasted turns. A 40-node function built
one call at a time is 80+ round trips, each with schema overhead, and any failure halfway
leaves a partial graph.

The Scripting toolset exists for this: batch multiple operations into a single script
execution once the plan is settled. Rules:

- **Plan first, batch second.** Batching an unverified plan just produces the wrong graph
  faster and makes the failure harder to bisect.
- **Idempotent scripts.** Check whether the node, variable, asset or actor already exists
  before creating it. Then a retry after a timeout is safe.
- **One logical unit per batch.** One function, or one event and its helpers. Not the whole
  Blueprint. Compile between units so errors are attributable.
- **Return something checkable.** Have the script report what it created so the result can be
  verified without a second read pass.

Wrap each batch in a single undo transaction where the scripting layer allows it. In Python
that is `unreal.ScopedEditorTransaction`. One Ctrl+Z should then reverse one function's worth
of work rather than leaving the user to undo forty node creations one at a time. This is the
cheapest protection available against a graph the user does not want.

Arbitrary Python execution inside the editor is privileged: full access to toolset APIs, the
asset database, the project on disk, and editor-privileged functions. It will succeed without
a second confirmation when approvals are off. Use it to batch known-good operations. Do not
use it to explore, and do not use it for anything destructive without asking first.

## Known limits

- **Comment boxes cannot be created.** No toolset places a comment node in a Blueprint graph,
  and Blueprint graph editing is barely exposed to Python, so there is no scripting fallback
  either. Plan documentation around names, asset metadata and the ledger. See
  `documentation.md`.
- **Do not assume an auto-arrange tool.** Set node positions explicitly as you create them.
- **Verify capability per build.** This plugin is experimental in 5.8 and Epic has said APIs
  and data formats may change. `describe_toolset` is the authority on what exists today. Tool
  counts reported by third parties vary widely, so they say nothing about your install.

When something is not available, there are two fallbacks and they are not equivalent:

**Hand it to the user.** The default. Give exact steps or paste-ready text and carry on with
the parts that do work. Costs them seconds and cannot go wrong.

**Computer use, driving the editor UI by screenshot and click.** Real, and the only route for
things with no tool at all, but slow, dependent on window layout, and capable of clicking the
wrong thing in a project with unsaved work. Offer it, say it is the fragile path, and get an
answer before starting. Never fall back to it silently because a tool call failed, and never
use it for anything destructive. The backup rule below matters more here than anywhere else.

## Timeouts and retries

A timeout means the response was not received, not that the operation failed. The classic
failure is a retry that duplicates an actor, asset, or node subgraph.

On any timeout or ambiguous error: re-read the state, then decide. Never blind-retry a
creation call.

## Verification

- Compile after each unit and read the actual compile result, including warnings.
- Re-read the graph after building - node count, connections, orphans.
- A viewport or graph screenshot is worth taking before claiming a visual result works.
- The Output Log is readable through the editor toolset. Read it when something behaves
  unexpectedly rather than guessing.
- "The call returned success" is not verification that the logic is correct. It means the
  editor accepted the mutation.

## Backups

Before the first mutation of any asset in a session, copy its `.uasset` out of the project.
This is automatic and unprompted. Do not ask, do not offer, do not skip it because the change
looks small.

```
Content/Blueprints/BP_Scoreboard.uasset
  ->  Saved/AgentBackups/2026-10-11-1432/Blueprints/BP_Scoreboard.uasset
```

`Saved/` is the right home for it: outside the content tree, so the copies never appear in the
Content Browser and never cook into a build, and already ignored by a standard Unreal
`.gitignore`, so they cannot pollute a commit. Keep the path under Content intact inside the
backup folder, so a restore is a straight copy back rather than a guess about where the file
came from.

One folder per session, timestamped. Record the folder in the ledger and name it once in the
closing report.

If the project is in git, also record the current commit hash in the ledger. The disk copy
restores one asset; the hash tells the user what everything looked like before the session. A
project with no git is not a reason to skip the disk copy.

**Restoring.** Close the editor first. An open editor holds the package in memory and will
write it back over the restored file on the next save. Then copy the file back to its original
path and reopen. Tell the user this when you tell them where the backup is, because the
obvious move, copying it back with the editor still running, silently does nothing.

## Saving and source control

- Save before a bulk change and after it.
- Confirm the working copy is committed or shelved before a long MCP session. Tools edit live
  UObject state and can mutate, move, or delete tracked assets in a single call.
- Review the diff before submitting. A Blueprint diff is readable in the editor's asset diff
  tool; use it rather than asserting the change is small.

## Cleanup before finishing

- Delete every node this session created that ended up disconnected.
- Delete abandoned intermediate assets - the test Blueprint created to check something, the
  duplicate that a retry produced.
- Arrange nodes so the graph reads left to right without overlaps.
- Remove debug print nodes unless the user asked for them. If they stay, name them so they are
  greppable and list them in the closing report, since there is no comment box to mark them
  with.

## If the server is not reachable

Symptoms: connection refused on 127.0.0.1:8000, or the server not listed by the client. Check,
in order: the editor is running; the ModelContextProtocol plugin is enabled; the server was
started (`ModelContextProtocol.StartServer` in the console, or auto-start configured); the
port is not taken by something else - it can be started on a different port. Report which
check failed rather than retrying the same call.
