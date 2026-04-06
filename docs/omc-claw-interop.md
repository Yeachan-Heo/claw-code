# OMC ↔ claw-code interop status

Last updated: 2026-04-06

## Exact runtime contract gaps

### 1) Slash command contract mismatch

- OMC exposes plugin/namespaced slash commands such as `/oh-my-claudecode:hud setup`.
- `claw` only parses the hard-coded slash registry in `rust/crates/commands/src/lib.rs` and rejects unknown namespaced commands in `rust/crates/rusty-claude-cli/src/main.rs`.
- Result: OMC command files are not callable through claw's slash dispatcher even when the OMC plugin is present.

### 2) Plugin manifest schema mismatch

- OMC's plugin manifest follows the Claude Code plugin contract (`skills`, `mcpServers`, Claude lifecycle hooks, directory/glob command catalogs).
- claw's plugin loader expects the Rust plugin contract in `rust/crates/plugins/src/lib.rs`: explicit executable `tools`, explicit executable `commands`, and only `PreToolUse` / `PostToolUse` / `PostToolUseFailure` hooks.
- Result: installing OMC as a claw plugin cannot provide OMC slash commands, hook lifecycle behavior, bundled skills, or MCP registrations.

### 3) Session lifecycle mismatch

- OMC relies on Claude lifecycle hooks and per-session state under `.omc/state/sessions/{sessionId}/`, plus session-end artifacts under `.omc/sessions/*.json`.
- claw persists conversation sessions as `.claw/sessions/*.jsonl` and its status/resume flow is local CLI state, not Claude hook/session lifecycle emulation.
- Result: OMC stop/session-start/session-end/session-recovery behaviors do not have the runtime events they need.

### 4) Status/HUD mismatch

- OMC's HUD contract expects `/oh-my-claudecode:hud`, `omc hud`, and OMC replay/session artifacts.
- claw currently has status snapshots (`/status`, `claw status`) but no OMC-compatible HUD/statusline runtime; `rust/TUI-ENHANCEMENT-PLAN.md` still lists “No status bar / HUD” as an open gap.
- Result: OMC observability/statusline features cannot attach to claw.

### 5) Skills interoperability is only partial

- claw already reads OMC-compatible skill roots (`.omc/skills`, `.agents/skills`, `~/.omc/skills`, `~/.claude/skills/omc-learned`).
- But OMC plugin-managed `skills` manifests are still unsupported because claw does not import plugin-declared skill catalogs.
- Result: local/shared OMC skill directories can interoperate, but installing OMC as a plugin does not make its bundled skills available through claw.

## Practical operator guidance

- **Works today:** direct local skill discovery from OMC-compatible skill roots.
- **Does not work today:** OMC plugin slash commands, OMC hook lifecycle, OMC HUD/statusline, OMC session-end/session-start runtime behavior.
- **Fastest failure signal:** if you try to install or use an OMC Claude Code plugin manifest, claw should now report that the manifest uses the Claude Code contract rather than failing ambiguously.
