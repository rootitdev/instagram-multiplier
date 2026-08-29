# instagram-multiplier (+ vpn-path)

Portable helpers from a PasarGuard + Xray **2-hop** stack (Iran relay → Exit).

This repo holds two tools:

1. **instagram-multiplier** — selective Instagram billing (1.5× by default) for listed panel users  
2. **vpn-path** — switch / auto-failover the Iran→Exit backhaul selector in the panel core  

No live user lists or secrets are included.

---

## 1) instagram-multiplier

| File | Role |
|---|---|
| `instagram_multiplier.py` | Inject `CAT-instagram-u<id>` outbounds/rules; weight stats |
| `instagram-multiplier` | CLI: `add` / `remove` / `list` |
| `instagram-multiplier.env.example` | Multiplier + domains |
| `instagram-multiplier-users.txt.example` | User-id list template |

### Live paths (Exit / panel host)

```text
/usr/local/sbin/instagram-multiplier
/root/vpn-infra/scripts/instagram_multiplier.py
/etc/vpn-monitor/instagram-multiplier.env
/etc/vpn-monitor/instagram-multiplier-users.txt
```

### Install

```bash
install -m 755 ./instagram-multiplier /usr/local/sbin/instagram-multiplier
install -m 644 ./instagram_multiplier.py /root/vpn-infra/scripts/instagram_multiplier.py
mkdir -p /etc/vpn-monitor
cp -n ./instagram-multiplier.env.example /etc/vpn-monitor/instagram-multiplier.env
cp -n ./instagram-multiplier-users.txt.example /etc/vpn-monitor/instagram-multiplier-users.txt
chmod 600 /etc/vpn-monitor/instagram-multiplier.env /etc/vpn-monitor/instagram-multiplier-users.txt
```

CLI expects Docker `pasarguard` and `/root/vpn-infra/scripts/iran_sync_export.py`.

```bash
instagram-multiplier list
instagram-multiplier add <panel-username>
instagram-multiplier remove <panel-username>
```

---

## 2) vpn-path (sometimes typed “vpn-patch”)

Runs on the **Exit** host. Changes the backhaul balancer selector inside the panel core (`TO-AMS-CF` / `XHTTP` / `DIRECT` / `TUIC` / `auto`). Iran live Xray picks it up via the desired-state sync — not via editing systemd JSON on Iran.

| Path in repo | Install to |
|---|---|
| `vpn-path/vpn-path` | `/usr/local/sbin/vpn-path` |
| `vpn-path/vpn-path-core.py` | beside scripts (imported) |
| `vpn-path/vpn_path_lib.py` | `/root/vpn-infra/scripts/` |
| `vpn-path/vpn_paths.py` | `/root/vpn-infra/scripts/` |
| `vpn-path/path_health.py` | Iran health probe (used by failover) |
| `vpn-path/vpn_failover.py` | Exit failover daemon |
| `vpn-path/vpn-failover.service` + `.timer` | `/etc/systemd/system/` |
| `vpn-path/vpn-path-iran-stub.sh` | optional Iran `/usr/local/sbin/vpn-path` |

### Install (Exit)

```bash
# put Python modules where the CLI expects them
install -m 644 vpn-path/vpn_path_lib.py vpn-path/vpn_paths.py \
  vpn-path/vpn-path-core.py vpn-path/vpn_failover.py vpn-path/path_health.py \
  /root/vpn-infra/scripts/
install -m 755 vpn-path/vpn-path /usr/local/sbin/vpn-path
install -m 644 vpn-path/vpn-failover.service vpn-path/vpn-failover.timer \
  /etc/systemd/system/
mkdir -p /var/log/vpn-path
systemctl daemon-reload
systemctl enable --now vpn-failover.timer
vpn-path status
```

Adapt hardcoded Iran SSH target / IPs in the scripts for your stack. Secrets stay in `/root/.vpn-secrets/.env` only (`PG_CORE_ID`, panel API key, etc.).

```bash
vpn-path status|auto|cf|xhttp|direct|tuic|publish|test
```

---

## Notes

- Do not commit live Instagram user lists or `.env` secrets.
- Multiplier only affects billing math; it does not change connection speed.
- `vpn-path` on Iran is a stub; run real commands on Exit.
