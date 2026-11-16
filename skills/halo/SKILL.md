---
name: halo
description: Put a human's eyes and mouse on a browser running on a headless worker, AnyDesk style, with one command (`halo`, also `screenshare`) that brings up Xvfb, Chrome and a RustDesk server on the worker and opens the RustDesk viewer on the controller already authenticated. Use whenever a task on a worker needs a human for a login, 2FA, a captcha or an OAuth consent step, "I need to see the browser on the worker", "let me log in there", "show me the screen", "halo", "screenshare", RustDesk, VNC, Xvfb, virtual display, remote GUI, or when the viewer cannot reach the worker.
metadata:
  harness: [claude, codex, cursor, opencode]
  platform: [darwin, linux]
  requires: "npm i -g fleet-halo on the controller (puts `halo` on PATH); rustdesk, Xvfb, Chrome or Chromium and ImageMagick on the worker; Tailscale"
---

# Halo

A worker has no display. When a task there needs a human (a login, 2FA, a captcha, an OAuth
consent screen), run one command on the controller:

```bash
halo                     # bring it all up and open the viewer
halo https://site/login  # same, and open that URL in the remote Chrome
```

The target is `SCREENSHARE_HOST` (an ssh alias, default `worker`). When worker-chrome runs on
the worker, the viewer shows that Chrome, so a login the user finishes there is the one agents
use. `SCREENSHARE_CHROME=own` shows a separate Xvfb `:99` Chrome instead. The viewer opens already
authenticated with that worker's random password. Nothing to type.

| Command | Does |
| --- | --- |
| `halo [url]` | Starts whatever is missing on the worker, checks the port, opens the viewer |
| `halo goto <url>` | Opens a new tab in the remote Chrome. No viewer |
| `halo shot [out.png]` | Screenshot of the remote screen. Default `/tmp/halo.png` |
| `halo status` | Checks Xvfb, Chrome, the RustDesk server and voxl |
| `halo down` | Quits the viewer only. The remote session and logins keep running |
| `halo stop` | Also stops the units halo started on the worker |
| `halo where` | Prints the worker's short hostname and the display being shared |

With `SCREENSHARE_VIEWER_URL` set, `halo [url]` opens that URL (`{worker}` and
`{display}` filled in) instead of the RustDesk viewer. If voxl is installed on the worker, the
display is also streamed to the browser over tailnet HTTPS (port 16999 unless a voxl already
serves it). RustDesk keeps running either way as the fallback.

## Agent workflow

1. `halo shot` first and look at the PNG. If the page is already past the login, nobody
   needs to look at anything.
2. Otherwise run `halo <url>`, tell the user exactly what to do on the screen, and wait.
3. Verify with `halo shot` again. Leave the remote side running. Run `halo down`
   only when the user is done.

**Shared Chrome.** If a Chrome with CDP on 9222 is already running on `:99` (another bot or job
on the worker), halo adopts it instead of starting a second one, and `goto` opens a tab in
that browser. Close tabs you opened (`curl -s http://127.0.0.1:9222/json/close/<id>` on the
worker) so the other job doesn't act on the wrong one.

## When it breaks

- `halo: remote side failed`: rerun traced, `bash -x "$(command -v halo)"`, to see
  which step died.
- `RustDesk on ... is not reachable`: Tailscale is down on one end, or the server died.
  `halo status`.
- **Black viewer:** the server is not capturing `:99`. Run `halo` again; it restarts it there.
- **Password rejected:** delete `~/.screenshare/password` on the worker, run `halo`. It
  makes a new one and restarts the server with it.
- **Xvfb died:** the server does not survive its display going away. `halo stop`, then
  `halo`.

How the chain works, the firewall rule, and the traps the script already handles (do not undo
them): the package README, `$(npm root -g)/fleet-halo/README.md`.
