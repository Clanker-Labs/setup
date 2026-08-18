---
name: clanker-fleet
description: Wire machines in the Clanker-Labs ecosystem to shared services — pointing apps at a local or remote LLM served by LeHarness, provisioning the leharness role, and the Tailscale-only networking rules the fleet follows. Use when connecting an app or another host to the LLM server, enabling LeHarness on a machine, or reasoning about how the machines reach each other.
---

# Wiring the Clanker-Labs fleet

Machines are provisioned by this repo (Ansible) and reach each other over
**Tailscale only**. LLM serving lives in `Clanker-Labs/LeHarness`, which this
repo installs rather than reimplements.

## Enabling LeHarness on a machine

Off by default — serving pulls multi-GB weights. In your inventory/config:

```yaml
leharness:
  enabled: true
  destination: "~/apps/LeHarness"
  version: master          # note: the default branch is master, not main
  systemd: true
  dashboard: true
  start: false             # true downloads weights during provisioning
  configure:
    enabled: true
    engine: vllm           # vllm | ollama | llamacpp
    topology: single       # single | head | worker
    preset: qwen3-coder-30b
    bind: tailscale
```

To keep **several models resident at once**, use `presets` instead of `preset`
(Ollama only — vLLM and llama.cpp serve one model per process):

```yaml
  configure:
    engine: ollama
    presets: [qwen3-next-80b, qwen3-coder-30b, qwen3-vl-8b, qwen3-8b-fast]
```

Each preset contributes its alias, so that machine serves `chat`, `coder`,
`vision` and `fast` at one base URL. Sizes are checked before anything is
pulled.

The role clones the repo and calls its own `detect` / `configure` / `up`
scripts. **Do not reimplement hardware detection or engine selection here** —
that logic belongs to LeHarness and is deliberately called, not copied.

## Pointing an app at a model

Apps read the canonical `.env`. For a LeHarness on the same machine:

```
LLM_API_BASE=http://127.0.0.1:8700/v1     # local install; port may be remapped
LLM_MODEL=openai/chat
LLM_API_KEY=
```

For a **remote** harness (e.g. the DGX Spark) use its Tailscale MagicDNS name,
not a raw IP — the name survives the node changing address:

```
LLM_API_BASE=http://<machine>.<tailnet>.ts.net:8000/v1
LLM_MODEL=openai/chat
LLM_API_KEY=
```

Then `make configure`. Containers on the shared `ecosystem` docker network reach
a local harness as `http://leharness:8000/v1`.

**Get the real values rather than guessing**, since bind, port and aliases are
all configurable:

```bash
leharness status --json            # on the box
curl -s http://<host>:8701/api/models   # from anywhere on the tailnet
```

## Local vs remote LeHarness — do not conflate them

A machine can run its own harness *and* use another one. Ports differ per
machine (a local install is often remapped off 8000 when another app owns that
port). When documenting or configuring, always say which host you mean.

## Networking rules

- **Nothing binds `0.0.0.0`.** Tailscale reachability is the access control.
  LeHarness refuses an all-interfaces bind outright.
- There is **no login and no API key** on the harness. Send any non-empty string
  where a client demands one. Do not add an auth scheme to compensate; do not
  publish these URLs off the tailnet.
- Prefer the **MagicDNS name** over an IP in anything you write down.
- Engine debug ports (vLLM 8001, Ollama 11434, llama.cpp 8002) stay on loopback.
  Only the gateway and the dashboard are published to the tailnet.

## Reading a remote harness's telemetry

The dashboard serves JSON on port 8701 (`/api/status`, `/api/series`,
`/api/models`, `/api/usage`, `/api/events`). Treat a remote harness as
**read-only**: never `POST /api/control` from another machine — it is
administered on its own box.

Poll server-side and cache; degrade to "unreachable" when the host is asleep
rather than breaking the page. The harness's own poller runs every 15s, so
polling faster re-reads the same samples.

## Privacy invariant

LeHarness stores request **metadata** only — never prompt or completion bodies.
Anything the fleet builds on top of it must preserve that.
