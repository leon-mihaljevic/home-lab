# Networking

## LAN

- Subnet: `192.168.100.0/24`
- Router: ISP-provided (A1 Hrvatska, ZTE), no separate managed switch
- DHCP pool: roughly `.100`–`.254`
- Both VMs use **static DHCP reservations** rather than static IPs configured in-guest, so addressing stays predictable without touching each VM's own network config:
  - `nas-01` → `192.168.100.50`
  - `docker-01` → `192.168.100.60`

## Remote access — Tailscale, not port forwarding

No ports are forwarded on the router, and there's no public-facing reverse proxy, domain, or DNS entry pointing at this lab. All remote access — from a laptop or phone, on any network — goes through **Tailscale's private mesh network** instead:

- Tailscale is installed on the Proxmox host and both VMs
- Access works identically whether on the same LAN or traveling, with no change to firewall rules
- This mirrors the "no public ingress, private networking only" pattern that's the recommended default in cloud environments too (the AWS equivalent being a VPN or PrivateLink-style private connection rather than opening security groups to `0.0.0.0/0`)

## Why this setup over the alternatives

- **Static reservations over static in-guest IPs:** keeps IP assignment centralized at the router/DHCP level — rebuilding a VM from scratch doesn't require remembering to re-apply a static IP inside the guest OS.
- **Tailscale over port forwarding:** eliminates an entire class of exposure (scanners hitting an open port, needing to keep a public-facing service patched) at the cost of needing the Tailscale client on any device that wants access — an acceptable tradeoff for a single-user/family lab.

## Planned (not yet implemented)

- VLAN segmentation — separating PC, IoT, and server-infrastructure traffic onto different broadcast domains, currently everything shares the same flat `/24`
