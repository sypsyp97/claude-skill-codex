# Codex Skill for Claude Code

> Teach Claude Code how to invoke OpenAI's Codex CLI for advanced code analysis, refactoring, and automated editing.

A [Claude Code skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) that enables Claude to leverage OpenAI Codex CLI's advanced reasoning models for code reviews, refactoring, security audits, architecture analysis, and automated transformations.

![Demo](demo.png)

## Overview

This filesystem-based skill automatically activates when you mention Codex-related tasks. Simply ask Claude to review or refactor your code, and it will:

- Select optimal model (`gpt-5` vs `gpt-5-codex`) and reasoning effort
- Execute Codex CLI with appropriate parameters and sandbox settings
- Handle session resume, error recovery, and multi-step workflows
- Summarize results and suggest next steps

## Prerequisites

Before installing this skill:

- **Codex CLI installed**: The `codex` command must be available on your `PATH`
- **Codex configured**: Valid credentials in `~/.codex/config.toml`
- **Verify installation**: Run `codex --version` to confirm setup

If you don't have Codex installed, see the [Codex CLI documentation](https://developers.openai.com/codex/cli/reference).

## Installation

Clone and copy the skill to your `.claude/skills` directory:

**Personal Installation (all projects):**
```bash
# Unix/Linux/macOS
git clone --depth 1 https://github.com/sypsyp97/claude-skill-codex.git /tmp/codex-skill && \
mkdir -p ~/.claude/skills && cp -r /tmp/codex-skill/.claude/skills/codex ~/.claude/skills/codex && \
rm -rf /tmp/codex-skill

# Windows (PowerShell)
git clone --depth 1 https://github.com/sypsyp97/claude-skill-codex.git $env:TEMP\codex-skill; `
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills"; `
Copy-Item -Recurse -Force "$env:TEMP\codex-skill\.claude\skills\codex" "$env:USERPROFILE\.claude\skills\codex"; `
Remove-Item -Recurse -Force "$env:TEMP\codex-skill"
```

**Project-Specific Installation:**
```bash
# Unix/Linux/macOS
git clone --depth 1 https://github.com/sypsyp97/claude-skill-codex.git /tmp/codex-skill && \
mkdir -p .claude/skills && cp -r /tmp/codex-skill/.claude/skills/codex .claude/skills/codex && \
rm -rf /tmp/codex-skill

# Windows (PowerShell)
git clone --depth 1 https://github.com/sypsyp97/claude-skill-codex.git $env:TEMP\codex-skill; `
New-Item -ItemType Directory -Force -Path ".claude\skills"; `
Copy-Item -Recurse -Force "$env:TEMP\codex-skill\.claude\skills\codex" ".claude\skills\codex"; `
Remove-Item -Recurse -Force "$env:TEMP\codex-skill"
```

Verify: Check that `~/.claude/skills/codex/SKILL.md` or `.claude/skills/codex/SKILL.md` exists. Claude Code will automatically discover it.

## Usage

Simply mention Codex in your conversation with Claude Code:

```bash
Review my Python code for security issues using codex
Use codex to refactor this module to async/await
Analyze the architecture of this codebase with codex
Resume the last codex session and apply the suggested changes
```

Claude will automatically activate the skill, ask for preferences (model and reasoning effort), and execute the appropriate command.

**Example:**
```bash
codex exec -m gpt-5-codex \
  --config model_reasoning_effort="high" \
  --sandbox read-only \
  --full-auto \
  "Analyze this repository..." 2>/dev/null
```

> **Note:** Thinking tokens (stderr) are suppressed by default. To see them for debugging, ask: "Show me the thinking tokens from codex"

## Configuration Reference

### Model Selection Guide

| Task Type | Recommended Model | Reasoning Effort |
|-----------|------------------|------------------|
| Quick syntax fixes | `gpt-5` | minimal |
| Code review | `gpt-5` | medium |
| Refactoring | `gpt-5-codex` | medium |
| Architecture analysis | `gpt-5` | high |
| Security audit | `gpt-5` | high |
| Algorithm optimization | `gpt-5-codex` | high |
| Documentation generation | `gpt-5` | low |

### Sandbox Modes

- **read-only**: Analysis and documentation (default)
- **workspace-write**: Code modifications and file creation
- **danger-full-access**: System operations (rarely needed)

For complete details including workflows, CLI reference, error handling, and best practices, see [SKILL.md](./.claude/skills/codex/SKILL.md).

## Resources

- [SKILL.md](./.claude/skills/codex/SKILL.md) - Complete documentation with workflows and CLI reference
- [GitHub Issues](https://github.com/sypsyp97/claude-skill-codex/issues) - Report bugs or request features
- [Claude Code Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) - How skills work
- [Codex CLI Reference](https://developers.openai.com/codex/cli/reference) - Codex documentation

## Contributing

Contributions welcome! Fork the repository, edit `.claude/skills/codex/SKILL.md`, test locally, and submit a pull request.

## License

See [LICENSE](./LICENSE) for details.

## Acknowledgments

Inspired by [skills-directory/skill-codex](https://github.com/skills-directory/skill-codex.git) with extensive enhancements for production use, including comprehensive CLI documentation, multi-platform support, workflow orchestration, advanced error handling, and detailed troubleshooting guides.
