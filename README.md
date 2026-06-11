# Anti-OOP Design Skill

Anti-OOP Design is a new business-aggregation-based design method for building clean, testable software architecture with AI agents.

Instead of starting from entities, class hierarchies, or preset layers, it first gathers related business rules, facts, computed results, and side effects into one view. The gathered rule is then split into pure computation and side-effect boundaries, so business logic can be tested with plain inputs and outputs while external code remains thin and focused.

This repository packages the method as ready-to-use skills and instruction files for Codex, Claude Code, and opencode, including a Claude Code marketplace plugin. It is designed to help humans and AI agents collaborate on module design, refactoring, and architecture evolution without scattering business rules across services, entities, repositories, and controllers.

Because only one platform version is installed at a time, every platform package keeps the same skill name:

```yaml
name: anti-oop-design
```

## Layout

```txt
.claude-plugin/
  marketplace.json

AGENTS.md
CLAUDE.md
README.md

codex/
  anti-oop-design/
    SKILL.md

claude/
  .claude-plugin/
    plugin.json
  skills/
    anti-oop-design/
      SKILL.md

opencode/
  anti-oop-design/
    SKILL.md
```

## Which Folder Should I Use?

- Use `codex/` for Codex or OpenAI Agent Skills.
- Use `claude/` for the Claude Code plugin package.
- Use `opencode/` for opencode local skills.

The skill content is intentionally almost identical across platforms. The separate folders make installation explicit and leave room for platform-specific changes later.

## Zero-Install Project Instructions

If you do not want to install a skill or plugin, copy one of the root instruction files into your project:

- `CLAUDE.md` for Claude Code
- `AGENTS.md` for Codex and other agents that read AGENTS.md

These files contain the same Anti-OOP Design workflow in a project-level instruction format.

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

## Install for Claude Code Through Marketplace

Add this repository as a Claude Code marketplace:

```txt
/plugin marketplace add cypress927/anti-oop-design-v2.SKILL
```

Install the plugin:

```txt
/plugin install anti-oop-design@anti-oop-design-skills
```

The bundled skill is namespaced by the plugin name:

```txt
/anti-oop-design:anti-oop-design
```

You can also use the non-interactive CLI form:

```powershell
claude plugin marketplace add cypress927/anti-oop-design-v2.SKILL
claude plugin install anti-oop-design@anti-oop-design-skills
```

## Test the Claude Plugin Locally

From the repository root:

```powershell
claude --plugin-dir .\claude
```

## Install for Claude Code as a Local Skill

If you do not want marketplace/plugin installation, copy the skill itself:

```powershell
Copy-Item -Recurse .\claude\skills\anti-oop-design $env:USERPROFILE\.claude\skills\
```

This local-skill install is separate from the marketplace plugin flow.

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

Codex and opencode use the shared local skill shape:

```txt
skill-name/
  SKILL.md
```

Claude Code uses a plugin package:

```txt
claude/
  .claude-plugin/plugin.json
  skills/anti-oop-design/SKILL.md
```

The repository root also contains `.claude-plugin/marketplace.json`, which lets users add this GitHub repository as a Claude Code marketplace and install the plugin from it.

## References

- Codex Agent Skills: https://developers.openai.com/codex/skills
- Claude Code Skills: https://code.claude.com/docs/en/skills
- Claude Code Plugins: https://code.claude.com/docs/en/plugins
