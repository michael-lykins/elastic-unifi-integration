# UniFi Integration

Monitor your UniFi network infrastructure with Elastic Agent. This integration collects
metrics and logs from a **UDM-Pro** (or any UniFi OS gateway) using the UniFi Network
Local REST API and syslog forwarding.

## Requirements

- A UDM-Pro, UDM-SE, or other UniFi OS device
- A UniFi OS API key (Settings → Admins & Users → API Keys, read-only access is sufficient)
- Elastic Agent reachable to the UDM-Pro on port 443 (REST API) and UDP 5514 (syslog)
- Kibana 8.13.0+ or 9.0.0+ and an Elastic Basic subscription or higher

## Setup

1. In Kibana → Fleet → Agent Policies, add the **UniFi** integration.
2. Under **UniFi Network API (CEL)**, enter your UDM-Pro URL and API key.
3. Under **UniFi Syslog (UDP)**, confirm the listen port (default 5514) and enable
   remote syslog on the UDM-Pro: Settings → System → Logging → Remote Syslog.
4. Enable or disable individual data streams as needed.

## Data Streams

### `unifi.device` — Device Metrics (metrics, CEL)

Per-device metrics polled from the UniFi Network REST API every 60 seconds.

**All devices:**
- Identity: name, model, type, IP, MAC, firmware version, uptime
- Health: state (connected/disconnected), satisfaction score, connected client count
- Uplink: speed, tx/rx bytes, tx/rx rates, errors, drops

**UDM-Pro / Gateway only:**
- WAN latency (`uplink.latency_ms`), WAN IP, instantaneous rx/tx rate
- Last speed test results: ping, download Mbps, upload Mbps

**Switches only:**
- Temperature (°C), fan speed (%), total PoE power draw (W)

---

### `unifi.health` — Network Health (metrics, CEL)

Per-subsystem health metrics covering WAN, LAN, and WLAN segments.

- Subsystem state, number of connected users, number of guests
- Tx/rx bytes, tx/rx rates per subsystem
- WLAN: number of APs, channels, radio utilization

---

### `unifi.client` — Client Metrics (metrics, CEL)

Per-connected-client metrics for all wired and wireless clients.

- Identity: hostname, MAC, IP, VLAN
- Signal strength (dBm), noise, SNR
- Tx/rx bytes, tx/rx rates, satisfaction score
- Association: SSID, AP name, radio band, channel

---

### `unifi.dpi` — DPI Application Usage (metrics, CEL)

Deep Packet Inspection usage statistics per client and application.

- Per-client, per-application tx/rx bytes
- Application name and category resolved from 2,120+ app IDs and 35 category IDs
  mapped directly from UniFi firmware

---

### `unifi.alarm` — Active Alarms (logs, CEL)

Active network alarms from the UniFi controller.

- Alarm key, subsystem, message
- Device reference, site, timestamps
- Covers connectivity failures, device issues, and IDS/IPS alerts

---

### `unifi.ips_event` — IPS/IDS Events (logs, CEL)

Intrusion detection and prevention events.

- Signature ID, category, severity
- Source/destination IP and port
- Action taken (drop, alert), timestamp

---

### `unifi.firewall_policy` — Firewall Policies (logs, CEL)

Firewall rule configuration collected from the UniFi Network Integration API v1.

- Rule name, action (accept/drop/reject)
- Source and destination zones
- Rule ordering index, logging state, enabled/disabled

---

### `unifi.events` — Syslog Events (logs, UDP)

System and network events forwarded from the UDM-Pro via UDP syslog (port 5514).

- **WiFi** — associations, disassociations, auth failures (hostapd)
- **DHCP** — lease grants, renewals, releases (dnsmasq)
- **Firewall** — iptables BLOCK/ACCEPT decisions
- **System** — device health alerts, reboots, link changes, SSH logins

**UniFi setup:** Settings → System → Logging → Enable Remote Syslog,
set the Elastic Agent host and port (default 5514).

---

### `unifi.netconsole` — Netconsole (logs, UDP)

Raw kernel and driver debug messages broadcast over UDP by UniFi APs and switches.
Useful for low-level hardware diagnostics.

---

### `unifi.rogue_ap` — Neighboring / Rogue APs (metrics, CEL)

Neighboring access points detected by UniFi APs during wireless scans.

- SSID, BSSID, channel, band
- Signal strength, security mode
- Flagged as rogue if MAC is not in your known device list
