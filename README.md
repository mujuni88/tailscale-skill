# Tailscale skill for Claude Code

A Claude Code skill that gives Claude deep knowledge of Tailscale, headscale, and the wider Tailscale product family — from setting up an exit node on your home server to wiring up Aperture for LLM API governance, the Kubernetes operator, tsrecorder, and everything in between.

> [!NOTE]
> Early alpha. This skill is not yet officially supported. Please don't file Tailscale support tickets for issues you hit with it, and treat its output as a starting point rather than a final answer. Configuration syntax in particular evolves; the skill links out to canonical [Tailscale documentation](https://tailscale.com/docs) for the parts that change most often.

## Install the skill

To install the skill, use `git` to clone this repository and make it available to your LLM.

For example, if you are using Claude Code, you can make the skill available to every session using the following command:

```bash
git clone https://github.com/tailscale/tailscale-skill ~/.claude/skills/tailscale
```

Claude Code auto-discovers anything under `~/.claude/skills/<name>/` containing a `SKILL.md`. After cloning, the skill is available in every session.

To install only in your project, clone the repository into the `.claude/skills/tailscale` directory within your project.

```bash
cd ~/your-project
git clone https://github.com/tailscale/tailscale-skill .claude/skills/tailscale
```

For other LLMs, clone the repository into the location the LLM uses for skills.


## Using the Tailscale skill

You activate the skill two ways:

- **Explicit:** type `/tailscale` followed by your question. This always invokes the skill when installed.
- **Implicit:** ask your LLM a Tailscale question conversationally, like "how do I set up a subnet router for my home network?", and the LLM consults the skill if the question matches the skill's description.

The implicit path depends on your LLM's judgment and isn't 100% reliable. When in doubt, use the `/tailscale` slash command.

## What's in the skill

`SKILL.md` is a topic index that routes the LLM to focused reference files in the `references/` directory. Topics include:

- **Core mesh VPN:** exit nodes, subnet routers, MagicDNS, grants (and legacy ACLs), tagging
- **Connectivity diagnostics:** DERP, NAT traversal, peer relay, `tailscale netcheck` / `ping` / `status`
- **Sharing & publishing:** Taildrop, Taildrive, Tailscale Serve (private), Tailscale Funnel (public)
- **Containers & Kubernetes:** the operator, Docker sidecar pattern, ProxyGroup, Connector CRD, API server proxy
- **Enterprise rollout:** SCIM provisioning, MDM (Jamf/Intune), device posture, device approval, auth keys, SSO
- **Session recording:** `tsrecorder` setup, S3 destinations, fail-closed enforcement, SSH and Kubernetes recording
- **Aperture:** AI gateway for centralizing LLM API keys, quotas, per-user usage dashboards, coding-agent integration
- **CLI:** every `tailscale` subcommand with flags and examples
- **REST API:** authentication, device management, policy file, webhooks

Each reference file is self-contained. The LLM should only load the relevant references to answer your question.

## Issues and feedback

Open an issue at https://github.com/tailscale/tailscale-skill/issues. This is the place for skill-specific problems (a reference is wrong, Claude gives bad advice for X, the install instructions don't work on Y). For Tailscale product questions unrelated to the skill, use the regular Tailscale support channels.
