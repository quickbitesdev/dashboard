# Mullvad VPN Guide

## CRITICAL RULE
**NEVER install Mullvad VPN directly on the host or inside homebot/money-maker containers.**
It breaks all network connectivity including SSH.

## Architecture
Mullvad runs in its own dedicated Docker container as a SOCKS5/HTTP proxy.
Other containers route traffic through it ONLY when needed.

## Setup (Docker container)

### 1. Buy Mullvad Account
- Go to https://mullvad.net/en/account
- Pay with Wise virtual card (€5/month)
- Save account number

### 2. Create Mullvad Proxy Container
```yaml
# /mnt/GrimWin/containers/mullvad/docker-compose.yml
services:
  mullvad:
    image: ghcr.io/qdm12/gluetun
    container_name: mullvad-proxy
    restart: always
    cap_add:
      - NET_ADMIN
    environment:
      - VPN_SERVICE_PROVIDER=mullvad
      - VPN_TYPE=wireguard
      - WIREGUARD_PRIVATE_KEY=<from mullvad account>
      - WIREGUARD_ADDRESSES=<from mullvad account>
      - SERVER_COUNTRIES=Finland
      - HTTPPROXY=on
      - HTTPPROXY_LISTENING_ADDRESS=:8888
      - SOCKSPROXY=on
      - SOCKSPROXY_LISTENING_ADDRESS=:1080
    ports:
      - "8888:8888"   # HTTP proxy
      - "1080:1080"   # SOCKS5 proxy
```

### 3. Usage from Other Containers
```bash
# HTTP proxy
curl --proxy http://localhost:8888 https://example.com

# SOCKS5 proxy
curl --socks5 localhost:1080 https://example.com

# Python requests
import requests
proxies = {"https": "socks5://mullvad-proxy:1080"}
requests.get("https://example.com", proxies=proxies)
```

### 4. Usage in Money-Maker
When a bot needs VPN (e.g., Reddit posting):
```python
import os
proxies = None
if os.environ.get("USE_VPN"):
    proxies = {"https": "socks5://localhost:1080"}
response = requests.get(url, proxies=proxies)
```

## Status: ACTIVE
- Account: 6478927010800950 (env: MULLVAD_ACCOUNT)
- Paid until: 2026-04-26
- Purchased with Wise virtual card (5€/month)
- Renewal requires manual 3DS approval via Wise app

## Docker Proxy (RUNNING)
- Container: `mullvad-proxy` at `/mnt/GrimWin/containers/mullvad/`
- HTTP Proxy: `http://localhost:8889`
- Current server: Sweden

## Usage from Other Containers
```python
import requests
# Route through VPN
proxies = {"http": "http://host.docker.internal:8889", "https": "http://host.docker.internal:8889"}
# Or from host network mode containers:
proxies = {"http": "http://localhost:8889", "https": "http://localhost:8889"}
response = requests.get("https://example.com", proxies=proxies)
```

```bash
# From command line
curl --proxy http://localhost:8889 https://am.i.mullvad.net/ip
```

## Known Limitations
- **Reddit blocks Mullvad IPs** — all known VPN providers are blacklisted by Reddit. Cannot use for Reddit posting. Need residential proxy for Reddit.
- Good for: general web scraping, account creation on services that don't block VPNs, privacy
- Bad for: Reddit, some Google services, sites that block datacenter IPs
