# DNS-OpenVPN Script (Arch Linux)

This script integrates OpenVPN DNS settings with `systemd-resolved` on Arch Linux.
Its from me to me.

## Features

- Installed in `/etc/openvpn/`
- Reads DNS information pushed by an OpenVPN server
- Applies DNS settings to the VPN tunnel using `systemd-resolved`
- Forces all DNS traffic through the VPN while connected
- Automatically cleans up DNS configuration when the VPN disconnects
