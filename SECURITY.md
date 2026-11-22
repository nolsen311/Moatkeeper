# Security Policy

## Reporting a vulnerability

Please report vulnerabilities privately through GitHub's
**Report a vulnerability** button on this repository's Security tab
(private vulnerability reporting). Do not open a public issue.

Include what you found, how to reproduce it, and the impact you expect. We
will acknowledge reports and keep you informed as we work on a fix.

## Scope

Of particular interest:

- Anything that exposes stored firewall credentials, session tokens or client
  certificate keys outside the platform keystore.
- Ways to make the app accept a certificate the user didn't confirm, or to
  connect without the pinned certificate.
- Ways to make the app change a firewall without an explicit, confirmed user
  action.
- Firewall data (addresses, configuration, certificates' private keys) in
  logs, backups or any channel the app doesn't disclose.
- Supply-chain or build-reproducibility problems.

## We will never ask for your firewall credentials

No one from the Moatkeeper project will ever ask for your firewall password,
API tokens or certificates, by any channel. Anyone who does is attempting to
take over your network.

## Official builds

Only builds signed with the project's published keys are official. See
[TRADEMARKS.md](TRADEMARKS.md).
