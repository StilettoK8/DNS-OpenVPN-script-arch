# OpenVPN systemd-resolved DNS Hook

This repository contains a small Bash script used as an **OpenVPN up/down hook**
to correctly configure DNS using **systemd-resolved**.

It exists because I wanted VPN DNS to *just work* without manually fixing
`resolv.conf`, broken split-DNS, or DNS leaks after connecting.

---

## Note

> This script exists for *my specific use case*.
>
> It is meant to integrate OpenVPN with **systemd-resolved** in a clean,
> predictable way.
>
> The goal is **quick, reliable DNS setup** on VPN connect and a clean revert
> on disconnect, nothing more.
>
> It is more or less *vibe-coded with intent*: written to solve my problem,
> not to be a universal OpenVPN DNS solution.
>
> Use as inspiration unless your setup closely matches mine.

---

## What This Script Does

On VPN **connect (`up`)**:
- Reads DNS options (`dhcp-option DNS`, `DOMAIN`, `DOMAIN-SEARCH`) pushed by OpenVPN
- Applies DNS servers to the VPN interface via `resolvectl`
- Routes **all DNS queries** through the VPN interface (`~.`)

On VPN **disconnect (`down`)**:
- Reverts DNS configuration for the VPN interface
- Restores the system’s previous DNS state automatically

---

## Requirements

- Linux system using **systemd**
- `systemd-resolved` enabled and running
- OpenVPN (classic, not NetworkManager)
- `resolvectl` available in `$PATH`

---

## Script

```bash
#!/usr/bin/env bash
# OpenVPN DNS hook for systemd-resolved

set -e

IFACE="${dev}"    # e.g. tun0

DNS_SERVERS=""
SEARCH_DOMAINS=""

# Parse foreign_option_* env vars from OpenVPN
for optionname in ${!foreign_option_*}; do
    option="${!optionname}"
    set -- $option
    if [ "$1" = "dhcp-option" ]; then
        case "$2" in
            DNS)
                DNS_SERVERS="$DNS_SERVERS $3"
                ;;
            DOMAIN|DOMAIN-SEARCH)
                SEARCH_DOMAINS="$SEARCH_DOMAINS $3"
                ;;
        esac
    fi
done

case "$script_type" in
    up)
        # Set DNS servers on this interface
        if [ -n "$DNS_SERVERS" ]; then
            resolvectl dns "$IFACE" $DNS_SERVERS
        fi

        # Route all DNS queries via this interface
        resolvectl domain "$IFACE" "~."

        # Optional: mark this interface as default DNS route
        # resolvectl default-route "$IFACE" true
        ;;

    down)
        # Revert any DNS configuration for this interface
        resolvectl revert "$IFACE"
        ;;
esac

exit 0
