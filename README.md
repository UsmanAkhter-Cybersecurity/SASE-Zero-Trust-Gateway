# 🛡️ Guided Lab: Setting Up a Secure Web Gateway Sandbox

This is a practical, hands-on learning lab where I followed technical deployment instructions to understand basic secure routing principles. The goal of this project was to get comfortable using Docker and configure a system that routes outbound traffic through a secure tunnel edge.

What I learned to do in this lab:

Followed step-by-step deployment documentation to spin up containerized environments.
Configured basic network settings to route traffic through a secure egress tunnel.

---

## 📊 Live Operational Telemetry Verification

### System Telemetry Dashboard
Below is the live validation telemetry captured during operational testing, demonstrating real-time query resolution and threat mitigation:

![SASE Live Dashboard](screenshots/dashboard-telemetry.png.png)

### Key Boundary Metrics:
* **Total Network Queries Processed:** 17
* **Proactive Perimeter Interceptions:** 2 Malicious/Tracking domains blocked 
* **Aggregate Boundary Cleanliness Rate:** 11.8% of aggregate outbound traffic neutralized
* **Active Threat Intelligence Pool:** 83,068 Live security signatures loaded

---

## 🏗️ Architectural Overview & Design Pillars

The infrastructure layer runs entirely via decoupled open-source platforms and cloud-native edges, strictly adhering to zero commercial capital expenditure constraints.

1. **Network Plane Isolation:** Creates a completely segmented virtual Docker bridge network (`sase_isolated_bridge`) bound to a custom `172.20.0.0/16` subnet to ensure absolute sandbox security and prevent lateral host pivoting.
2. **Persistent Telemetry Logging:** Utilizes relative host filesystem mappings (`./../telemetry-logs/`) to guarantee that internal threat configurations, audit logs, and core databases survive systemic reset or teardown events.
3. **Hidden Egress Routing Table:** Establishes an outbound-only connection via an authenticated `cloudflared` network daemon executing a secure handshake over HTTPS (Port 443), dropping the necessity for inbound hardware port-forwarding entries.

---

## 💻 Orchestration Blueprint (`docker-compose.yml`)

The production infrastructure topology is deployed via the following multi-service configuration matrix:

```yaml
services:
  threat-interceptor:
    container_name: sase-dns-sinkhole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80/tcp"
    environment:
      TZ: 'UTC'
      WEBPASSWORD: '${PIHOLE_PASSWORD}'
      DNS1: '1.1.1.1'
      DNS2: '1.0.0.1'
    volumes:
      - './../telemetry-logs/dns-config:/etc/pihole'
      - './../telemetry-logs/dns-traffic:/etc/dnsmasq.d' 
    networks:
      sase-secure-plane:
        ipv4_address: 172.20.0.5
    restart: unless-stopped

  egress-tunnel:
    container_name: sase-cloud-tunnel
    image: cloudflare/cloudflared:latest
    command: tunnel --no-autoupdate run --token [REDACTED_SECURE_TOKEN]
    networks:
      sase-secure-plane:
        ipv4_address: 172.20.0.10
    restart: unless-stopped

networks:
  sase-secure-plane:
    name: sase_isolated_bridge
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
          gateway: 172.20.0.1
