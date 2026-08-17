# Changelog

## Unreleased

### Added
- Optional non-interactive LeHarness configuration pass-through for engine,
  topology, model preset, cluster addresses, and Tailscale gateway binding.
- Target-user/path-aware `leharness.service` and `leharness-dashboard.service`
  templates ordered after Docker and Tailscale.

### Fixed
- LeHarness now tracks its real `master` default branch instead of nonexistent
  `main`.
- Provisioning rechecks and fails on unresolved LeHarness prerequisites rather
  than silently reporting success.
- Playbook layout tests now include the wired `leharness` role.
