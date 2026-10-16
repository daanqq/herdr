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

### Upstream sync skill

Commit `068694dc`. `.agents/skills/herdr-upstream-sync/SKILL.md` is the
fork's merge-based workflow for syncing with `upstream/master`. The
`.claude/skills` symlink to `.agents/skills` exposes the repository skills to
Claude Code. Upstream ships neither, so they should never conflict.

## Moved out of the fork

These features used to be source patches and now live in the user's own
configuration, so a sync does not touch them. Do not reintroduce them as
patches; flag upstream changes that would break the external replacement.

- Pi resume through a proxy wrapper: `alias pi=...` in `~/.zshrc.local`.
  Restore types `pi --session <ref>` into the pane's interactive shell, which
  expands the alias. Breaks if upstream stops resuming through the shell.
- Pi session name metadata: `herdr-pi-session-name.ts` beside the managed
  `herdr-agent-state.ts` in `~/.pi/agent/extensions/` reports `pi_session`
  through `pane.report_metadata`. Breaks if upstream changes that method or its
  per-source `seq` rule.
- Alabaster themes: `[theme.custom.dark]` and `[theme.custom.light]` in
  `~/.config/herdr/config.toml` override every palette token while
  `auto_switch` is on. Breaks if upstream renames palette tokens or mode
  overrides.
