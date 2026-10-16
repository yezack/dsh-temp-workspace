# Changelog

All notable changes to this project are documented in this file.

## [0.2.0] - 2026

### Added
- **A collapsible "临时工作区" group inside the Workspace list.** Every
  temporary workspace — and, nested under it, its own conversations — now lives
  in one group row of the ordinary "工作区" list instead of mixing into it as
  extra top-level workspaces. The row is pinned **first** in the list and
  out-dented to `padding-left: 0`, so it reads as the parent of the workspace
  rows below it rather than as one more workspace. It carries a count, a caret
  that remembers its collapsed state across page reloads, and hover actions for
  *new temporary workspace*, *keep permanently*, *new session* and *delete*.
- `/temp-workspace/api/list` now also reports each entry's registry `title` and
  `sessionIds`; the boot-confirm payload reports `title` too, so the dialog
  lists `2026-10-16 09:45` instead of a throwaway folder's UUID.

### Changed
- **New temp workspaces are named after their creation moment**
  (`yyyy-MM-dd HH:mm`, local time) instead of every one sharing the title
  "临时工作区" — the sidebar rows are now telling apart at a glance.
- The browser half injects `uiWorkspace` and `sessions` in addition to
  `workspaces` (the Workspace *controller*) and now resolves navigation through
  the right service: `uiWorkspace.startSession/openSession/pickDirectory`.

### Fixed
- Creating a temp workspace now actually opens a conversation in it. The "open a
  new session" step used to call `startSession` on the Workspace controller,
  where that method does not exist, so it threw and was swallowed — the
  workspace appeared in the sidebar with no session.
- Creating a temp workspace no longer trips the boot confirmation dialog. The
  "pending deletion" list fell back to the durable marker once the boot pass had
  already finished, so a workspace created *now* was reported as held from a
  previous boot. `/pending` is now authoritative after the boot pass completes.
- The per-row permanent-keep button no longer relies on every temp workspace
  sharing the title "临时工作区"; the actions live on the group's own rows.
- `apply` is idempotent across a hot reload: `dsh-client-hmr` re-evaluates the
  browser half on every rebuild and does not always run the previous instance's
  dispose, which left a second group (and a second poll timer) in the sidebar.
  The new instance now tears the previous one down and drops any stray nodes.
- The temp list is refreshed by its own 4s interval instead of riding the
  `MutationObserver`: a tick only happens when the DOM changes, so a change made
  out of band (boot cleanup, another tab, the host route) used to leave a stale
  group row behind.

### Notes
- The group is decoration over the live DOM, because DSH declares no slot for a
  synthetic workspace group (`sidebar.workspaces` is a single hole owned by the
  native browser). Temp workspaces' native rows are hidden with `display:none`
  — never moved, and the group subtree is only ever inserted as the tree's first
  child, which React never references — so the native tree keeps reconciling
  untouched. If a future Harness changes the sidebar's DOM shape, the group may
  need a matching placement update — the same caveat the injected icon already
  carries.
- The native Workspace search can still surface a temp conversation; only the
  list rows are regrouped.

## [0.1.0] - 2026

### Added
- Temporary workspace for DSH web: a one-click throwaway workspace whose
  conversations are deleted at the next Harness restart (immediately or after a
  delay, optionally behind a confirm dialog).
- Pin a temp workspace to keep it permanently: files move into the folder you
  pick, the folder name becomes the workspace title, and its conversations are
  migrated (followed by a real host-level restart to re-index).
- Settings → Plugins card (`dsh-temp-workspace`) with `deleteMode`,
  `deleteDelay` and `confirmBeforeDelete`.
- npm prebuilt install (`@wjj-8283/dsh-temp-workspace`) so users get a
  one-command install that skips the `allowBuilds` build-approval step.

### Changed
- `build.mjs` gained a `--check` mode to keep the committed prebuilt
  `lib/client.js` in sync with `src/client.js`.

### Fixed
- Works on dsh 0.1.7: the settings mechanism moved from the removed
  per-namespace `ctx.settings.register(ns, schema)` scope to the Loader entry's
  `Config` schema, and writes now go through
  `ctx.configEditor.edit(ctx.fiber.entry, …)`.
- Saving a setting no longer re-runs the boot cleanup: the boot pass and the
  confirmation-dialog state live at module scope, so a Loader config reload in
  the same process cannot re-arm the dialog or delete held workspaces.

### Notes
- No runtime dependencies on official `@deepseek-ai/*` packages: the plugin
  receives services through the Cordis `ctx` injection, so nothing is
  duplicated inside the profile. `Config` does need `@deepseek-ai/schemastery`
  at module scope, so it is resolved from the dsh runtime root
  (`<DSH_HOME>/profiles/node_modules`) — see the comment in `lib/index.js`.
