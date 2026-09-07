# Install

Pick one route. They all end the same way: Claude reads `SKILL.md` before it touches your
Unreal project.

- [Route A: Claude Code plugin](#route-a-claude-code-plugin-recommended). Recommended. Two
  commands, and updates are one more.
- [Route B: plain skill folder, personal](#route-b-plain-skill-folder-personal). Every project
  on your machine.
- [Route C: plain skill folder, project-level](#route-c-plain-skill-folder-project-level).
  Travels with the Unreal project, so everyone who clones it gets the skill.
- [Route D: claude.ai and the desktop app](#route-d-claudeai-and-the-desktop-app). Zip upload.

Then do [step 5](#5-check-it-loaded) and [step 6](#6-connect-unreal-to-mcp).

---

## Route A: Claude Code plugin (recommended)

1. Open Claude Code in any project.

2. Add this repo as a plugin marketplace. Type this at the Claude Code prompt. It is a slash
   command, not a shell command:

   ```
   /plugin marketplace add mihirbathani/unreal-mcp-discipline.skill
   ```

3. Install the plugin from that marketplace:

   ```
   /plugin install unreal-mcp-discipline@unreal-mcp-discipline
   ```

   The `name@marketplace` form looks like a typo because the plugin and the marketplace share
   a name. It is correct. Copy it exactly.

4. Restart Claude Code.

To update later: `/plugin marketplace update unreal-mcp-discipline`.

To remove it: `/plugin uninstall unreal-mcp-discipline`.

---

## Route B: plain skill folder, personal

This installs it for every project on your machine, with no plugin system involved. Claude
Code reads any folder under `~/.claude/skills/`.

macOS and Linux:

```bash
git clone https://github.com/mihirbathani/unreal-mcp-discipline.skill.git ~/.claude/skills/unreal-mcp-discipline
```

Windows, in PowerShell:

```powershell
git clone https://github.com/mihirbathani/unreal-mcp-discipline.skill.git "$env:USERPROFILE\.claude\skills\unreal-mcp-discipline"
```

The target folder name matters more than the repo name. Keep it `unreal-mcp-discipline`.

Restart Claude Code. To update, run `git pull` inside that folder.

---

## Route C: plain skill folder, project-level

Use this when the skill should travel with the Unreal project. Run it from your Unreal project
root, the folder holding the `.uproject` file.

macOS and Linux:

```bash
mkdir -p .claude/skills
git clone https://github.com/mihirbathani/unreal-mcp-discipline.skill.git .claude/skills/unreal-mcp-discipline
```

Windows, in PowerShell:

```powershell
New-Item -ItemType Directory -Force .claude\skills
git clone https://github.com/mihirbathani/unreal-mcp-discipline.skill.git .claude\skills\unreal-mcp-discipline
```

Then commit the folder to your project's repo. Delete the nested `.git` directory first unless
you want it behaving like a submodule:

```bash
rm -rf .claude/skills/unreal-mcp-discipline/.git
git add .claude/skills/unreal-mcp-discipline
git commit -m "Add unreal-mcp-discipline skill"
```

Restart Claude Code.

---

## Route D: claude.ai and the desktop app

1. Get a zip of this repo. Either use
   [Code, then Download ZIP](https://github.com/mihirbathani/unreal-mcp-discipline.skill/archive/refs/heads/main.zip),
   or build one:

   ```bash
   git clone https://github.com/mihirbathani/unreal-mcp-discipline.skill.git unreal-mcp-discipline
   rm -rf unreal-mcp-discipline/.git
   zip -r unreal-mcp-discipline.zip unreal-mcp-discipline
   ```

   The zip has to contain a single folder with `SKILL.md` at its top level.

2. In Claude, open Settings, then Capabilities, and turn on code execution and file creation.
   Skills will not upload without them.

3. Scroll to Skills, click Upload skill, and select the zip.

4. Toggle the skill on.

Team and Enterprise accounts use Customize, then Skills instead, and an organization owner has
to enable Skills in Organization settings first.

---

## 5. Check it loaded

Ask Claude:

> What skills do I have?

`unreal-mcp-discipline` should be in the list. If it is not:

| What you see | What to do |
| --- | --- |
| Missing after route A | Restart Claude Code, then run `/plugin` to confirm it shows as installed. |
| Missing after route B or C | Check the path. `SKILL.md` has to sit directly inside `.claude/skills/unreal-mcp-discipline/`, not one folder deeper from the clone. |
| Listed but never triggers | Name it: "use unreal-mcp-discipline for this". |
| Route D upload rejected | The zip needs one folder with `SKILL.md` at its top level, not the loose files. |

---

## 6. Connect Unreal to MCP

The skill governs how Claude behaves. It does not connect anything. Your MCP server does that.

1. In Unreal Engine 5.8, open Edit, then Plugins. Search for ModelContextProtocol, enable it,
   and restart the editor.

2. Start the server from the editor console:

   ```
   ModelContextProtocol.StartServer
   ```

   Or configure auto-start in project settings.

3. Point your MCP client at `http://127.0.0.1:8000/mcp`.

4. Confirm the connection by asking Claude to list toolsets. It should call `list_toolsets` and
   come back with actors, blueprints, materials, and the rest.

If the server is unreachable and you get connection refused on 127.0.0.1:8000, check in this
order: the editor is running, the plugin is enabled, the server was actually started, and
nothing else holds port 8000. There is more detail in
[references/mcp-operations.md](references/mcp-operations.md).

> The server has no authentication and binds to loopback. Any process running as the same user
> can talk to it. Do not expose the port, and do not run it on a shared machine.

Running a community Unreal MCP server instead? The skill still works, but the tool names in
`references/mcp-operations.md` are Epic's. Edit that one file to match your server.
`node-reduction.md` and `documentation.md` are server-agnostic.

---

Next: [USAGE.md](USAGE.md), for what to type once it is installed.
