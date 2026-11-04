# Guacamole — browser access to Yana's live desktop

**Status: already running.** This documents the real, already-deployed setup
on Yana's own machine.

## The URL

```
https://desk.yana-sr.com/guacamole/
```

Protected by Cloudflare Access, then by Guacamole's own login on top. Open it
from any browser — this is what makes it usable from a double-hop at work
(work PC → RDC to a remote Windows box → browser → this URL), where installing
an RDP client or SSH tunnel isn't an option.

## How it's actually built

Two Podman Quadlet containers, running as systemd user services on her
machine (`systemctl --user status guacd guacamole`):

| File | Does |
|---|---|
| [`quadlet/guacd.container`](quadlet/guacd.container) | `guacd`, the RDP/VNC proxy. Bound to localhost only — never exposed |
| [`quadlet/guacamole.container`](quadlet/guacamole.container) | The Guacamole web app |
| `~/guacamole/user-mapping.xml` (on her machine, **not in git** — has a real password) | Guacamole's flat-file login + saved RDP connection. Template: [`user-mapping.xml.example`](user-mapping.xml.example) |

Both quadlet files belong in `~/.config/containers/systemd/` on her machine;
systemd/podman auto-generates the matching `.service` units from them.

The RDP connection in `user-mapping.xml` points at her live KRdp (Plasma)
session — the same one `yana-screen` connects to from nogi's side. That's
deliberate: it's the real running desktop, not a fresh session.

## Exposing it (already done)

A Cloudflare Tunnel connector already runs on her machine, remotely managed
via the Cloudflare Zero Trust dashboard (Networks → Tunnels). The route for
`desk.yana-sr.com` plus an Access application on that hostname are both
already configured there.

## If this ever needs to be rebuilt from scratch

1. `sudo dnf install podman`
2. Put `user-mapping.xml` (real password, from the template) at `~/guacamole/user-mapping.xml`
3. Put both `.container` files in `~/.config/containers/systemd/`
4. `systemctl --user daemon-reload && systemctl --user enable --now guacd guacamole`
5. In the Cloudflare Zero Trust dashboard: add a public hostname on the existing
   tunnel pointing at the Guacamole container's port, and an Access application
   on it with the same policy as the other `*.yana-sr.com` apps.
