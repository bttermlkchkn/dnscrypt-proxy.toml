# dnscrypt-proxy Privacy Configuration (Anonymized DNSCrypt)

Stop worrying about your ISP snooping, telemetry from public providers, or configuring complex firewall rules. This configuration enables **maximum privacy** for your DNS queries using **Anonymized DNSCrypt** via high-security relays.

<img width="880" height="275" alt="image" src="https://github.com/user-attachments/assets/948dba5e-8593-4399-847f-50b458da01be" />


## 🛡️ Features

*   **Anonymized DNS:** Your traffic is routed through multiple servers. No single entity (Relay, Resolver, or ISP) knows both **who you are** and **what you are searching for**.
*   **No Google/Cloudflare:** Uses Quad9 and European privacy-focused resolvers (DCT, DigitalPrivacy).
*   **High Privacy Jurisdictions:** Relays are prioritized in Switzerland, Germany, and France (countries with strong privacy laws).
*   **Security:** DNSSEC is enforced by default to prevent DNS poisoning.
*   **Malware Blocking:** Uses Quad9's filtered servers as primary protection.
*   **Redundancy:** If Quad9 goes down, the system automatically fails over to `dct-de` or `dct-fr` without losing privacy.

## 🚀 Quick Start

1.  **Stop the service:**
    ```bash
    sudo systemctl stop dnscrypt-proxy
    ```

2.  **Backup your current config:**
    ```bash
    sudo cp /etc/dnscrypt-proxy/dnscrypt-proxy.toml /etc/dnscrypt-proxy/dnscrypt-proxy.toml.backup
    ```

3.  **Paste the config below:**
    *   Open your config file: `sudo nano /etc/dnscrypt-proxy/dnscrypt-proxy.toml`
    *   **Delete everything** in the file.
    *   **Paste** the configuration block found at the bottom of this README.

4.  **Restart the service:**
    ```bash
    sudo systemctl start dnscrypt-proxy
    ```

## ⚠️ Crucial Step: Configure System DNS

The `dnscrypt-proxy` service is now running locally on port `53`. **You must tell your operating system to use it.**

Run this command to verify:
```bash
cat /etc/resolv.conf
```

*   If it says `nameserver 127.0.0.1`, you are done.
*   If it lists an IP (e.g., `192.168.1.1`), you need to change your network settings (DHCP or Static) to point to `127.0.0.1`.

## 🛠️ Configuration

Save this content as `dnscrypt-proxy.toml` or git clone the repo and move the file.

```toml
listen_addresses = ['127.0.0.1:53']
max_clients = 250

ipv4_servers = true
ipv6_servers = false
dnscrypt_servers = true
doh_servers = false
odoh_servers = false

server_names = [
  'quad9-dnscrypt-ip4-filter-pri',
  'quad9-dnscrypt-ip4-filter-alt',
  'dct-de',
  'dct-fr',
  'digitalprivacy.diy-dnscrypt-ipv4'
]

require_dnssec = true
require_nolog = true
require_nofilter = false

ignore_system_dns = true
# Quad9 and mullvad as bootstrap resolvers
bootstrap_resolvers = ['9.9.9.9:53', '194.242.2.2:53']
netprobe_address = '9.9.9.9:53'
netprobe_timeout = 60

cache = true
cache_size = 4096
cache_min_ttl = 2400
cache_max_ttl = 86400

[anonymized_dns]
skip_incompatible = true
## Relays prioritized by privacy laws (Switzerland, Germany, France)
routes = [
  { server_name='*', via=['anon-cs-ch', 'anon-cs-de2', 'anon-dct-de', 'anon-digitalprivacy.diy-ipv4', 'anon-cs-fr'] },
]

[sources.quad9-resolvers]
urls = [
    'https://quad9.net/dnscrypt/quad9-resolvers.md',
    'https://raw.githubusercontent.com/Quad9DNS/dnscrypt-settings/main/dnscrypt/quad9-resolvers.md'
]
minisign_key = "RWTp2E4t64BrL651lEiDLNon+DqzPG4jhZ97pfdNkcq1VDdocLKvl5FW"
cache_file = "quad9-resolvers.md"
refresh_delay = 72
prefix = "quad9-"

[sources.public-resolvers]
urls = [
  'https://raw.githubusercontent.com/DNSCrypt/dnscrypt-resolvers/master/v3/public-resolvers.md',
  'https://download.dnscrypt.info/resolvers-list/v3/public-resolvers.md'
]
cache_file = 'public-resolvers.md'
minisign_key = 'RWQf6LRCGA9i53mlYecO4IzT51TGPpvWucNSCh1CBM0QTaLn73Y7GFO3'
refresh_delay = 73

[sources.relays]
urls = [
  'https://raw.githubusercontent.com/DNSCrypt/dnscrypt-resolvers/master/v3/relays.md',
  'https://download.dnscrypt.info/resolvers-list/v3/relays.md',
]
cache_file = 'relays.md'
minisign_key = 'RWQf6LRCGA9i53mlYecO4IzT51TGPpvWucNSCh1CBM0QTaLn73Y7GFO3'
refresh_delay = 73
prefix = ''
```

## 🧐 Troubleshooting

**Service fails to start (`FATAL` error):**
*   Ensure you have **completely deleted** old content in the file before pasting.
*   Do not use a text editor that auto-formats. Use `nano` or `vim`.

**Logs show `quad9-doh...` instead of `quad9-dnscrypt...`:**
*   This means `doh_servers` is still `true`. Check your `toml` file carefully. `doh_servers` MUST be `false` for anonymization to work.

**Internet stops working:**
*   This almost always means your **System DNS** (in Network Manager) is still pointing to `8.8.8.8` or your router, not `127.0.0.1`.

## 📜 License / Credits

Configuration created by a fellow extremely-paranoid privacy advocate. Special thanks to the maintainers of **Quad9**, **DCT**, and **dnscrypt-proxy**.
