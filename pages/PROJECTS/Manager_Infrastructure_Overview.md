# Manager Infrastructure & Operations

**Status:** ACTIVE  
**Hardware:** Ubuntu 26.04, i3-6100, 14GB RAM, 98GB disk  
**Location:** Home server (Tailscale 100.77.129.93)  
**Last Updated:** 2026-06-09

## Server Specifications
- **OS:** Ubuntu 26.04
- **CPU:** i3-6100
- **RAM:** 14GB
- **Storage:** 98GB
- **Network:** Tailscale VPN (100.77.129.93)
- **SSH:** Active (port 22, currently password-auth—needs pubkey hardening)

## Services Running
- **Hermes Gateway:** Running, connected to Telegram
- **Monitoring Stack:** Grafana, Loki, Promtail (all healthy)
- **Jellyfin:** Media streaming service (operational)
- **Docker:** All services containerized and healthy

## Backup System
- **Hardware:** Seagate 7.3TB external drive
- **Schedule:** Daily at 03:45 UTC
- **Sources:** b-terminal (Windows), brandons-z-flip5 (Android)
- **Protocol:** rsync + SSH over Tailscale
- **Rotation:** 3-generation policy
- **Shares:** SMB (seagate, manager-home) mounted at /mnt/seagate (exFAT)
- **Last successful backup:** May 30 16:36-17:04 UTC
- **Recent incident:** Power loss Jun 3 (14:00 UTC / 7 AM PST) caused unmount; drive healthy after remount; manual backup Jun 3 16:02 UTC successful
- **Next automated:** Jun 4 03:45 UTC

## Security Status
- **SSH:** Currently password-auth only (needs pubkey hardening)
- **Firewall:** Configured
- **Updates:** Current

## Known Issues & Blockers
1. **SSH pubkey authentication:** Not yet hardened (password-auth only)
   - **Priority:** Medium (internal network only, but best practice)
   - **Action:** Implement pubkey-only authentication

2. **Telegram bot token:** Invalid (prevents Sharon access)
   - **Status:** Pending token refresh from BotFather
   - **Action:** Get new token, update config.yaml, restart gateway

3. **Backup power resilience:** Recent power loss caused unmount
   - **Status:** Drive healthy; manual recovery successful
   - **Action:** Monitor next scheduled backup (Jun 4 03:45 UTC)

## Related Notes
[[Hermes_Agent_Development]]
[[System_Architecture_Overview]]
[[Brandon_Infrastructure_Profile]]
