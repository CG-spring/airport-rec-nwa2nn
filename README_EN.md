# Proxy Service Field Manual: Subscriptions, Clients & Troubleshooting

> A hands-on manual for getting the most out of your subscription-based proxy service ("airport"). Covers subscription formats, client configuration, split routing, traffic management, performance tuning, and troubleshooting. Complements our selection guide by focusing on *how to use it well*.

## Table of Contents

- [1. Introduction](#1-introduction)
- [2. Subscriptions: Getting & Converting](#2-subscriptions-getting--converting)
- [3. Client Configuration, Step by Step](#3-client-configuration-step-by-step)
- [4. Rules & Split Routing](#4-rules--split-routing)
- [5. Traffic Management & Billing](#5-traffic-management--billing)
- [6. Performance Tuning](#6-performance-tuning)
- [7. Troubleshooting Quick Reference](#7-troubleshooting-quick-reference)
- [8. Multi-Device Strategies](#8-multi-device-strategies)
- [9. Advanced Playbook](#9-advanced-playbook)
- [10. FAQ](#10-faq)
- [11. Security Hardening Checklist](#11-security-hardening-checklist)
- [12. Disclaimer & License](#12-disclaimer--license)

## 1. Introduction

Many users blame their provider for a poor experience when the real problem is how they use the service. This manual walks through the complete usage chain: subscription → client → routing → traffic → performance → troubleshooting.

### 1.1 The Full Workflow

```
Purchase plan → Get subscription URL → Import into client → Update nodes
    → Pick a node → Enable proxy → Verify → Daily use → Monitor quota → Renew
```

### 1.2 Common Misconceptions

- **"More nodes = better."** Most users rely on 3–5 nodes daily. Quality beats quantity;
- **"Global mode is faster."** It routes domestic traffic overseas — slower and wasteful;
- **"Lower latency = faster."** Latency is handshake speed; throughput depends on bandwidth and route load;
- **"No need to update the client."** Kernels evolve quickly; old versions may lack new protocols. Always update.

## 2. Subscriptions: Getting & Converting

### 2.1 Anatomy of a Subscription URL

```
https://example.com/api/v1/client/subscribe?token=xxxxx
```

The `token` parameter is an encrypted account identifier carrying your plan, quota, and expiry. **Never share this link.**

### 2.2 Common Subscription Formats

| Format | Prefix | Typical clients |
|--------|--------|-----------------|
| SS | `ss://` (base64) | Shadowsocks family |
| V2Ray | `vmess://` / `vless://` | V2Ray family |
| Trojan | `trojan://` | Trojan clients |
| Clash | YAML config | Clash family |
| Surge | Dedicated format | Surge / Stash |

### 2.3 Subscription Conversion

When a provider only offers a generic subscription but your client needs another format:

```bash
# Local subconverter via Docker
docker run -d --name subconverter -p 25500:25500 tindy2013/subconverter:latest

# Example conversion: Clash → Surge
# https://localhost:25500/sub?target=surge&url=YOUR_SUBSCRIPTION_URL
```

**Notes**: prefer local/self-hosted converters to avoid leaking node info; verify node count and names after conversion; re-convert periodically since node lists change.

## 3. Client Configuration, Step by Step

### 3.1 Clash Verge Rev (Windows / macOS / Linux)

1. Install from official GitHub Releases;
2. Open the app → Profiles → New Profile;
3. Paste the subscription URL, name it (e.g., "Primary"), save;
4. Refresh the profile and wait for nodes to load;
5. In the Proxies tab, select a proxy group and node;
6. Enable System Proxy, or turn on TUN Mode;
7. Verify connectivity in your browser.

### 3.2 Shadowrocket (iOS)

1. Install Shadowrocket from the App Store;
2. Tap "+" in the top-right, choose type "Subscribe";
3. Paste the URL and save;
4. Open the subscription, pick a node;
5. Flip the master switch; approve the VPN configuration prompt;
6. Enable auto-update under Settings → Subscription.

### 3.3 v2rayN (Windows)

1. Download v2rayN plus a kernel (v2ray-core / Xray-core);
2. Extract and launch; point v2rayN to the kernel path on first run;
3. Server → Add subscription → enter URL → update;
4. Select a node and set it active;
5. Enable system proxy (PAC or on-demand).

### 3.4 Clash Meta for Android

1. Install Clash Meta for Android (or Mihomo Party);
2. Profiles → New Profile → Import from URL;
3. Paste the URL, save, and activate the profile;
4. Tap the master switch; grant the VPN permission;
5. Choose a node; keep Rule mode on.

## 4. Rules & Split Routing

### 4.1 Why Split Routing Matters

Directing domestic traffic straight and overseas traffic through the proxy:

- **Lower latency** for domestic sites;
- **Saves quota** — domestic traffic doesn't count;
- **More stable** — less node load, fewer throttle events;
- **Better privacy** — domestic services connect directly.

### 4.2 Rule Examples (Clash)

Rules match top-down; the first match wins:

```yaml
rules:
  # Domestic direct
  - GEOIP,CN,DIRECT
  - DOMAIN-SUFFIX,baidu.com,DIRECT
  - DOMAIN-SUFFIX,taobao.com,DIRECT
  # Streaming via proxy
  - DOMAIN-SUFFIX,youtube.com,PROXY
  - DOMAIN-SUFFIX,netflix.com,PROXY
  # Ad blocking
  - DOMAIN-KEYWORD,adservice,REJECT
  # Fallback
  - MATCH,PROXY
```

### 4.3 Rule vs. Global vs. Direct

| Mode | Behavior | When to use |
|------|----------|-------------|
| Rule | Per-rule routing | Daily use (recommended) |
| Global | Everything through proxy | Need a fixed exit IP |
| Direct | Everything direct | Temporary troubleshooting |

## 5. Traffic Management & Billing

### 5.1 The Big Consumers

- **Streaming**: 1080p ≈ 1.5 GB/hour; 4K ≈ 6 GB/hour;
- **OS updates**: Windows/macOS updates can reach several GB;
- **Cloud sync**: iCloud/OneDrive background sync is easily forgotten;
- **P2P**: often restricted or excluded — but still burns bandwidth.

### 5.2 Saving Quota

- Cap video quality at 1080p or auto;
- Disable background autoplay and auto-updates for apps;
- Route domestic apps directly (split routing handles this);
- Watch the client's live traffic stats; investigate unusual spikes;
- Use per-app proxying so only necessary apps go through the tunnel.

### 5.3 Billing Rules to Confirm Before Buying

- Quota reset cycle: calendar month vs. rolling 30 days;
- Overage policy: throttled / suspended / metered top-ups;
- Concurrent device limit;
- Expiry notice — avoid sudden disconnects.

## 6. Performance Tuning

### 6.1 Lower Latency

- Pick geographically close nodes (HK/JP for mainland users);
- Use dedicated lines (IEPL/IPLC) to avoid peak congestion;
- Enable TCP Fast Open if the kernel supports it;
- Prefer wired connections or 5 GHz Wi-Fi.

### 6.2 Raise Throughput

- Choose nodes with ample bandwidth (check provider speedtests);
- Avoid heavy tasks during peak hours (20:00–24:00);
- Split large downloads across time slots;
- Test alternative protocols (Hysteria2 shines on lossy links).

### 6.3 Client Tweaks

```yaml
profile:
  store-selected: true        # remember node choice
  store-fake-ip: true         # cache fake-ip mappings
sniffer:
  enable: true                # traffic sniffing for accurate routing
tcp-concurrent: true          # multiplexing for many connections
```

## 7. Troubleshooting Quick Reference

| Symptom | Check | Fix |
|---------|-------|-----|
| All nodes fail | Account expired / quota exhausted | Renew or reset |
| One node fails | Try another node | Node issue — wait or switch |
| Connects but no internet | System proxy on? | Enable proxy/TUN |
| Pages won't load | DNS settings | Use client built-in DNS |
| Very slow | Node/protocol | Dedicated line or Hysteria2 |
| Gaming disconnects | Packet loss | Gaming line / wired network |
| Subscription update fails | Link & network | Regenerate in panel |
| Device limit reached | Panel online devices | Kick idle devices |
| DNS errors | DNS leak | Enable DNS protection |
| WeChat/QQ issues | Rule mode off | Ensure domestic direct |

### Golden Troubleshooting Order

1. **Switch nodes** — rules out a single bad node;
2. **Switch protocols** — rules out compatibility issues;
3. **Check locally** — network, firewall, client settings;
4. **Contact support** — provide node, time, and symptoms.

## 8. Multi-Device Strategies

### 8.1 Allocating Limited Device Slots

Most plans allow 3–5 concurrent devices:

- Keep primary devices (laptop, phone) always connected;
- Connect secondary devices (tablet, TV) on demand;
- Use router-level proxying to consume just one slot.

### 8.2 Router-Based Whole-Home Proxy

```bash
# OpenClash on OpenWrt (example)
opkg update
opkg install luci-app-openclash
# Import your subscription into OpenClash; enable auto-update
```

Pros: every device (TV, consoles) is covered; configure once; DNS optimization at router level.

### 8.3 Advanced Home Networking

- Main router + sidecar router: the sidecar handles the proxy;
- x86 soft routers handle heavy multi-device loads;
- Enable "proxy overseas traffic only" so domestic usage is unaffected.

### 8.4 Family/Team Sharing

- Assign devices to fixed members to avoid kick-offs;
- Schedule heavy tasks off-peak;
- One router proxy covers the whole household;
- Some providers offer team plans cheaper than multiple individual ones.

## 9. Advanced Playbook

### 9.1 Self-Hosted Fallback

Deploy Xray on a VPS with a one-click script:

```bash
# 3x-ui panel one-click install
bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh)
```

Combine: airport as primary, self-hosted as fallback for redundancy.

### 9.2 Subscription Backup & Local Management

- Export subscriptions to local files periodically;
- Manage multiple subscriptions locally with sub-store, then distribute to clients;
- Schedule automatic subscription refreshes to avoid stale nodes.

### 9.3 Diagnostic Tools

| Tool | Purpose |
|------|---------|
| ping / mtr | Route tracing & packet loss |
| iperf3 | Bandwidth testing |
| dig / nslookup | DNS diagnostics |
| traceroute | Network bottleneck location |
| Wireshark | Packet capture (advanced) |

### 9.4 Automation

Automate routine operations with scripts:

```bash
# Refresh Clash subscription daily at 3 AM (crontab)
0 3 * * * /usr/local/bin/clash-update-sub.sh
```

```bash
# Node health check with auto-switch (sketch)
#!/bin/bash
for node in $(clash list-nodes); do
  if clash test-node "$node" --timeout 2s >/dev/null; then
    echo "$node OK"
  fi
done
```

Principles: auto-refresh subscriptions, auto-detect dead nodes, push alerts (e.g., via a Telegram bot).

## 10. FAQ

**Q1: How is provider quota different from local traffic?** In rule mode the client separates proxied vs. direct traffic; providers only count proxied traffic. Small discrepancies between panel and client are normal — trust the panel.

**Q2: Why is it slow after 8 PM?** Evening peak congestion is universal. Switch to a dedicated line, try Hysteria2, or move heavy tasks to late night.

**Q3: Can I use my subscription on a new phone?** Yes — the link binds to your account, not a device. Mind the device limit; remove old devices in the panel.

**Q4: Does the provider see my browsing history?** Depends on their logging policy. Reputable providers state "no logs" or minimal operational logs. For sensitive activity, self-host.

**Q5: Laptop and phone at the same time?** Fine within the device limit — e.g., laptop + phone + router proxy fits a typical 3-device plan.

**Q6: Subscription domain blocked?** Some providers offer backup domains; contact support. Alternatively download the config as a local file (refresh regularly).

**Q7: How do I know a node is blocked?** One node times out while others work — its IP is likely blocked. Wait for an IP rotation or switch nodes.

**Q8: Does quota reset on renewal?** Most providers reset to zero on expiry; some allow rollover. Confirm before buying.

**Q9: Can I use it overseas to access CN platforms?** Yes — pick a domestic relay node. Note that return lines may be billed separately.

**Q10: What if the provider disappears?** Start with monthly plans and avoid long pre-pays. Chargebacks may work but aren't guaranteed — start small.

## 11. Security Hardening Checklist

- [ ] Never share subscription links or submit them to unknown online tools
- [ ] Enable DNS protection in the client
- [ ] Keep banking/payments off the proxy
- [ ] Review connected devices regularly
- [ ] Keep the client and kernel up to date
- [ ] Reset the subscription immediately if you suspect leakage
- [ ] Comply with local laws; use for lawful purposes only

## 12. Disclaimer & License

**Disclaimer**: This repository is for technical learning and exchange only and does not constitute a recommendation of any service. Users are responsible for assessing risks and complying with local laws. The author assumes no liability for any consequences of using this content.

**License**: [MIT License](LICENSE). Contributions welcome.

---

*Last updated: August 2026*
