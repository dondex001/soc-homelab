# Home SOC Lab

A home Security Operations Center built and debugged on VirtualBox: a highly available firewall, a SIEM, and a case-management stack wired together into one alert pipeline. Everything is documented here, including what broke and how it was fixed.

## Architecture

```
                 Internet / NAT
                       |
         +-------------+--------------+
         |   OPNsense CARP pair       |
         |  Master .1  Backup .2      |
         |  LAN VIP 192.168.1.100     |
         |  pfsync 10.0.0.0/24        |
         +-------------+--------------+
                       | intnet 192.168.1.0/24
   +----------+--------+---------+-------------+
   |          |                  |             |
 Wazuh     Ubuntu Server     Windows 10      Kali
 manager   (Docker stack)    client          (SSH jump)
 .10       .30               DHCP (.126)
           TheHive / Cortex / MISP
```

## Network map

| Device | Address | Notes |
|---|---|---|
| OPNsense Master LAN | 192.168.1.1 | Firewall, DHCP, Zenarmor |
| OPNsense Backup LAN | 192.168.1.2 | CARP backup |
| Failover Lab (multi-WAN) | 192.168.1.3 | Cloned from Master |
| CARP LAN VIP | 192.168.1.100 | VHID 1 |
| CARP WAN VIP | 10.0.2.254 | VHID 2 |
| pfsync | 10.0.0.1 / 10.0.0.2 | Master / Backup |
| Wazuh manager | 192.168.1.10 | Static; falls back to DHCP .192 |
| Ubuntu Server | 192.168.1.30 | Docker host (TheHive 9000, Cortex 9001, MISP 443) |
| Windows 10 client | 192.168.1.126 | DHCP |

## Components

| Component | Status | Doc |
|---|---|---|
| Foundation (OPNsense + Kali) | Working | [docs/00-foundation.md](docs/00-foundation.md) |
| OPNsense HA (CARP + pfsync) | Working | [docs/01-opnsense-carp.md](docs/01-opnsense-carp.md) |
| Multi-WAN failover | Working | [docs/02-multiwan-failover.md](docs/02-multiwan-failover.md) |
| Zenarmor (Free) | Working, no syslog streaming on Free tier | [docs/03-zenarmor.md](docs/03-zenarmor.md) |
| Wazuh manager + agents | Working (Windows 10, Ubuntu) | [docs/04-wazuh.md](docs/04-wazuh.md) |
| TheHive + Cortex + MISP (Docker) | Working | [docs/05-thehive-cortex-misp.md](docs/05-thehive-cortex-misp.md) |
| Wazuh → TheHive integration | Working, alerts confirmed | [docs/06-wazuh-thehive-integration.md](docs/06-wazuh-thehive-integration.md) |
| Troubleshooting log | Reference | [docs/07-troubleshooting.md](docs/07-troubleshooting.md) |

## Roadmap

- [ ] Tune alert noise (vulnerability-detector alerts flooded TheHive on first scan)
- [ ] Promote an alert to a case and enrich observables with Cortex (VirusTotal, MalwareBazaar)
- [ ] Confirm TheHive ↔ MISP link end to end
- [ ] Forward OPNsense logs to Wazuh over syslog (free alternative to Zenarmor streaming)
- [ ] Re-enable the "Block http/https bypass" LAN rules
- [ ] Add Caldera for adversary emulation

## Security notice

This is a lab on an isolated internal network. **No credentials, API keys or tokens are in this repo.** Config snippets use placeholders such as `YOUR_THEHIVE_API_KEY`.

## References

- Wazuh, TheHive, Cortex, MISP, OPNsense and Zenarmor official documentation.
- The Wazuh→TheHive script and the base Docker Compose stack started from public community repositories (the script's source is linked in its doc); everything here was adapted, debugged and tuned for this lab.
