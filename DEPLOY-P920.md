# HexStrike AI — P920 container deploy (MarQed)

Fork of HexStrike AI via the `netcuter` "Hexstrike 7 PL" security-hardened branch.
Runs as an MCP server that lets AI agents (Claude Code) drive 150+ security tools.

> **Authorized testing only.** Offensive tooling. Run against systems you own or
> have explicit written permission to test. P920 = test/acceptatie, isolated.

## Where it runs
- Host: **P920 / marqed003** (Tailscale `100.84.112.24`)
- Port: **8888**, published **only on the Tailscale interface** (not public/LAN).
- Reachable from the tailnet: `http://100.84.112.24:18888/`

## First deploy
```bash
# on P920
git clone https://github.com/zeeneddie/hexstrike-ai.git ~/hexstrike-ai
cd ~/hexstrike-ai
cp .env.p920.example .env
# put a real key in .env:
sed -i "s/replace-me/$(openssl rand -hex 24)/" .env
docker compose -f docker-compose.yml -f docker-compose.p920.yml up -d --build
```

## Health check
```bash
curl -fsS http://100.84.112.24:18888/health    # /health bypasses auth
```

## Connect Claude Code (MCP)
The server requires `HEXSTRIKE_API_KEY`. Point the MCP client (`hexstrike_mcp.py`)
at `http://100.84.112.24:18888` and pass the key. See `hexstrike-ai-mcp.json`.

## Security knobs (set in docker-compose.p920.yml)
| env | value | why |
|-----|-------|-----|
| `HEXSTRIKE_HOST` | `0.0.0.0` | Flask must listen inside the container for the port map to work |
| `HEXSTRIKE_REQUIRE_API_KEY` | `true` | no anonymous access to offensive tooling |
| `HEXSTRIKE_GUARDRAILS` + `_FAIL_CLOSED` | `true` | scope-enforcement / blast-radius, deny on doubt |
| `HEXSTRIKE_VALIDATE_COMMANDS` | `true` | command sanitization |
| `HEXSTRIKE_RATE_LIMIT` | `true` | throttle |

## Image notes
P920 builds from **`Dockerfile.p920` (Kali rolling base)** — the upstream/netcuter
`Dockerfile` targets `python:3.11-slim`, where `nikto`/`gobuster` and most of the
150 tools are not in Debian repos and the build fails. Kali ships them.
Current tool set: nmap, nikto, gobuster, dirb, hydra, sqlmap, whatweb, wafw00f.
Tools beyond that report "not available" until added to `Dockerfile.p920`'s
`apt-get install` list (Kali has the rest: ffuf, feroxbuster, nuclei, wpscan, …).

## Promotion flow
laptop = dev · **P920 = test/acceptatie (here)** · P620 = productie. Do NOT promote
offensive tooling to P620 without a separate security review.
