# UniFi Integration

Monitor your UniFi network infrastructure with Elastic Agent.

## Data streams

### `unifi.device` (metrics)

Per-device metrics polled from the **UniFi Network Local REST API** every 60 seconds.

Collected for all devices:
- Device identity: name, model, type, IP, MAC, firmware version, uptime
- Device health: state, satisfaction score, connected client count
- Uplink stats: up/down, speed, tx/rx bytes and rates, errors, drops

Collected for UDM-Pro / gateway only:
- WAN uplink latency (`metrics.unifi.device.uplink.latency_ms`)
- WAN IP, drop count, instantaneous rx/tx rate
- Last speed test results (ping, download, upload)

Collected for switches:
- Temperature, fan speed, total PoE power draw

**Requirements:**
- A UDM-Pro or compatible UniFi OS device
- A UniFi OS API key (Settings → Admins & Users → API Keys)
- Elastic Agent reachable to the UDM-Pro on port 443

### `unifi.events` (logs)

Syslog events forwarded from the UDM-Pro via UDP. Covers:
- **WiFi** — associations, disassociations, auth failures (hostapd)
- **DHCP** — lease grants, renewals, releases (dnsmasq)
- **Firewall** — iptables BLOCK/ACCEPT decisions (kernel)
- **System** — device health alerts, reboots, link state changes (mcad, ubnt-util)

**UniFi setup:** Settings → System → Logging → Enable Remote Syslog,
set the Elastic Agent host and port (default 5514).

### `unifi.netconsole` (logs)

Raw kernel/driver debug messages broadcast over UDP by UniFi APs and switches.

## Setup

1. In Kibana → Fleet → Agent Policies, add the **UniFi** integration.
2. Configure the UniFi API URL and API key for the `unifi.device` stream.
3. Configure the listen port for `unifi.events` (default 5514) and point the
   UDM-Pro remote syslog setting to your Elastic Agent host.
4. Optionally enable `unifi.netconsole` and set the listen port (default 10001).
