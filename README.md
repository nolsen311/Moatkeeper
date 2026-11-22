# Moatkeeper

A mobile app for managing pfSense® firewalls through Netgate's API.

> **Status:** pre-development. The design is in
> [docs/design-decisions.md](docs/design-decisions.md); the build order is in
> [docs/roadmap.md](docs/roadmap.md). There is no app to install yet.

## What it will do

- **See your firewall at a glance:** status, gateways, CPU and temperature,
  firmware and package updates.
- **Flip rules from your phone:** every rule per interface, with on/off
  toggles and how often each one is hit.
- **Find and cut off a device:** DHCP leases and ARP, and kill a device's
  connections.
- **Diagnose:** ping and port tests from the firewall.
- **Deal with certificates for you:** self-signed certificates, private CAs,
  broken chains and client certificates (mutual TLS), with expiry warnings.
- **Run with least privilege:** the app sets up its own limited account on the
  firewall instead of keeping your admin login.

Moatkeeper talks only to your firewalls, over your LAN or VPN. There is no
Moatkeeper server, and no analytics or tracking.

## Layout

| Path | Contents |
| --- | --- |
| `core/` (planned) | Rust: the Netgate API model, mapping, sessions and error handling, shared by both apps. |
| `android/` (planned, first) | Kotlin and Jetpack Compose: UI, OS integration and the network transport. |
| `ios/` (planned, second) | Swift and SwiftUI. |
| [`docs/`](docs/) | [Design decisions](docs/design-decisions.md) (the source of truth), the [roadmap](docs/roadmap.md), [follow-ups](docs/follow-ups.md) and [research](docs/research/netgate-api.md). |

## Requirements

A firewall running pfSense with Netgate's API: pfSense Plus 26.07 or later,
and pfSense CE from 26.10.

## Contributing and security

See [CONTRIBUTING.md](CONTRIBUTING.md) (commits need a DCO sign-off) and
[SECURITY.md](SECURITY.md). Please report vulnerabilities privately.

## License

GPL-3.0-or-later, with additional permissions allowing app store
distribution; see [LICENSING.md](LICENSING.md). The Moatkeeper name and logo
are covered by [TRADEMARKS.md](TRADEMARKS.md).

pfSense® is a registered trademark of Rubicon Communications, LLC (Netgate).
Moatkeeper is not affiliated with or endorsed by Netgate.
