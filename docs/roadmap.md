# Moatkeeper Roadmap

Working plan for building Moatkeeper. `docs/design-decisions.md` is the
source of truth for *what* to build; this file is the order to build it in.
Claude Code: start at the first unchecked item, work on a branch, and open a
pull request per item or small group of items. Keep this file updated as
items land.

## Maintainer tasks (outside the code)

- [ ] Squash-merge PR #1 with a `Signed-off-by:` line in the merge message.
- [ ] Settings → Code security: enable Dependabot alerts and Dependabot
      security updates.
- [x] Private vulnerability reporting enabled (2026-11-21).
- [ ] Decide where repository settings live: a private `Moatkeeper-infra`
      (Terraform, like Terpoir-infra) or a module in Terpoir-infra. Then add
      a `protect-main` ruleset there: squash-only, linear history, no
      deletion or force-push, `dco` as a required check (more checks as CI
      lands).
- [ ] Name clearance: USPTO and EUIPO search for "Moatkeeper", Play Store and
      App Store search. Read Netgate's trademark policy for "for pfSense®"
      wording.
- [ ] Domain for the project site and the Android application ID.
- [ ] Decide paid app vs. free with a Pro unlock, and which features are Pro.
- [ ] Remove the internal hostname and LAN address from the public
      Pfsense-REST repo (`CLAUDE.md`, `tests/test_netgate_writes.py`).
- [ ] A test firewall or VM for live checks, separate from the home network.

## Phase 0: Repository and CI foundations

- [x] License, licensing permissions, trademark, contributing and security
      policies; design decisions; this roadmap.
- [x] DCO enforcement workflow (`.github/workflows/dco.yml`).
- [x] Dependabot version updates for GitHub Actions.
- [ ] CodeQL code scanning (Rust, Kotlin, Actions), actions pinned by SHA.
- [ ] OpenSSF Scorecard workflow.
- [ ] Rust CI: `cargo fmt --check`, `cargo clippy -D warnings` (deny
      `dbg_macro`, `print_stdout`, `print_stderr`, `todo`, `unwrap_used` in
      release code), `cargo test`.
- [ ] `cargo-deny` (GPL-compatible licenses, advisories, sources) and
      `cargo-vet`.
- [ ] Add `cargo` and `gradle` to Dependabot once `core/` and `android/`
      exist.

## Phase 1: Rust core (`core/`)

- [ ] Workspace and crates: `moatkeeper-api` (models, mapping, error typing),
      `moatkeeper-session` (login, 300 s token, refresh, re-login, lockout
      latch), `moatkeeper-ffi` (UniFFI).
- [ ] Transport interface the platforms implement (request in, response out),
      with a test double.
- [ ] Models and mapping for the first feature set: system status,
      monitoring, gateways, firmware and packages, rules (with `state`
      counters), leases and ARP, diagnostics, certificates (public fields
      only).
- [ ] Scrubbed fixtures from recorded responses; mapping tests against them.
- [ ] Change descriptions for writes (rule toggle, state kill), used by the
      confirmation screens.
- [ ] Certificate pin records and expiry calculations.

## Phase 2: Android app (`android/`)

- [ ] Project, `foss` and `play` flavours, Kotlin bindings smoke test in CI.
- [ ] OkHttp transport: pinning, private CA, client certificates (PKCS#12
      import and KeyChain picker), refresh-cookie handling.
- [ ] Connect flow: certificate review and pin, diagnosis of broken chains.
- [ ] Credential storage in Keystore, biometric unlock, backup exclusion.
- [ ] Dashboard; Rules with toggles and hit counts; Devices with state kill;
      Diagnostics; Certificates with expiry warnings.

## Phase 3: Simplified administration

- [ ] Least-privilege onboarding: create the app's own user and group, then
      discard the admin credentials (needs the LIVE-CHECK on user and group
      endpoints and privilege names).
- [ ] Certificate fixes where the API allows (LIVE-CHECK on renewal).
- [ ] Expiry notifications.

## Phase 4: Releases

- [ ] Signing key, reproducible `foss` builds, signed `SHA256SUMS`, GitHub
      Releases (following Terpoir's release process).
- [ ] Google Play listing and the paid or Pro model.

## Phase 5: iOS

- [ ] Swift package from the core, URLSession transport, feature parity with
      Android.

## Later

- Live state table, firewall log tail, traffic graphs, Wake-on-LAN.
- Multiple firewalls (Pro candidate).
