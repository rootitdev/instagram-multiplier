# instagram-multiplier

Selective Instagram traffic billing helper for a PasarGuard + Xray 2-hop stack.

For listed panel users, Instagram-domain traffic is counted on a dedicated Xray outbound (`CAT-instagram-u<id>`) and billed at `INSTAGRAM_MULTIPLIER` (default 1.5×) when stats are ingested. Other users are unchanged.

## Files

| File | Role |
|---|---|
| `instagram_multiplier.py` | Core: inject CAT outbounds/rules into Xray JSON; weight stats deltas |
| `instagram-multiplier` | CLI: `add` / `remove` / `list` by panel username |
| `instagram-multiplier.env.example` | Multiplier + domain list |
| `instagram-multiplier-users.txt.example` | User-id list template |

## Live paths (on the Exit/panel host)

```text
/usr/local/sbin/instagram-multiplier
/root/vpn-infra/scripts/instagram_multiplier.py   # or beside the CLI on PYTHONPATH
/etc/vpn-monitor/instagram-multiplier.env
/etc/vpn-monitor/instagram-multiplier-users.txt
```

## Install (Exit server)

```bash
install -m 755 ./instagram-multiplier /usr/local/sbin/instagram-multiplier
install -m 644 ./instagram_multiplier.py /root/vpn-infra/scripts/instagram_multiplier.py
mkdir -p /etc/vpn-monitor
cp -n ./instagram-multiplier.env.example /etc/vpn-monitor/instagram-multiplier.env
cp -n ./instagram-multiplier-users.txt.example /etc/vpn-monitor/instagram-multiplier-users.txt
chmod 600 /etc/vpn-monitor/instagram-multiplier.env /etc/vpn-monitor/instagram-multiplier-users.txt
```

Edit `INSTAGRAM_MULTIPLIER` in the env file as needed. The CLI expects:

- Docker container `pasarguard` with sqlite at `/code/db.sqlite3`
- Export script at `/root/vpn-infra/scripts/iran_sync_export.py` (triggers apply to Iran after add/remove)

## Usage

```bash
instagram-multiplier list
instagram-multiplier add <panel-username>
instagram-multiplier remove <panel-username>
```

## Notes

- Do not commit live user lists or secrets.
- Multiplier only affects billing math in stats ingest; it does not change connection speed.
- Injection runs during Iran config export; empty user list removes all CAT rules.
