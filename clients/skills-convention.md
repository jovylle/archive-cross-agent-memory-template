# Skills convention

Each AI client discovers skills from its own directory. Rather than duplicating skill files, keep one canonical copy and symlink it into every agent.

## Canonical location

```
~/.agents/skills/
  my-skill/
    SKILL.md
  another-skill/
    SKILL.md
    notes/
      ...
```

## Symlink into each agent

```sh
SKILL=my-skill

ln -s ~/.agents/skills/$SKILL ~/.claude/skills/$SKILL
ln -s ~/.agents/skills/$SKILL ~/.copilot/skills/$SKILL
ln -s ~/.agents/skills/$SKILL ~/.kiro/skills/$SKILL
ln -s ~/.agents/skills/$SKILL ~/.config/opencode/skills/$SKILL
ln -s ~/.agents/skills/$SKILL ~/.codex/skills/$SKILL
```

Restart each agent after creating the symlinks. Most clients scan their skills directory at startup.

## SKILL.md format

Each skill needs a `SKILL.md` at its root. The front-matter `name` and `description` control how agents discover and activate the skill:

```markdown
---
name: my-skill
description: One sentence. Mention the trigger phrase if there is one.
---

# My Skill

Procedures and references here.
```

Keep the description precise — agents use it to decide whether to load the skill for a given task.

## Why this pattern

- **Single source of truth.** Edit `~/.agents/skills/my-skill/SKILL.md` once; all clients see the update after restart.
- **No copies to keep in sync.** Symlinks are cheap; copies diverge.
- **Portable handoff.** The skill is at a stable absolute path that any agent or handoff note can reference regardless of which client created it.

## Gotcha: ECC engine and symlinked vault roots

The ECC memory engine accepts symlinked skill directories, but it **refuses a symlinked vault root** for `ECC_MEMORY_USER_ROOT`. Always set the vault path to the real resolved directory, not a symlink.
