# Shared memory (cross-agent template)

A reusable layout for **one local memory vault** shared by Claude Code, GitHub Copilot CLI, Kiro, and OpenCode. Each client uses the same vault; the actual memories stay on your machine. This repository contains **synthetic examples only**, not a hosted service or anyone's private vault.

The [ECC Memory Vault](https://github.com/affaan-m/ECC) supplies the `ecc.memory.v1` Markdown format, `ecc memory` CLI, and `ecc-memory-mcp` server. This template supplies client instructions and a linked-note example; it does not redistribute ECC runtime code.

## Start a local vault

Install Node.js and npm, then:

```sh
npm install -g ecc-universal@2.2.1
ecc memory init --scope user
ecc memory doctor --scope user
```

The default user vault is `~/.ecc/memory/`. You can relocate it anywhere by setting `ECC_MEMORY_USER_ROOT` to an absolute path in every client's MCP server environment. The engine refuses symlinked vault roots — use a real directory.

```sh
# Example: move the vault to a dedicated directory
mkdir -p ~/.shared-memory/memory
ecc memory init --scope user   # run once with ECC_MEMORY_USER_ROOT set in env
ECC_MEMORY_USER_ROOT=~/.shared-memory/memory ecc memory doctor --scope user
```

All clients must point at the same path via the env var, or they will each see a different vault.

User-scope recall is opt-in: CLI commands need `--scope user`, and MCP calls need `scopes: ["user"]` for search and `scope: "user"` for read. The MCP examples enable `ECC_MEMORY_ALLOW_USER_SCOPE=1` and use a distinct `ECC_MEMORY_HARNESS` for each client.

## Connect clients

**Claude Code (CLI-based):** Place [clients/claude/SKILL.md](clients/claude/SKILL.md) at `~/.claude/skills/shared-memory/SKILL.md`. Claude invokes it via the `ecc` CLI, not an MCP server.

**GitHub Copilot CLI:** Merge [clients/copilot/copilot-instructions.md](clients/copilot/copilot-instructions.md) into `~/.copilot/copilot-instructions.md`. Register the MCP server:

```sh
copilot mcp add shared-memory \
  --env ECC_MEMORY_HARNESS=copilot \
  --env ECC_MEMORY_ALLOW_USER_SCOPE=1 \
  --env ECC_MEMORY_USER_ROOT=/absolute/path/to/vault \
  -- ecc-memory-mcp
```

**Kiro IDE or CLI:** Merge [clients/kiro/mcp.json](clients/kiro/mcp.json) into `~/.kiro/settings/mcp.json`. Place [clients/kiro/cross-agent-memory.md](clients/kiro/cross-agent-memory.md) in `~/.kiro/steering/`.

**OpenCode:** Merge [clients/opencode/mcp-snippet.jsonc](clients/opencode/mcp-snippet.jsonc) into the `mcp.servers` object in `~/.config/opencode/opencode.jsonc`.

### Absolute path for the MCP executable

Many GUI-based IDEs (Kiro desktop, OpenCode, VS Code extensions) do not inherit the shell `PATH` that npm uses. If `ecc-memory-mcp` is not found, replace the `command` with the absolute paths to `node` and the MCP script:

```sh
# Find the paths on your machine
which node                          # e.g. /home/you/.local/share/fnm/.../node
npm root -g                         # e.g. /home/you/.local/share/fnm/.../lib/node_modules
# Then use:
#   command: /absolute/path/to/node
#   args:    ["/absolute/path/to/node_modules/ecc-universal/scripts/memory-mcp.mjs"]
```

See the OpenCode snippet for a concrete example of the array `command` form.

## Skills convention

Keep shared skills in a **canonical directory** (`~/.agents/skills/`) and symlink it into each agent's skills folder. This way a skill file lives in one place and all clients discover it automatically on restart.

```sh
# Canonical home
mkdir -p ~/.agents/skills/my-skill

# Symlink into each agent
ln -s ~/.agents/skills/my-skill ~/.claude/skills/my-skill
ln -s ~/.agents/skills/my-skill ~/.copilot/skills/my-skill
ln -s ~/.agents/skills/my-skill ~/.kiro/skills/my-skill
ln -s ~/.agents/skills/my-skill ~/.config/opencode/skills/my-skill
ln -s ~/.agents/skills/my-skill ~/.codex/skills/my-skill
```

Each agent discovers skills from its own directory; the canonical location keeps edits in sync without copying files. See [clients/skills-convention.md](clients/skills-convention.md) for the full pattern.

## Linked notes, loaded only when relevant

The [synthetic support overview](examples/vault/contexts/mem_example_support_hub.md) links to a [triage sub-note](examples/vault/contexts/mem_example_support_triage.md) by stable memory ID. It illustrates a tiny overview followed by a focused read, not a preloaded dump of every note. Example files are **not installed** into anyone's vault; create your own records with:

```sh
ecc memory save --scope user --source-harness <client> --target all \
  --kind context --title "<title>" --stdin <<'MEMORY'
<relevant context, evidence, and what still needs checking>
MEMORY
```

Use `--link <memory-id>` for related records. Search before saving to avoid duplicates. Search a focused topic only for a resumption, prior decision, recurring issue, or cross-agent handoff; read a few relevant IDs. Check with `ecc memory doctor --scope user` after adding linked records.

## Gotchas

- **Engine refuses symlinked vault roots.** The MCP engine requires a real directory for `ECC_MEMORY_USER_ROOT`; a symlink pointing to the vault will be rejected. Always set the env var to the real resolved path.
- **Project scope recreates `~/.ecc`.** Running any ECC command with project scope (the default when no scope is given) from your home directory will create `~/.ecc/memory/project/`. Use `--scope user` (CLI) or `scope: "user"` (MCP) explicitly for cross-agent shared memory.
- **Absolute MCP paths.** GUI IDEs may not inherit the npm `PATH`. If `ecc-memory-mcp` is not found, use the absolute `node` binary + script path in `command`/`args`. See the section above.
- **Agent restart required.** MCP config changes take effect only after restarting the agent. Verify the server loaded with a test search before assuming it works.
- **Copilot and OpenCode use different config keys.** Copilot's server uses `env`; OpenCode uses `environment`. Kiro and other JSON MCP configs use `env`. Check the client's own schema.
- **Claude's native auto memory is separate.** Claude Code has its own internal memory that does not sync with this vault. The Claude skill here uses the ECC CLI directly.
- **Never commit real vault files.** Do not commit `~/.ecc/memory/`, `~/.shared-memory/memory/`, or any relocated vault path. Add these to `.gitignore`. Only templates and placeholders belong in this repo.

## Privacy and boundaries

- Never commit credentials, customer data, private work artifacts, or raw transcripts.
- A public template is not a good place for real project notes. Keep private memories in the local vault; only deliberately reviewed material belongs in a repository.
- Tools must request user scope explicitly. Other clients may need permission to open files linked from a memory outside their workspace.

This repository is MIT-licensed. The ECC runtime is a separate dependency under its own license.
