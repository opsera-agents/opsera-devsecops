# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Claude Code plugin** (marketplace agent) that exposes 4 Opsera DevSecOps tools via the Opsera MCP server (`https://agent.opsera.ai/mcp`). Authentication is OAuth-based (no API tokens).

## Plugin Architecture

This is a **content-only plugin** — no build, lint, or test steps. All logic lives in the Opsera MCP server; the plugin provides agent definitions, skills, and commands that orchestrate MCP tool calls.

```
.claude-plugin/plugin.json   → Plugin manifest (name, version, author, keywords, component paths)
.mcp.json                    → Opsera MCP server config (HTTP + OAuth)
agents/devsecops.md          → Main agent: system prompt, tool routing, telemetry rules
skills/*/SKILL.md            → 5 model-invoked skills (auto-triggered by context)
commands/*.md                → 4 user-invoked slash commands (/architecture-analyze, etc.)
hooks/pre-commit-scan.sh     → PreToolUse hook: gates git commits behind security scan
settings.json                → Pre-approved tool permissions + hook configuration
```

## Key Conventions
- **Telemetry is mandatory**: Every tool execution MUST be followed by a `report-telemetry` call. This is enforced in `agents/devsecops.md` and each skill's SKILL.md.
- **Phased execution**: Architecture analysis, security scan, and compliance audit use multi-pass phased execution (`_execution_id` / `_phase_result` continuation). Skills document the exact phase flow.
- **Skills vs Commands**: Each tool has both a skill (Claude auto-invokes based on context) and a command (user triggers via `/command-name`). Keep these in sync when adding new tools.
- **Free trial scope**: v1.0.0 includes 4 tools. Future versions add more incrementally. The plugin.json version tracks this.

## Adding a New Tool

1. Create `skills/<tool-name>/SKILL.md` with frontmatter (`name`, `description`) and execution steps
2. Create `commands/<tool-name>.md` with user-facing instructions
3. Add the MCP tool name to `settings.json` permissions allow-list
4. Update `agents/devsecops.md` to include the new tool in the capabilities section and routing logic
5. Bump version in `.claude-plugin/plugin.json` and add entry to `CHANGELOG.md`

## MCP Server Reference

The Opsera MCP server at `https://agent.opsera.ai/mcp` provides all tools. The 4 currently used:
- `mcp__opsera__architecture-analyze` — multi-pass architecture risk analysis
- `mcp__opsera__security-scan` — phased security vulnerability scanning
- `mcp__opsera__compliance-audit` — multi-pass compliance framework auditing
- `mcp__opsera__sql-security` — SQL vulnerability scan/fix/PII/compliance/privileges
- `mcp__opsera__report-telemetry` / `mcp__opsera__opsera_report_telemetry` — telemetry (called after every execution)
