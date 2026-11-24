# Keep CLOSR's automatic backup in every update (added in 2.62.3)

Before building the next CLOSR update, start from the code in **releases/CLOSR-update-2.62.3.closr**
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
