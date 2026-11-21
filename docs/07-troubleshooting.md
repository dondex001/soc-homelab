# Troubleshooting Log

## Ubuntu could not download anything over HTTPS (wget showed `opnsense-webca`, 403, then timeouts)

**Root cause:** two manual **Destination NAT** rules (Firewall → NAT → Destination NAT → Interface rules) redirected all LAN port 80/443 traffic to `127.0.0.1` on Squid's ports. They were independent of the "Enable Transparent HTTP proxy" checkbox and survived reboots.

**How it was found:** enable logging on the LAN rules, then Firewall → Log Files → Live View filtered on the client IP showed every connection going to `127.0.0.1:3129`.

**Fix:** disable both NAT rules and apply.

Ruled out first: IPv6, DNS, routing, Squid SSL inspection, the "Block bypass" rules.

## OPNsense GUI slow or unresponsive

- `configd` crashing: `service configd restart` (sometimes twice) and `service lighttpd restart`.
- Near-zero free RAM: raise the Master from 1 GB to 2 GB.
- Host RAM pressure: disable background apps (for example BlueStacks services) in Startup Apps.

## Windows client lost its lease after the firewall was power-cycled

`ipconfig /release` then `ipconfig /renew`. Check `ipconfig /all` for the DHCP server `192.168.1.1`.

## Wazuh manager unreachable (SSH timeout)

The static IP was lost and the VM fell back to DHCP. On its console: `ip a`, then `sudo service network restart`.

## `dpkg` "frontend lock is locked"

Find the holder with `sudo fuser -v /var/lib/dpkg/lock-frontend` (usually `unattended-upgrades`), stop it, remove stale locks, then `sudo dpkg --configure -a`.

## "No such file or directory" on a file that exists

Check the script's first line (shebang). It may point to an interpreter that does not exist, as with Wazuh's bundled `pip3`.

## Terminal quirks

- Shared clipboard turned `|` into `\` and `~` into `-`. Use SSH from another machine and right-click to paste, or type short single-line commands.
- Always check the shell prompt: diagnostics were run on `wazuh-server` instead of `ubuntu-server` more than once.

## Cassandra timeouts in TheHive

At 3 GB RAM the Ubuntu VM swapped heavily; Cassandra GC pauses caused `WriteTimeoutException`. Fix: 4 GB RAM and a smaller Cassandra heap.

## Docker disk full

The LVM root volume was smaller than the virtual disk: `lvextend -l +100%FREE` then `resize2fs`.
