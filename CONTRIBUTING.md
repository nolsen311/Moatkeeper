# Contributing to Moatkeeper

Thanks for your interest. Moatkeeper manages people's firewalls, and
contributions are held to that bar.

## Before you start

Read [docs/design-decisions.md](docs/design-decisions.md). It records the
architecture, the security model, and the decisions already made. Proposals
that change a recorded decision should open an issue first.

## Ground rules

- **No third-party SDKs** for analytics, advertising, crash reporting, or
  attribution. New dependencies need a stated reason and are reviewed for
  network access, telemetry, and license compatibility.
- **No credentials, tokens, firewall addresses or configuration in logs**,
  ever, in any build type.
- **API logic lives in the Rust core**: the data model, session handling,
  error typing and change descriptions. Platform code does UI, OS
  integration and the network transport (TLS, pinning, client certificates).
- **Writes are explicit.** Nothing changes a firewall without a user action
  and a confirmation that names the change.
- **Test data is scrubbed.** Responses recorded from a real firewall must have
  hostnames, IP and MAC addresses, serials, `netgate_id`, user names and rule
  descriptions replaced before they are committed. Raw captures go in the
  git-ignored `captures-raw/`.

## Developer Certificate of Origin

All commits must be signed off, certifying the
[Developer Certificate of Origin 1.1](https://developercertificate.org/):
that you wrote the change or otherwise have the right to submit it under the
project's license, including the additional permissions in
[LICENSING.md](LICENSING.md).

Sign off by committing with `-s`:

```
git commit -s -m "Describe the change"
```

This adds a line such as:

```
Signed-off-by: Your Name <you@example.com>
```

You may use a pseudonym and a no-reply address. A GitHub
`users.noreply.github.com` address works. The email in your sign-off must
match the commit's author email.

The `DCO` check enforces this on every pull request. GitHub bots such as
Dependabot are exempt. If the check fails, add sign-offs with
`git rebase --signoff <base>` and force-push your branch.

## Reporting bugs

Bugs are tracked in [GitHub Issues](https://github.com/nolsen311/Moatkeeper/issues)
with the `bug` label; the roadmap is for planned work only. Issues are
public, so leave out your firewall's addresses, hostnames, rule names and
anything else about your network.

## Security issues

Do not open public issues for vulnerabilities. See [SECURITY.md](SECURITY.md).
