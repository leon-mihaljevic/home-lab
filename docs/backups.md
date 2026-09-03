# Backups

## Strategy

- No RAID on any array in this lab — data is treated as **replaceable**, and backups are treated as what actually matters. RAID protects against a drive failure interrupting uptime; it does nothing against accidental deletion, ransomware, or a bad config push, which is what backups are actually for here.
- A dedicated 500GB HDD (`backup500`) is set aside purely as the **Proxmox backup target** — physically and logically separate from the drives serving live data, so a single drive failure can't take out both a VM's storage and its backup at once.

## What's backed up

- Proxmox's built-in scheduled backup job runs **daily at midnight**, backing up both VM disk images — `nas-01` and `docker-01` — to `backup500`.
- This captures full VM state (OS, configs, and in nas-01's case, whatever's on its attached storage), not just user files — a full VM restore is possible from any retained backup point, not just a file-level recovery.

## Philosophy: containers are disposable, data is not

The Docker stacks on `docker-01` (Portainer, Uptime Kuma, Homepage, Jellyfin) are treated as fully rebuildable — `docker compose down && docker compose up -d` should always be safe. This was **verified in practice**, not just assumed: Jellyfin was torn down and rebuilt as a deliberate test, confirming its configuration/library data (which lives in a named volume, not the container itself) survived the cycle intact.

This separation — ephemeral compute vs. persistent data — is the same principle behind containers/pods in orchestrated environments generally, and specifically mirrors how AWS separates compute (EC2/ECS, disposable) from storage (EBS/EFS/S3, persistent) as distinct concerns.

## Known gap (not yet addressed)

- No **Proxmox Backup Server** deployed — currently using Proxmox VE's built-in scheduled backup feature directly to a local disk, not PBS's deduplicated backup store. PBS would add features like better retention/pruning policies, incremental backups, and (eventually) offsite/remote sync — worth evaluating once local backup hygiene is solid.
- Retention/generation count for existing backups isn't currently tracked or enforced — worth defining an explicit policy (e.g. "keep last 7 daily + last 4 weekly") rather than letting backups accumulate indefinitely or age out ad hoc.
