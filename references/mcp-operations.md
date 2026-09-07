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

Arbitrary Python execution inside the editor is privileged: full access to toolset APIs, the
asset database, the project on disk, and editor-privileged functions. It will succeed without
a second confirmation when approvals are off. Use it to batch known-good operations. Do not
use it to explore, and do not use it for anything destructive without asking first.

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
- Remove debug print nodes unless the user asked for them, in which case put them in a clearly
  marked debug comment box.

## If the server is not reachable

Symptoms: connection refused on 127.0.0.1:8000, or the server not listed by the client. Check,
in order: the editor is running; the ModelContextProtocol plugin is enabled; the server was
started (`ModelContextProtocol.StartServer` in the console, or auto-start configured); the
port is not taken by something else - it can be started on a different port. Report which
check failed rather than retrying the same call.
