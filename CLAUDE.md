# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project Overview

**GitHub Spec Kit** is a comprehensive toolkit for implementing Spec-Driven Development (SDD) - a methodology that emphasizes creating clear specifications before implementation. **Specify CLI** is the command-line interface that bootstraps projects with the Spec Kit framework.

## Build, Test, and Development Commands

```bash
# Install dependencies
uv sync

# Run CLI help
uv run specify --help

# Run tests
uv run pytest

# Run specific test with verbose output
uv run pytest tests/test_file.py::test_name -v

# Run tests with coverage
uv run pytest --cov

# Run linting (if configured)
uv run ruff check src/
```

## High-Level Architecture

### AGENT_CONFIG - Single Source of Truth

The `AGENT_CONFIG` dictionary in `src/specify_cli/__init__.py` is the **single source of truth** for all AI agent metadata. This pattern eliminates the need for special-case mappings throughout the codebase.

**Critical Design Principle**: Use the **actual CLI tool name** as the dictionary key, not a shortened version.

```python
# ✅ CORRECT - matches actual executable name
"cursor-agent": {
    "name": "Cursor",
    "folder": ".cursor/",
    "commands_subdir": "commands",
    "install_url": "https://cursor.com/",
    "requires_cli": True,
}

# ❌ WRONG - requires special-case mapping
"cursor": { ... }  # Don't do this!
```

**Field Definitions**:
- `name`: Human-readable display name
- `folder`: Directory for agent-specific files (e.g., `.claude/`)
- `commands_subdir`: Subdirectory name within the folder (default: `"commands"`)
  - Common values: `"commands"`, `"agents"` (copilot), `"workflows"` (windsurf, kilocode), `"prompts"` (codex, kiro), `"command"` (opencode - singular)
- `install_url`: Installation documentation URL (`None` for IDE-based agents)
- `requires_cli`: `True` for CLI tools, `False` for IDE-based agents

### Extension System

The extension system allows modular additions to Specify CLI:
- **Official catalog**: `extensions/catalog.json`
- **Community catalog**: `extensions/catalog.community.json`
- Extensions add new functionality without bloating core code
- See `extensions/README.md` for details

### Template and Command System

- **Templates**: `templates/` directory contains markdown templates for commands
- **Command formats**: Markdown (Claude, Cursor, Copilot) or TOML (Gemini, Qwen, Tabnine)
- **Argument placeholders**:
  - Markdown: `$ARGUMENTS`
  - TOML: `{{args}}`
  - Script: `{SCRIPT}` (replaced with actual script path)

## Supported AI Agents

Specify CLI supports 20+ AI coding agents including:

**CLI-Based Agents** (require CLI tool check):
- Claude Code (`claude`), Gemini CLI (`gemini`), Cursor (`cursor-agent`)
- Qwen Code (`qwen`), opencode (`opencode`), Codex CLI (`codex`)
- Kiro CLI (`kiro-cli`), Amp (`amp`), SHAI (`shai`), Tabnine CLI (`tabnine`)

**IDE-Based Agents** (no CLI check needed):
- GitHub Copilot (`.github/agents/`), Windsurf (`.windsurf/workflows/`)
- Kilo Code (`.kilocode/rules/`), IBM Bob (`.bob/commands/`)

## Development Workflow

1. Test changes with specify CLI commands in your coding agent of choice
2. Verify templates in `templates/` directory
3. Test script functionality in `scripts/` directory
4. Ensure memory files (`memory/constitution.md`) are updated for major process changes

### Testing Template/Command Changes Locally

Since `uv run specify init` pulls released packages, test local changes:

```bash
# 1. Create release packages
./.github/workflows/scripts/create-release-packages.sh v1.0.0

# 2. Copy to test project
cp -r .genreleases/sdd-copilot-package-sh/. <path-to-test-project>/

# 3. Open and test in the agent
```

## Important Practices

### Version Management
- Any changes to `__init__.py` require a version rev in `pyproject.toml`
- Add entries to `CHANGELOG.md` for all changes

### When Adding New Agents

1. Add to `AGENT_CONFIG` in `src/specify_cli/__init__.py` (use actual CLI tool name as key!)
2. Update `--ai` parameter help text in `init()` command
3. Update `README.md` Supported AI Agents table
4. Update `.github/workflows/scripts/create-release-packages.sh`
5. Update `scripts/bash/update-agent-context.sh` and `scripts/powershell/update-agent-context.ps1`
6. Update `.devcontainer/devcontainer.json` and `.devcontainer/post-create.sh` if needed

### Code Style
- Follow existing code conventions in the repository
- Keep changes focused - avoid "improvements" beyond what was requested
- Don't add docstrings, comments, or type annotations to unchanged code

## AI Contribution Disclosure

If using AI assistance to contribute to Spec Kit, **disclose this in the pull request or issue**. Include:
- Extent of AI use (documentation comments vs. code generation)
- Confirmation that you personally tested and understand the changes

Trivial typo/spacing fixes don't need disclosure.

## Key Files

| File | Purpose |
|------|---------|
| `src/specify_cli/__init__.py` | Main CLI code + AGENT_CONFIG |
| `pyproject.toml` | Package configuration (v0.2.0, Python 3.11+) |
| `AGENTS.md` | Detailed guide for adding new AI agents |
| `CONTRIBUTING.md` | Development workflow and contribution guidelines |
| `README.md` | Complete CLI reference and user documentation |
| `templates/commands/` | Command templates (specify, plan, tasks, implement, etc.) |

## Slash Commands Available After `specify init`

| Command | Description |
|---------|-------------|
| `/speckit.constitution` | Create project governing principles |
| `/speckit.specify` | Define requirements and user stories |
| `/speckit.plan` | Create technical implementation plan |
| `/speckit.tasks` | Generate actionable task breakdown |
| `/speckit.implement` | Execute tasks to build the feature |
| `/speckit.clarify` | Clarify underspecified areas |
| `/speckit.analyze` | Cross-artifact consistency analysis |
| `/speckit.checklist` | Generate quality validation checklists |
