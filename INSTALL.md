# Install

Pick one route. They all end with the same thing: Claude reads `SKILL.md` before it touches
your Unreal project.

- [Route A — Claude Code plugin](#route-a--claude-code-plugin-recommended) (recommended, one command, updates cleanly)
- [Route B — plain skill folder, personal](#route-b--plain-skill-folder-personal) (every project on your machine)
- [Route C — plain skill folder, project-level](#route-c--plain-skill-folder-project-level) (travels with the Unreal project, for teams)
- [Route D — claude.ai and the desktop app](#route-d--claudeai-and-the-desktop-app) (zip upload)

Then do [Step 5 — check it loaded](#step-5--check-it-loaded) and
[Step 6 — connect Unreal](#step-6--connect-unreal-to-mcp).

---

## Route A — Claude Code plugin (recommended)

**Step 1.** Open Claude Code in any project.

**Step 2.** Add this repo as a plugin marketplace. Type this into the Claude Code prompt (it
is a slash command, not a shell command):

```
/plugin marketplace add mihirbathani/unreal-mcp-discipline.skill
```

**Step 3.** Install the plugin from that marketplace:

```
/plugin install unreal-mcp-discipline@unreal-mcp-discipline
```

The `name@marketplace` form looks odd because the plugin and the marketplace share a name.
That is correct — copy it exactly.

**Step 4.** Restart Claude Code.

**To update later:**

```
/plugin marketplace update unreal-mcp-discipline
```

**To remove:**

```
/plugin uninstall unreal-mcp-discipline
```

---

## Route B — plain skill folder, personal

Installs it for every project on your machine. No plugin system involved — Claude Code reads
any folder under `~/.claude/skills/`.

**macOS / Linux:**

```bash
git clone https://github.com/mihirbathani/unreal-mcp-discipline.skill.git ~/.claude/skills/unreal-mcp-discipline
```

**Windows (PowerShell):**

```powershell
git clone https://github.com/mihirbathani/unreal-mcp-discipline.skill.git "$env:USERPROFILE\.claude\skills\unreal-mcp-discipline"
```

The target folder name matters more than the repo name — keep it `unreal-mcp-discipline`.

Restart Claude Code.

**To update:** `git pull` inside that folder.

---

## Route C — plain skill folder, project-level

Use this when the skill should travel with the Unreal project, so everyone who clones the
project gets it. Run from your **Unreal project root** (the folder with the `.uproject` file).

**macOS / Linux:**

```bash
mkdir -p .claude/skills
git clone https://github.com/mihirbathani/unreal-mcp-discipline.skill.git .claude/skills/unreal-mcp-discipline
```

**Windows (PowerShell):**

```powershell
New-Item -ItemType Directory -Force .claude\skills
git clone https://github.com/mihirbathani/unreal-mcp-discipline.skill.git .claude\skills\unreal-mcp-discipline
```

Then commit `.claude/skills/unreal-mcp-discipline` to your project's repo. Delete the nested
`.git` folder first if you don't want a submodule-shaped mess:

```bash
rm -rf .claude/skills/unreal-mcp-discipline/.git
git add .claude/skills/unreal-mcp-discipline
git commit -m "Add unreal-mcp-discipline skill"
```

Restart Claude Code.

---

## Route D — claude.ai and the desktop app

**Step 1.** Build a zip of this repo. Download it from the
[repo's Code → Download ZIP](https://github.com/mihirbathani/unreal-mcp-discipline.skill/archive/refs/heads/main.zip),
or make one yourself:

```bash
git clone https://github.com/mihirbathani/unreal-mcp-discipline.skill.git unreal-mcp-discipline
cd unreal-mcp-discipline && rm -rf .git
cd .. && zip -r unreal-mcp-discipline.zip unreal-mcp-discipline
```

The zip must contain a single folder with `SKILL.md` at its top level.

**Step 2.** In Claude, open **Settings → Capabilities** and turn on code execution and file
creation. Skills will not upload without them.

**Step 3.** Scroll to **Skills**, click **Upload skill**, select the zip.

**Step 4.** Toggle the skill on.

Team and Enterprise accounts use **Customize → Skills** instead, and an organization owner has
to enable Skills in **Organization settings** first.

---

## Step 5 — check it loaded

Ask Claude:

> What skills do I have?

`unreal-mcp-discipline` should be in the list. If it isn't:

| Symptom | Fix |
| --- | --- |
| Not listed after Route A | Restart Claude Code. Then `/plugin` to confirm it shows as installed. |
| Not listed after Route B or C | Check the path: `SKILL.md` must sit directly inside `.claude/skills/unreal-mcp-discipline/`, not in a nested folder from the clone. |
| Listed but never triggers | Name it explicitly: "use unreal-mcp-discipline for this". |
| Route D upload rejected | The zip has to contain one folder with `SKILL.md` at its top level, not the loose files. |

---

## Step 6 — connect Unreal to MCP

The skill governs how Claude behaves. It does not connect anything — your MCP server does.

**Step 1.** In Unreal Engine 5.8, open **Edit → Plugins**, search for
**ModelContextProtocol**, enable it, restart the editor.

**Step 2.** Start the server. In the editor console:

```
ModelContextProtocol.StartServer
```

Or configure auto-start in project settings.

**Step 3.** Point your MCP client at `http://127.0.0.1:8000/mcp`.

**Step 4.** Confirm the connection by asking Claude to list toolsets. It should call
`list_toolsets` and come back with actors, blueprints, materials, and friends.

**If the server is unreachable** (connection refused on 127.0.0.1:8000), check in this order:
the editor is running → the plugin is enabled → the server was actually started → nothing else
holds port 8000. More detail in
[references/mcp-operations.md](references/mcp-operations.md).

> The server has no authentication and binds to loopback. Any process running as the same user
> can talk to it. Do not expose the port, and do not run it on a shared machine.

**Using a community Unreal MCP server instead?** The skill still works, but the tool names in
`references/mcp-operations.md` are Epic's. Edit that one file to match your server.
`node-reduction.md` and `documentation.md` are server-agnostic.

---

Next: [USAGE.md](USAGE.md) — what to actually type once it's installed.
