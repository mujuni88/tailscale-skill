---
name: tailscale
description: >-
  Guide for installing, configuring, and managing Tailscale, headscale, and the
  Tailscale product family. Covers core mesh VPN (exit nodes, subnet routers,
  access controls, SSH, MagicDNS), Docker and Kubernetes integration, CI/CD
  pipelines, ephemeral nodes, infrastructure access, site-to-site networking,
  app connectors, Aperture (AI/LLM gateway for governance, cost control, usage
  visibility), device posture, MDM, SCIM provisioning, SSH and kubectl session
  recording, tsrecorder, Taildrop, Tailscale Serve, and Funnel. Use when someone
  asks about Tailscale networking, mesh VPN, VPN replacement, containers,
  Kubernetes operator, CI/CD runners, device management, session recording,
  audit logging, LLM API access, AI cost control, file sharing, or exposing
  internal services — even when they describe the scenario without naming the
  product.
---

# Tailscale

Tailscale is a zero-config mesh VPN that creates a secure peer-to-peer network (called a **tailnet**) between your devices. It uses WireGuard for encryption and connects devices directly rather than routing through a central gateway.

Beyond the core VPN, Tailscale offers a family of products built on the same identity and networking layer. This skill covers all of them — consult the reference file for whichever topic is relevant.

## How references work

References fall into two shapes depending on what the skill needs to do for the user.

**Descriptive references** (most files — `aperture.md`, `containers.md`, `enterprise.md`, `device-management.md`, `session-recording.md`, `api.md`) help you *describe* a topic to a user: explain it, draft config, recommend an approach. These use a hybrid layout — stable mental model and load-bearing config shapes inline, plus a curated list of canonical `tailscale.com/docs/...` URLs to **WebFetch for current detail**. Follow the in-file instructions about when to fetch. When WebFetch is available, prefer the live page over the inline summary for specifics (config keys, env vars, supported models, pricing). When WebFetch is unavailable, answer from inline content and tell the user which doc page to consult.

**Operational references** (currently `cli.md`) help you *operate* a tool on the user's machine — Claude actually invokes the commands. These keep concrete command/flag content inline because wrong flags break real systems. The fallback is *local*, not network: run `tailscale help <subcommand>` to verify a flag before suggesting it. The canonical docs URL is the second fallback, for when the tool isn't installed.

The remaining references (`access-control.md`, `common-tasks.md`, `connectivity.md`, `exit-nodes.md`, `installation.md`, `subnet-routers.md`) are smaller and self-contained — read the file, answer the question.

## Core concepts

- **Tailnet**: Your private network of authenticated devices and users.
- **WireGuard**: The encryption protocol underneath Tailscale. Key management is automatic.
- **MagicDNS**: Automatic DNS names for every device (e.g., `ssh my-server`).
- **100.x.y.z addresses**: Each device gets a stable Tailscale IP in the CGNAT range.
- **Tailnet policy file**: JSON config in the admin console that defines access controls, groups, tags, and SSH rules. Deny-by-default.

## Authoring defaults

When the user asks you to write or edit a tailnet policy file:

- **Use grants, not ACLs.** Grants are Tailscale's recommended way to express access rules — they cover what ACLs do (network-layer access) plus application-layer capabilities (Kubernetes, Aperture, tsrecorder, Taildrive) in one form. ACLs are still supported for reading existing policies and migrations, but every new access rule you write should be a grant. See `references/access-control.md` for the conversion pattern and https://tailscale.com/docs/reference/grants-vs-acls for the canonical comparison.
- **The grant-vs-ACL choice only applies to access rules.** Other policy-file sections have their own dedicated syntax and aren't grants: `"ssh"` (SSH access), `"autoApprovers"` (auto-approving advertised routes and exit nodes), `"nodeAttrs"` (node-level attributes like Funnel), `"postures"` (device posture definitions, referenced from grants via `srcPosture`), and the `"groups"`/`"tagOwners"` definitions.

## Quick start

```bash
curl -fsSL https://tailscale.com/install.sh | sh   # Install (Linux)
sudo tailscale up                                    # Connect
tailscale status                                     # Verify
```

For macOS/Windows, download from https://tailscale.com/download. For mobile, use the app stores.

## Topic index

Read the reference file that matches the user's question. Each file is self-contained.

### Networking & connectivity

| Topic | Reference file | When to read |
|-------|---------------|--------------|
| Installation | `references/installation.md` | Installing Tailscale on any platform, updating, uninstalling |
| Exit nodes | `references/exit-nodes.md` | Routing all internet traffic through a device (VPN-style), travel security |
| Subnet routers | `references/subnet-routers.md` | Reaching devices that can't run Tailscale (printers, cameras, cloud VPCs) |
| Access control | `references/access-control.md` | Grants, ACLs, tags, groups, policy file structure |
| Connectivity | `references/connectivity.md` | Peer relay, DERP servers, NAT traversal, tailnet lock, connection types |
| Common tasks | `references/common-tasks.md` | Tailscale SSH, MagicDNS, auth keys, key expiry |

### Sharing & publishing

| Topic | Reference file | When to read |
|-------|---------------|--------------|
| Sharing & publishing | `references/sharing-and-publishing.md` | Taildrop (file transfer), Taildrive (persistent folder sharing), Tailscale Serve (private), Tailscale Funnel (public) |

### Containers & orchestration

| Topic | Reference file | When to read |
|-------|---------------|--------------|
| Docker & Kubernetes | `references/containers.md` | Running Tailscale in Docker containers, sidecar pattern, Docker Compose, Kubernetes operator, cluster ingress/egress, Connector CRD, ProxyGroup |

### Enterprise & infrastructure

| Topic | Reference file | When to read |
|-------|---------------|--------------|
| Enterprise patterns | `references/enterprise.md` | VPN replacement, infrastructure access, ephemeral nodes, CI/CD integration (GitHub Actions), site-to-site networking, app connectors, auth keys for automation, Terraform provider |
| Device management | `references/device-management.md` | Device approval, device posture, MDM deployment, SCIM user/group provisioning, bulk device operations, enterprise rollout |
| Session recording | `references/session-recording.md` | tsrecorder setup, SSH session recording, S3 storage, Kubernetes kubectl recording, API request recording, failover, audit compliance |

### AI & LLM governance

| Topic | Reference file | When to read |
|-------|---------------|--------------|
| Aperture | `references/aperture.md` | AI gateway, LLM request routing, API key centralization, usage visibility, cost control, quotas, coding agent integration, MCP proxying |

### Reference

| Topic | Reference file | When to read |
|-------|---------------|--------------|
| CLI | `references/cli.md` | Tailscale CLI commands, flags, serve/funnel, file transfer, diagnostics, tailnet lock |
| API | `references/api.md` | Tailscale REST API, authentication, device management, DNS, policy file, webhooks |

## CLI quick reference

| Command | What it does |
|---------|-------------|
| `tailscale up` | Connect to your tailnet |
| `tailscale down` | Disconnect |
| `tailscale status` | Show connected devices |
| `tailscale ip` | Show your Tailscale IP addresses |
| `tailscale ping <host>` | Test connectivity to a device |
| `tailscale set --ssh` | Enable Tailscale SSH on this device |
| `tailscale set --advertise-exit-node` | Advertise as an exit node |
| `tailscale set --exit-node=<ip>` | Use a specific exit node |
| `tailscale set --advertise-routes=<cidr>` | Advertise subnet routes |
| `tailscale set --accept-routes` | Accept advertised subnet routes (Linux) |
