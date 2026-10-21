# Changelog

All notable changes to this project are documented in this file.

## [0.2.4] - 2026

### Changed
- **The better-workspace-mode temp-workspace title now carries the time to the
  second**: `临时工作区/yyyyMMddHHmmss` (e.g. `临时工作区/20261021175513`) instead of
  `临时工作区/yyyyMMdd`. Second precision is enough to tell same-day runs apart on
  its own, so the `-2` / `-3` suffix is now only a same-second guard; the
  `临时工作区` group name (and therefore the nesting) is unchanged.

## [0.2.3] - 2026

### Fixed
- **`新建会话` stopped opening the blank New Session page when another plugin
  wrapped `uiWorkspace.startSession`.** The "no argument means no Workspace"
  rule was a one-off method assignment made at activation, so any plugin that
  replaced the method later silently took over the call. `@linxin666/dsh-client-ui-git-graph`'s
  *auto-isolation* does exactly that: it pre-resolves the target Workspace for
  itself and forwards it to the previous implementation as an **explicit** id —
  so the shipped policy bound the new session to the current/most-recent
  Workspace again, and the hero page opened with a Workspace already chosen.
  The interception is now installed as a **property accessor**: a later
  assignment of `startSession` is captured by the setter and the plugin's
  policy is re-wrapped around it, so it stays outermost no matter which plugin
  patches last. A bare call still clears the panel; an explicit Workspace id
  (workspace-row `＋`) and draft options — `startSession(void 0, { agentPreset })`,
  used by the agent-preset creator chip — are forwarded untouched.

### Added
- **Interop with `dsh-better-workspace`.** That plugin owns the sidebar
  Workspace tree and derives virtual folders from every `/` in a Workspace title
  (`web/前端` nests under `web`). When it is part of the client graph — checked
  in the boot manifest (`dsh-better-workspace`), with a `.bw-tree` DOM probe as
  fallback — this plugin now rides that mechanism instead of injecting its own
  group:
  - temporary workspaces are titled **`临时工作区/yyyyMMdd`** (e.g.
    `临时工作区/20261020`), so better-workspace nests all of them under one
    `临时工作区` folder. A same-day collision gets `-2`, `-3`, …;
  - the sidebar injection is skipped entirely (no group row, no native-row
    hiding, no 4 s list poll) — the tree, rename/delete/reorder all come from
    better-workspace. Renaming goes through the registry `rename` call
    better-workspace's own rename uses; deleting through its menu still runs this
    plugin's cleanup hook (sessions + directory + marker);
  - the hero `新建临时工作区` chip, the boot confirm dialog and the settings
    card are unchanged.
  Note: better-workspace's **视图选项 → 工作区分组** must be **磁盘 + 名称** or
  **仅名称**; its default **磁盘目录** does not derive `/` groups and shows the
  raw `临时工作区/20261020` title flat.

### Removed
- **The sidebar "临时工作区" hourglass icon button** next to «Add workspace», and
  all the machinery that only existed to keep it alive: `findAddButton` /
  `insertIcon`, the `ADD_LABELS` / `ICON_CLASS` / `BUTTON_CLASS` constants, the
  `inserting` guard, the icon calls in the locale subscription, the rAF
  scheduler and the dispose path, and the now-unused `iconTitle` locale string.
  Creating a temp workspace is still reachable from the blank New Session page's
  **`新建临时工作区`** chip and from the **临时工作区** group row's **＋**. The
  icon was also the fragile part of the DOM decoration: it matched the native
  «Add workspace» button by *exact* `aria-label`, so a plugin that re-renders
  that header with a shortcut suffix in the label (e.g. `dsh-better-workspace`'s
  own `添加工作区 (Ctrl+Alt+O)` button) silently left the icon un-rendered.

## [0.2.2] - 2026

### Fixed
- **Works on dsh 0.2 (`0.2.1-rc.2`), which removed the `webRuntime` service.**
  `webRuntime` was part of the plugin's `inject` list, so cordis's injection gate
  kept the **whole plugin** from activating (its only symptom was
  `webRuntime — 提供者 0`). `inject` now declares just the three services the
  plugin truly requires — `webServer`, `workspaceRegistry`, `sessionPersistence`
  — and the browser-trust verdict is resolved per request instead:
  - dsh >= 0.2: `connection.allowsRemoteAuthorities`, the same answer the shell
    itself uses for every browser request;
  - dsh <= 0.1.x: `webRuntime.trustedHosts`, compared against the request's Host.
  One build therefore works on both releases.
- Audited every other host service against 0.2.1 while fixing this: the route
  shape (`webServer.register({ kind: 'prefix', path, handler(req, res) })`) and
  the `workspaceRegistry` (`create`/`get`/`list`/`delete`/`insertBefore`/
  `archiveSession`/`unarchiveSession`/`resolveByPath`/`attachSession`) and
  `sessionPersistence` (`root`/`locate`/`list`/`create`/`open`/`stat`) surfaces
  are unchanged, as are the client services this plugin injects (`workspaces`,
  `uiWorkspace`, `sessions`, `slots`, `locale`).

## [0.2.1] - 2026

### Added
- **A `新建临时工作区` control on the blank New Session page**, in the hero's
  control row beside the native `选择工作区` picker and the agent-preset chip
  (identical metrics: 28px tall, 8px padding, 8px radius, 13px/500 label). It
  creates a temp workspace and opens a conversation in it — the same action as
  the sidebar hourglass, now reachable exactly where a new session is started.
- It is **shown only while no Workspace is chosen** for that New Session: once
  one is picked, the shortcut to a temporary Workspace goes away with the
  decision. Detection reads the native chip's own state — its `aria-label` is
  always the "choose workspace" call to action while its visible label becomes
  the chosen Workspace's title — so no locale string is involved.

### Changed
- **"New Session" no longer binds a Workspace by itself.** The sidebar's
  `新建会话` button, the brand row and the `session.new` shortcut all call
  `uiWorkspace.startSession()` with no workspace, and the shipped policy then
  auto-bound the new session to the current Session's Workspace or to the most
  recently active one. That no-argument call now opens the **blank New Session
  page** instead, so the Workspace — or a fresh temporary one — is chosen
  deliberately there. A call with an explicit workspace id keeps the original
  behaviour, which this plugin's own temp-workspace rows rely on.

### Notes
- The hero's control row is a composition of three *single* slots declared by
  ui-conversation, so there is no list seat to register a fourth control into
  (filling one would shadow the shipped UI). The button is therefore appended to
  the live row, like the sidebar icon and the group.

## [0.2.0] - 2026

### Added
- **A collapsible "临时工作区" group inside the Workspace list.** Every
  temporary workspace — and, nested under it, its own conversations — now lives
  in one group row of the ordinary "工作区" list instead of mixing into it as
  extra top-level workspaces. The row is pinned **first** in the list and
  out-dented to `padding-left: 0`, so it reads as the parent of the workspace
  rows below it rather than as one more workspace. Its single leading icon is a
  **closed tray / opened tray pair** rather than the folder glyph the Workspace
  rows use — one icon carrying both the group's identity and its expanded or
  collapsed state, with no separate rotating caret. The row also carries a count,
  remembers its collapsed state across page reloads, and offers hover actions for
  *new temporary workspace*, *keep permanently*, *new session* and *delete*.
- `/temp-workspace/api/list` now also reports each entry's registry `title` and
  `sessionIds`; the boot-confirm payload reports `title` too, so the dialog
  lists `2026-10-16 09:45` instead of a throwaway folder's UUID.

### Changed
- **Published as `@yezack/dsh-temp-workspace`** — this fork's own scope is the
  only one this repository can publish to, so the package name, the Loader
  entry's `name`, and the `homepage`/`repository`/`bugs` URLs now all point at
  `yezack/dsh-temp-workspace`. The Loader entry **id** and the settings
  namespace stay `dsh-temp-workspace`, so existing settings are untouched;
  only the install spec changes.
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
