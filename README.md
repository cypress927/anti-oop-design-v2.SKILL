# Anti-OOP Design Skill

This repository provides the same Anti-OOP Design skill packaged for different agent platforms.

Because only one platform version is installed at a time, every platform package keeps the same skill name:

```yaml
name: anti-oop-design
```

## Layout

```txt
codex/
  anti-oop-design/
    SKILL.md

claude/
  anti-oop-design/
    SKILL.md

opencode/
  anti-oop-design/
    SKILL.md
```

## Which Folder Should I Use?

- Use `codex/` for Codex or OpenAI Agent Skills.
- Use `claude/` for Claude Code local skills.
- Use `opencode/` for opencode local skills.

The skill content is intentionally almost identical across platforms. The separate folders make installation explicit and leave room for platform-specific changes later.

## Install for Codex

Copy the Codex skill folder into your Codex skills directory:

```powershell
Copy-Item -Recurse .\codex\anti-oop-design $env:USERPROFILE\.codex\skills\
```

Expected result:

```txt
%USERPROFILE%\.codex\skills\anti-oop-design\SKILL.md
```

Restart Codex after copying.

## Install for Claude Code

Copy the Claude skill folder into your Claude Code skills directory:

```powershell
Copy-Item -Recurse .\claude\anti-oop-design $env:USERPROFILE\.claude\skills\
```

Expected result:

```txt
%USERPROFILE%\.claude\skills\anti-oop-design\SKILL.md
```

Restart Claude Code after copying.

## Install for opencode

For a project-level install, copy the opencode skill folder into the project's `.opencode/skills` directory:

```powershell
New-Item -ItemType Directory -Force .\.opencode\skills | Out-Null
Copy-Item -Recurse .\opencode\anti-oop-design .\.opencode\skills\
```

Expected result:

```txt
.opencode/skills/anti-oop-design/SKILL.md
```

Restart or reload opencode after copying.

## Format Notes

This repository uses the shared local skill shape used by Codex, Claude Code, and opencode:

```txt
skill-name/
  SKILL.md
```

Each `SKILL.md` contains YAML frontmatter with `name` and `description`, followed by Markdown instructions.

This is not a Claude Code marketplace plugin package. A Claude plugin package would add a `.claude-plugin/plugin.json` manifest around the skill.

## References

- Codex Agent Skills: https://developers.openai.com/codex/skills
- Claude Code Skills: https://code.claude.com/docs/en/skills
- Claude Code Plugins: https://code.claude.com/docs/en/plugins
