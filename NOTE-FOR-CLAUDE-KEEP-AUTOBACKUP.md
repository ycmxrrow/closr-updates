# Keep CLOSR's automatic backup and GitHub key in every update (added in 2.62.3 / 2.62.4)

Before building the next CLOSR update, start from the code in **releases/CLOSR-update-2.62.4.closr**
(or newer), not from an older copy, so these stay in:

1. `lib/autobackup.js` (the whole file).
2. In `main.js`, near the top after `const Upd = require('./lib/updater');`:
   `const AutoBk = require('./lib/autobackup'); let AUTOBK = null;`
3. In `main.js` `checkCloud()`, the line right before `Upd.install(b, BOOT.ovr);`:
   `if (AUTOBK) { try { await AUTOBK.run('before update ' + ver, ver); } catch (e) {} }`
4. In `main.js` `installUpdate()`, the line right before its `Upd.install(...)`:
   `if (AUTOBK) AUTOBK.run('before update ' + u.version, u.version).catch(() => {});`
5. In `main.js` `app.whenReady()`, right after `setInterval(() => checkCloud(), 2 * 3600 * 1000);`,
   the `AUTOBK = AutoBk.start({...})` block.

What it does: every hour, right before every update, and when a new version first starts, CLOSR saves
all its data to Google Drive for desktop (kissdawg account first) at
`My Drive/CLOSR Backups/<computer name>/` (falls back to `Documents/CLOSR Backups`). The `.closrbak`
files open with Import in the top bar. Never remove this.

## GitHub key (2.62.4) - required because this repository may be private

6. In `main.js`, after `const GH_RAW = ...`: the `ghKey()` and `ghFetch()` helpers. Every request to
   raw.githubusercontent.com / api.github.com (latest.json, releases, the update download, data.json and the
   data download) goes through `ghFetch(...)`, never plain `fetch(...)`.
7. `cfg.ghKey` is saved in CLOSR's settings only. Never add it to `BK_CFG`, backups or exports.
8. The `cloud-key` IPC handler in `main.js`, `cloudKey` in `preload.js`, `hasKey` in `cloud-state`/`sendCloud`,
   the `cloud-key` / `cloud-key-save` / `cloud-key-st` elements in `ui/index.html` and their code in `ui/appbar.js`.

If this repository is private and an update ships without `ghFetch`, every CLOSR that installs it can never
update itself again.

## Data lock (2.66.3) - keep it

9. In `main.js`, right above `function createWindow(hidden)`: `lockOn()`, `showingLock`, `sfDownSince`, `applyLock()` and
   `setInterval(applyLock, 3000)`. In `createWindow`, the main window loads `lockOn() ? 'lock.html' : 'index.html'` and sets
   `showingLock = lockOn()`.
10. `get-data`, `get-accounts` and `get-all` return `{locked: true, ...}` while `showingLock` is true.
11. The `lock-state` / `lock-set` IPC handlers (turning the lock OFF requires sfStatus === 'Connected'), `lockState` / `lockSet`
    in `preload.js`, the "Data lock (this computer)" checkbox (`lock-on`, `lock-st`) in `ui/index.html` + its code in
    `ui/appbar.js`, and `ui/lock.html`. `cfg.lockNoSf` is per computer: never add it to `BK_CFG`.
