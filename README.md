# LINUX
תיעוד מלא של כל מה שרלוונטי לדרך והשימוש בלינוקס 

## Yana remote access (from nogi's main rig)

Opens Yana's PC through a Cloudflare Access SSH tunnel (`ssh.yana-sr.com`). Tunnels start **on demand**; nothing runs at login.

| File | Does |
|---|---|
| `bin/yana-desktop` | Her XFCE desktop (xrdp, port 3390). Starts the tunnel if needed, then opens `sdl-freerdp` |
| `bin/yana-screen` | Her real screen (Plasma, KRdp, port 3389). She sees a "screen is shared" indicator |
| `systemd/yana-xrdp-tunnel.service`, `systemd/yana-krdp-tunnel.service` | The SSH tunnels. `Restart=on-failure`, max 3 starts per 5 min |

Install: copy `bin/*` to `~/.local/bin`, `systemd/*` to `~/.config/systemd/user`, run `systemctl --user daemon-reload`. Do not `enable` the services.

If a script says "login required": `cloudflared access login https://ssh.yana-sr.com` (once).

**Why on demand:** the xrdp tunnel was once enabled with `Restart=always`. When the Access login expired, `cloudflared` opened the login page in the browser every ~15 s (101 restarts).
