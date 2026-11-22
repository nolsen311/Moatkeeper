# Moatkeeper — Design Decisions

Working name: **Moatkeeper**. Domain: none yet (see `docs/follow-ups.md`).
Repository: https://github.com/nolsen311/Moatkeeper
Status: pre-development, high-level design. Last updated 2026-11-21.

## Purpose

A mobile app for people who run pfSense firewalls, built on Netgate's own API.
It makes everyday administration quick from a phone (checking health, flipping
rules, finding and cutting off a device) and takes the pain out of the parts
that usually stop people: certificates, credentials and safe access.

## Guiding principles

- **Simplify administration.** Every screen answers "what do I need to do?"
  before "what can I configure?". Common jobs take one or two taps; anything
  that changes the firewall says exactly what will change before it does.
- **Least privilege by default.** The app runs with its own limited account,
  never the user's admin login (see Onboarding).
- **Certificates are part of the product, not a prerequisite the user must
  sort out first.** Self-signed certificates, private CAs, broken chains and
  client certificates are handled in the app, with plain explanations (see
  Certificates and TLS).
- **No operator-run backend.** The app talks only to the user's firewalls.
  The project never sees credentials, configuration or traffic, and is never
  a subpoena or compromise target.
- **Honest disclosure.** When an action has risk (disabling a rule, killing
  states, rebooting), say what will happen and how to undo it.

## Licensing and business model (maintainer, 2026-11-21)

Modelled on Terpoir:

- **GPL-3.0-or-later** with the additional permissions in `LICENSING.md`
  (app stores; Google Play libraries). The source is public from day one.
  Rationale: users hand this app admin access to their firewall; being able to
  audit it is a selling point for exactly this audience.
- **Trademarks are kept.** The GPL covers the code, not the Moatkeeper name or
  logo (`TRADEMARKS.md`), so a modified build can't pose as the official one.
- **Two build flavours:** `foss` (GitHub Releases, later F-Droid; free) and
  `play` (Google Play). Revenue comes from the store build. **Open:** a paid
  app vs. a free app with a Pro unlock, and which features are Pro (multi-
  firewall management is the leading candidate). See `docs/follow-ups.md`.
- **Not affiliated with Netgate.** "pfSense" is Netgate's trademark. It is
  used only to describe compatibility ("for pfSense®"), never as part of the
  app's name or icon. Netgate's trademark policy is to be reviewed before any
  store listing (follow-up).

## Platforms and architecture

- Native apps on Android and iOS. **Android first, iOS second.** Feature
  parity, as in Terpoir: a feature added on one platform is added on the other.
- **Shared core in Rust**, exposed via UniFFI (Kotlin and Swift bindings). The
  core owns:
  - the Netgate API data model and the mapping from raw responses;
  - session handling (login, the 300-second token, refresh, re-login);
  - error typing (the API reports most failures as HTTP 400 with a message,
    so errors are classified from `errmsg`, not the status code);
  - validation and the "what will this change do" descriptions for writes;
  - certificate pin records and expiry logic.
- **The network transport lives in the platform layer (decided 2026-11-21).**
  The core defines a small transport interface (send a request, get a
  response); Android implements it with OkHttp, iOS with URLSession.
  Rationale: client certificates installed in the OS (Android KeyChain, the
  iOS keychain) have **non-exportable private keys**. Only the platform TLS
  stack can use them, so TLS, pinning enforcement and client-certificate
  presentation must happen there. This is the main departure from Terpoir,
  where the core does everything security-related.
- UI: Kotlin + Jetpack Compose on Android; Swift + SwiftUI on iOS. Platform
  code handles UI and OS integration only.

## Which API

- **Netgate's API only** (the Go-based controller on the firewall, as shipped
  with pfSense Plus and, from 26.10, every pfSense install). Paths are
  unversioned under `/api`; the spec is https://github.com/Netgate/pfsense-api.
- **Not the community REST API package (pfrest).** It's PHP-based and in
  maintenance mode; it stops working when Netgate removes PHP. It also
  misreports rule state (rules disabled in the web UI show as enabled).
- Research already done against a live 26.07 box is summarised in
  `docs/research/netgate-api.md`.

## Connectivity

- The app connects to firewalls on the LAN or over the user's VPN. It never
  encourages exposing the API to the internet, and onboarding says so.
- Users who do put it behind a reverse proxy with mutual TLS are supported
  through client certificates (below).

## Certificates and TLS (decided 2026-11-21)

- **Trust on first use with an explicit pin.** On first connect the app shows
  the firewall's certificate (subject, issuer, expiry, SHA-256 fingerprint)
  and pins it after the user confirms. A changed certificate later is never
  accepted silently: the user sees old and new side by side and re-confirms.
- **Broken chains are diagnosed, not just rejected.** For example, a firewall
  that serves an expired intermediate CA with a valid leaf: say what is wrong
  and how to fix it, and offer the fix where the API allows it (follow-up:
  whether the API can renew certificates is untested).
- **Private CAs** can be imported and trusted per firewall.
- **Client certificates (mutual TLS):** import a PKCS#12 (.p12/.pfx) file with
  its password, or on Android pick a certificate already installed in the
  system (KeyChain), so it can be managed by MDM.
- **Certificate monitoring:** the app lists the firewall's certificates and
  warns before any in-use certificate expires.

## Onboarding and credentials (decided 2026-11-21)

- **Least-privilege setup wizard.** The user signs in with an admin account
  once. The app creates its own user and group on the firewall with only the
  privileges its features need, then discards the admin credentials.
- Stored credentials are encrypted with Android Keystore (StrongBox where
  available) / the iOS Keychain (this-device-only), and unlocked with the
  device's biometrics or credential.
- App data is excluded from OS backups.

## Writes and safety

- Every write is an explicit user action with a confirmation that names the
  change. Nothing is changed in the background.
- After a change the app applies it the way the web UI does (pending changes
  are applied) and reports the result.
- Destructive actions (reboot, killing all states) need a second
  confirmation.

## First feature set

1. Dashboard: system status, gateways, CPU, temperature, firmware and package
   updates.
2. Rules: per interface, with on/off toggles and hit counters.
3. Devices: DHCP leases and ARP; per device, kill its connections.
4. Diagnostics: ping and port test.
5. Certificates: list, expiry warnings.

Later: live state table, firewall log tail, traffic graphs, Wake-on-LAN,
multiple firewalls.

## Logs and privacy

- No credentials, tokens, firewall addresses, hostnames, MACs or rule
  contents in logs, in any build type. Release builds strip logging.
- No third-party SDKs for analytics, ads, crash reporting or attribution.
- The app never contacts a project-run endpoint.
