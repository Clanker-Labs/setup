# Changelog

## Unreleased

### Added
- `clanker-fleet` Claude skill under `.claude/skills/` — how machines reach shared services, wiring apps to a local or remote LeHarness, and the Tailscale-only networking rules.
- `leharness.configure.presets` passes a resident model set through to `leharness configure --presets` (Ollama keeps several models loaded, one alias each).
- Optional non-interactive LeHarness configuration pass-through for engine,
  topology, model preset, cluster addresses, and Tailscale gateway binding.
- Target-user/path-aware `leharness.service` and `leharness-dashboard.service`
  templates ordered after Docker and Tailscale.

### Fixed
- LeHarness dashboard docs/tasks no longer mention an admin login token; Tailscale is the gate.
- LeHarness now tracks its real `master` default branch instead of nonexistent
  `main`.
- Provisioning rechecks and fails on unresolved LeHarness prerequisites rather
  than silently reporting success.
- Playbook layout tests now include the wired `leharness` role.
