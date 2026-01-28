# NLM Skill Installer Design

**Status:** ✅ **COMPLETED** (2026-01-28)

## Overview

Add `nlm skill` commands to install unified NotebookLM skills for multiple AI tools.

## Commands

```bash
nlm skill install <tool> [--level user|project]  # Install skill for AI tool
nlm skill list                                    # Show tools and install status
nlm skill uninstall <tool> [--level user|project] # Remove installed skill
nlm skill show                                    # Display skill content
```

## Supported Tools & Paths

| Tool | User Path | Project Path | Format |
|------|-----------|--------------|--------|
| claude-code | `~/.claude/skills/nlm-skill/` | `.claude/skills/nlm-skill/` | SKILL.md |
| codex | `~/.codex/AGENTS.md` (append) | `AGENTS.md` (append) | AGENTS.md section |
| opencode | `~/.config/opencode/skills/nlm-skill/` | `.opencode/skills/nlm-skill/` | SKILL.md |
| gemini-cli | `~/.gemini/skills/nlm-skill/` | `.gemini/skills/nlm-skill/` | SKILL.md |
| antigravity | `~/.gemini/antigravity/skills/nlm-skill/` | `.agent/skills/nlm-skill/` | SKILL.md |
| other | N/A | `./nlm-skill-export/nlm-skill/` | All formats |

Default level: `user`

**Note:** Folder name is `nlm-skill` to match the skill name in frontmatter.

## Skill Content (Unified)

Single skill covering both CLI and MCP:
- Triggers on NotebookLM-related requests
- Decision logic: "If MCP tools available, use them; otherwise use `nlm` CLI"
- Core workflows: auth, notebooks, sources, research, studio, chat
- Reference files in `references/` subdirectory

## Implementation

### Files to Create

1. **`src/notebooklm_tools/cli/commands/skill.py`** - CLI commands
2. **`src/notebooklm_tools/data/SKILL.md`** - Main skill file
3. **`src/notebooklm_tools/data/AGENTS_SECTION.md`** - Codex-specific content
4. **`src/notebooklm_tools/data/references/`** - Reference docs (command_reference.md, workflows.md, troubleshooting.md)

### Files to Modify

1. **`src/notebooklm_tools/cli/main.py`** - Register skill commands
2. **`pyproject.toml`** - Include data files in package

### Key Logic

```python
TOOL_CONFIGS = {
    "claude-code": {
        "user": Path.home() / ".claude/skills/nlm-skill",
        "project": Path(".claude/skills/nlm-skill"),
        "format": "skill.md",
    },
    "codex": {
        "user": Path.home() / ".codex/AGENTS.md",
        "project": Path("AGENTS.md"),
        "format": "agents.md",  # Append section with markers
    },
    "opencode": {
        "user": Path.home() / ".config/opencode/skills/nlm-skill",
        "project": Path(".opencode/skills/nlm-skill"),
        "format": "skill.md",
    },
    "gemini-cli": {
        "user": Path.home() / ".gemini/skills/nlm-skill",
        "project": Path(".gemini/skills/nlm-skill"),
        "format": "skill.md",
    },
    "antigravity": {
        "user": Path.home() / ".gemini/antigravity/skills/nlm-skill",
        "project": Path(".agent/skills/nlm-skill"),
        "format": "skill.md",
    },
    "other": {
        "project": Path("./nlm-skill-export"),
        "format": "all",
    },
}
```

For Codex: Append NLM section with markers (`<!-- nlm-skill-start -->` / `<!-- nlm-skill-end -->`).

For "other": Generate folder with all formats for manual copy.

## Implementation Notes

✅ **Completed Features:**
- Skill name changed from `nlm-cli-skill` to `nlm-skill` to reflect unified CLI/MCP coverage
- Updated existing CLI skill to include MCP tool examples alongside CLI commands
- Added tool detection logic: "Check for MCP tools first, fall back to CLI"
- Parent directory validation for user-level installs with 3 options (create/switch to project/cancel)
- Consistent `nlm-skill/` folder name across all installations and exports
- All 4 commands implemented: install, uninstall, list, show

✅ **All Files Created:**
- `src/notebooklm_tools/cli/commands/skill.py` (438 lines)
- `src/notebooklm_tools/data/SKILL.md` (unified CLI/MCP skill, ~700 lines)
- `src/notebooklm_tools/data/AGENTS_SECTION.md` (compact Codex format)
- `src/notebooklm_tools/data/references/` (copied from existing analysis)

✅ **All Files Modified:**
- `src/notebooklm_tools/cli/main.py` (added skill_app import and registration)
- `pyproject.toml` (added data files packaging)

## Verification Results

1. ✅ `nlm skill install claude-code --level project` → creates `.claude/skills/nlm-skill/`
2. ✅ `nlm skill install codex --level project` → appends section with markers to `AGENTS.md`
3. ✅ `nlm skill list` → shows installation status for all tools (user/project columns)
4. ✅ `nlm skill install other --level project` → creates `./nlm-skill-export/nlm-skill/`
5. ✅ `nlm skill uninstall codex --level project` → removes markers from `AGENTS.md`
6. ✅ `nlm skill show` → displays full skill content
7. ✅ Parent directory validation works with user-level installs
