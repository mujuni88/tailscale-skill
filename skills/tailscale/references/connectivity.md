# Connectivity: Peer Relay, DERP, and Tailnet Lock

This reference covers how Tailscale connections are established (direct vs relayed), the DERP network, peer relays (user-operated relays), and Tailnet Lock (cryptographic node signing).

> The Tailscale connection model is stable, but the specifics — peer-relay flags, DERP regions list, Tailnet Lock CLI subcommands — evolve. The shapes below are oriented toward the **why** and the **what to configure**; **WebFetch the matching page** for current flag names, region IDs, and step-by-step Tailnet Lock setup.

## Mental model

Tailscale tries three connection paths in order, all WireGuard-encrypted end-to-end:

1. **Direct peer-to-peer** — preferred. NAT traversal (STUN, port mapping) establishes a direct tunnel. Lowest latency, full throughput.
2. **Peer relay** — fallback through a user-operated relay device on the tailnet. Lower latency than DERP because the relay sits on your infrastructure.
3. **DERP relay** — final fallback through Tailscale's global relay network. Always works.

All relays forward **encrypted** packets blindly — relays (peer or DERP) cannot decrypt traffic. The choice of path is per-peer-pair, not tailnet-wide.

**NAT type matrix:**

| Peer A | Peer B | Result |
|---|---|---|
| No NAT / Easy NAT | Any | Direct |
| Hard NAT | Easy NAT | Direct (via traversal) |
| Hard NAT | Hard NAT | Relayed (peer relay or DERP) |

"Easy NAT" = UPnP / NAT-PMP / PCP support. "Hard NAT" = symmetric NAT, CGNAT, or strict firewalls.

DERP also serves a second role: **connection negotiation**. Even direct connections use DERP briefly to exchange discovery (DISCO) packets before switching to direct.

## Canonical shapes

### Configure a peer relay

On the device that will relay (Linux/macOS/Windows — not iOS/Android):

```bash
tailscale set --relay-server-port=<port>
```

The port must be reachable from devices that will use this relay (public IP, or port-forwarded).

Then in the tailnet policy file, grant relay capability:

```json
"grants": [{
  "src": ["autogroup:member"],
  "dst": ["tag:relay"],
  "app": {
    "tailscale.com/cap/relay": [{}]
  }
}]
```

Tag your relay devices with `tag:relay` (or whatever tag you used in the grant).

### Customize the DERP map

In the tailnet policy file, you can add custom DERP regions or omit defaults:

```json
"derpMap": {
  "OmitDefaultRegions": false,
  "Regions": {
    "900": {
      "RegionID": 900,
      "RegionCode": "myderp",
      "RegionName": "My Custom DERP",
      "Nodes": [{
        "Name": "myderp1",
        "RegionID": 900,
        "HostName": "derp.example.com"
      }]
    }
  }
}
```

The official DERP map (with current region IDs) is at `https://controlplane.tailscale.com/derpmap/default`. **Running your own DERP** is generally not recommended — peer relays solve the latency problem more simply and don't lose access to device sharing or cross-tailnet features.

### Tailnet Lock — initialize and operate

Tailnet Lock prevents unauthorized nodes from joining the tailnet even if the coordination server were compromised. Every new node must be signed by an existing trusted device.

Conceptual pieces:
- **Tailnet Lock Key (TLK)** — Ed25519 key pair on a signing node; admins designate which are trusted.
- **Tailnet Key Authority (TKA)** — local signed chain (think git) tracking trusted TLKs and signed node keys.
- **Authority Update Message (AUM)** — signed message that modifies trusted-key state.
- **Disablement secrets** — 10 strings generated at init; the **only** way to disable Tailnet Lock if needed. **Store them in a safe / password manager.** Losing them means the tailnet cannot be recovered without Tailscale support.

Core CLI flow (full setup is admin-console-driven):

```bash
tailscale lock init                                # On a chosen signing node
tailscale lock sign nodekey:<key> tlpub:<key>      # Sign a new device's join
tailscale lock add tlpub:<key>                     # Add a trusted signing key
tailscale lock remove tlpub:<key>                  # Remove one
tailscale lock revoke-keys tlpub:<key>             # Revoke compromised key (needs co-signing)
tailscale lock status                              # Inspect TKA state
tailscale lock log                                 # Recent AUMs
tailscale lock disable <secret>                    # Disable using a recovery secret
tailscale lock local-disable                       # Emergency: ignore TL on this node only
```

**Constraints to remember:**
- Up to 20 signing nodes.
- Rotate TLKs at most once per year (TKA growth bound).
- **Mutually exclusive with Device Approval** — pick one.
- Android devices can receive signatures but cannot sign.
- Initial trust is "trust on first use" from the coordination server — verify `tailscale lock status` on multiple nodes after init.

## Where to find current information

### Connection types & how Tailscale connects

| User is asking about… | Fetch |
|---|---|
| Direct vs relayed connection — full taxonomy | https://tailscale.com/docs/reference/connection-types |
| Device connectivity overview | https://tailscale.com/docs/reference/device-connectivity |
| How traffic routes through Tailscale | https://tailscale.com/docs/concepts/traffic-routing-through-tailscale |
| WireGuard background | https://tailscale.com/docs/concepts/wireguard |
| Encryption model | https://tailscale.com/docs/concepts/tailscale-encryption |
| STUN, port mapping, NAT traversal mechanics | https://tailscale.com/docs/reference/stun-protocol |
| WireGuard with dynamic IPs | https://tailscale.com/docs/reference/wireguard-dynamic-ip |

### DERP

| Topic | Fetch |
|---|---|
| DERP servers — purpose, regions, custom DERP | https://tailscale.com/docs/reference/derp-servers |
| Troubleshooting DERP routing | https://tailscale.com/docs/reference/troubleshooting/network-configuration/derp-routing |
| Client message: no DERP connection | https://tailscale.com/docs/reference/messages/client/no-derp-connection |
| Client message: no DERP home | https://tailscale.com/docs/reference/messages/client/no-derp-home |
| Coordination server down | https://tailscale.com/docs/reference/coordination-server-down |
| Coordination-server-issue client message | https://tailscale.com/docs/reference/messages/client/coordination-server-issue |

### Peer relay

| Topic | Fetch |
|---|---|
| Peer relay overview, setup, platform support | https://tailscale.com/docs/features/peer-relay |

### Tailnet Lock

| Topic | Fetch |
|---|---|
| Tailnet Lock — full setup + concepts | https://tailscale.com/docs/features/tailnet-lock |
| Whitepaper (cryptographic design) | https://tailscale.com/docs/concepts/tailnet-lock-whitepaper |

### Connectivity troubleshooting

| Topic | Fetch |
|---|---|
| Hard NAT issues | https://tailscale.com/docs/reference/troubleshooting/network-configuration/hard-nat-issues |
| CGNAT conflicts (with 100.64/10 ranges) | https://tailscale.com/docs/reference/troubleshooting/network-configuration/cgnat-conflicts |

## Answering pattern

For **"why is my connection slow / relayed"** questions, the mental model + NAT matrix + `tailscale ping`/`netcheck` output (see `references/cli.md`) is usually enough to diagnose. Fetch the connection-types or troubleshooting pages only when you need exact criteria (e.g., what counts as Easy NAT for a specific carrier).

For **peer-relay setup** questions, the inline shape (flag + grant) is enough to start. Fetch the peer-relay page for platform-specific notes and edge cases.

For **Tailnet Lock**, the inline concepts (TLK, TKA, AUM, disablement secrets) are stable. **Always fetch** when the user is about to enable it for real — initial setup has admin-console steps and irreversibility risk (lost disablement secrets) that justify reading the live page.

For **DERP custom deployment**, recommend against it by default; fetch the DERP-servers page if the user has a strong reason and needs the build/operate steps.
