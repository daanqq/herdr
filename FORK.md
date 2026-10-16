# Fork changes

This checkout is the `daanqq/herdr` fork (`origin`) of the canonical Herdr
repository (`upstream`). This file is the single inventory of behavior the fork
adds on top of `upstream/master`. Read it before every upstream sync, and keep
it current: add an entry when a fork feature lands, update it when a sync adapts
one, and delete the entry when the fork drops a feature in favor of upstream.

List fork-only commits with:

```bash
git log --oneline --no-merges $(git merge-base HEAD upstream/master)..HEAD
```

## Overlap protocol

An **overlap** is any upstream change that touches the same behavior as a fork
feature: the same config key or UI gesture, the same code path, a replacement
of the structure a fork feature hooks into, or an upstream feature that solves
the same problem differently. A textual merge conflict is one symptom; a clean
merge can still hide an overlap, so check each entry's overlap signals against
the incoming upstream commits, not only the conflicted files.

For every overlap, stop before resolving it and ask the user. Name the fork
feature, the upstream commit or PR, and what each side does, then offer the
options that actually apply:

1. **Keep fork**: preserve fork behavior, adapting it to the new upstream
   interface.
2. **Adopt upstream**: take upstream behavior, drop the fork feature, and
   delete its entry here (migrate config keys or note the break).
3. **Combine**: keep both when they compose, for example the fork setting as an
   option on top of the upstream mechanism.

Recommend one option with a reason. Mechanical conflicts that only move or
rename code around unchanged fork behavior do not need a question.

## Features

### Sidebar agent close menu

Commit `2023d305`. Right-clicking an agent in the sidebar opens a context menu
with a single **Close** item that closes the agent's whole tab through the
existing tab close confirmation, without changing the focused workspace, tab,
or pane. For an agent on another connected machine the client activates that
endpoint first and opens the menu once its snapshot arrives
(`pending_agent_context_menu`). TUI/client state only; no protocol change.

- Code: `src/client/shell/context_menu.rs` (`ClientContextMenuTarget::AgentTab`,
  `open_agent_context_menu`, `show_agent_context_menu`), right-click routing in
  `src/client/shell/mouse.rs`, pending-menu replay in
  `src/client/shell/endpoints.rs`, pending-menu reset in
  `src/client/shell/endpoint_navigation.rs` and `src/client/shell/state.rs`.
- Tests: `src/client/shell/tests/close_tab.rs`,
  `src/client/shell/tests/endpoints.rs`, and
  `api_tab_close_background_target_preserves_focused_workspace_tab_and_pane` in
  `src/app/api/tabs.rs` (pins the upstream API guarantee the feature relies on).
- Docs: `docs/next/website/src/content/docs/{,ja/,zh-cn/}agents.mdx`.
- Overlap signals: upstream adds its own sidebar agent context menu or
  right-click action; changes `ClientContextMenuTarget`, sidebar hit-testing
  (`hits.agents`, `hits.endpoint_agents`), `pending_agent_reveal`, or
  `tab.close` focus behavior.

### Alt-wheel scroll speed

Commit `4ef8dd9a`. New config key `ui.alt_mouse_scroll_lines` (default 15,
`DEFAULT_ALT_MOUSE_SCROLL_LINES`) sets lines per wheel notch while Alt is held,
in the shell client and in Unix direct attach clients. Live config reload
applies it.

- Code: `src/config.rs`, `src/config/model.rs` (`UiConfig`),
  `src/client/shell/config.rs` (`mouse_scroll_lines_for`),
  `src/client/shell/state.rs`, `src/client/shell/mouse.rs`,
  `src/client/attach.rs` (`AttachEscapeState`, `attach_scroll_action`),
  `src/client/mod.rs`, default config text in `src/main.rs`.
- Tests: `src/config/model.rs`, `src/client/shell/config.rs`,
  `src/client/attach.rs`, `src/client/shell/tests/mouse_selection.rs`.
- Docs: `docs/next/website/src/data/config-reference.json`,
  `docs/next/CHANGELOG.md`.
- Overlap signals: upstream assigns another meaning to Alt or other modifiers
  on wheel events (horizontal scroll, zoom, passthrough), adds scroll
  acceleration, or reworks `mouse_scroll_lines` plumbing.

### Terminal bell sound backend

Commit `fae45060`. New config key `ui.sound.backend` = `audio` (default, the
upstream mp3 player path) | `bell` (write BEL to the client's stdout) | `both`.

- Code: `src/config/sound.rs` (`SoundBackend`), `src/sound.rs` (`play`,
  `write_bell`), default config text in `src/main.rs`.
- Tests: `src/config/sound.rs`, `src/sound.rs`.
- Docs: `docs/next/website/src/content/docs/configuration.mdx`,
  `docs/next/website/src/data/config-reference.json`.
- Overlap signals: upstream adds bell or OS notification delivery, a different
  notification backend switch, or moves sound playback out of the client.

### Pi session name metadata

Commit `ca3535cb`. The bundled pi extension reports the pi session name as pane
metadata token `pi_session` under source `herdr:pi-session-name`, on
`session_start` and on `session_info_changed`. Sidebar configs can display it
like any other metadata token.

- Code: `src/integration/assets/pi/herdr-agent-state.ts` (`reportSessionName`).
- Tests: `src/integration/assets/herdr-agent-state.test.ts`,
  `pi_integration_reports_session_name_metadata` in `src/integration/tests.rs`.
- Overlap signals: upstream changes the pi extension, its integration asset
  version, `pane.report_metadata`, or starts reporting session names itself.
  Integration asset version rules from `AGENTS.md` apply when merging pi asset
  edits.

### Pi resume through the `py` wrapper

Commit `0b5f7787`. Session restore resumes pi with `py --session <ref>` instead
of `pi --session <ref>`; `py` is the user's local pi wrapper binary.

- Code: `("herdr:pi", "pi", ...)` arm of `plan` in `src/agent_resume.rs`.
- Tests: `src/agent_resume.rs`, `src/persist/restore.rs` (expected argv).
- Overlap signals: upstream changes the pi resume argv, adds a configurable
  resume command or binary override (which could replace this hardcoded
  patch), or changes restore plans.

### Alabaster themes

Commit `d86bbcb4`. Adds built-in themes `alabaster-dark` and `alabaster-light` and pairs
them as dark/light siblings for appearance-following theme switching.

- Code: `Palette::alabaster_dark`, `Palette::alabaster_light`, and
  `Palette::from_name` in `src/app/state.rs`; `THEME_NAMES` and
  `canonical_theme_name` in `src/config/theme.rs`; `sibling_theme_names` in
  `src/app/mod.rs`; built-in theme list in the default config text in
  `src/main.rs`.
- Overlap signals: upstream adds an Alabaster theme under any name, changes the
  `Palette` fields, or reworks theme name resolution or sibling pairing.

### Upstream sync skill

Commit `068694dc`. `.agents/skills/herdr-upstream-sync/SKILL.md` is the
fork's merge-based workflow for syncing with `upstream/master`. The
`.claude/skills` symlink to `.agents/skills` exposes the repository skills to
Claude Code. Upstream ships neither, so they should never conflict.
