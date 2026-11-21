# Zenarmor on OPNsense

- Installed on the Master with an **SQLite** database and the emulated **netmap** driver.
- Protects LAN (`em1`), zone `lan`.
- The engine was stopped after install and had to be started manually.
- Traffic only counts when it comes from a LAN client (for example Kali), not from OPNsense itself.

## Syslog streaming is a paid feature

Zenarmor → Settings → Reporting & Data → Streaming Data shows "Upgrade Your Subscription" on the **Free** edition. Per Zenarmor's documentation, Free and Home cannot stream to syslog; SOHO or Business is required.

## Free alternative (planned, not started)

1. OPNsense → System → Settings → Logging → Remote: UDP, `192.168.1.10`, port 514, application `filter`.
2. On the Wazuh manager, add to `/var/ossec/etc/ossec.conf` a `<remote>` block with `<connection>syslog</connection>`, port 514, protocol udp, `<allowed-ips>192.168.1.0/24</allowed-ips>`.
3. `sudo systemctl restart wazuh-manager`.
4. Verify with `sudo tail -f /var/ossec/logs/archives/archives.log` (may need `<logall>yes</logall>`).

This sends OPNsense logs only, not Zenarmor's web/app data.
