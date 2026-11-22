# CLAUDE.md

Standing instructions for Claude Code working in this repository.

## Read first

`docs/design-decisions.md` is the source of truth for architecture, the
security model, and decisions already made. Do not contradict a recorded
decision without raising it with the maintainer first. When a decision is
made in a session, update that doc in the same change.

`docs/roadmap.md` is the build order. Start at the first unchecked item in
the earliest phase, and tick items off in the same pull request that
completes them.

`docs/research/netgate-api.md` is what is known about Netgate's API. Items
marked **LIVE-CHECK** are unverified: don't build on them as facts.

## Workflow

- `main` is protected: never push to it. Work on a branch and open a pull
  request; the maintainer reviews, signs off, and squash-merges.
- Check a pull request is still open before pushing to its branch or
  editing it: `gh pr view <n> --json state -q .state | grep -qx OPEN`.
  The maintainer merges as work goes along.
- Pin every GitHub Action by full commit SHA with a version comment.
- Stacked pull requests: after the lower PR is squash-merged, rebase the next
  branch onto `main` (`git rebase --onto main <old lower tip> <branch>`) and
  force-push it with `--force-with-lease`. Only on branches Claude created.
- When the network blocks a source, tell the maintainer at once, naming the
  host. Never fill the gap silently: any placeholder, workaround or
  substitute gets an entry in `docs/follow-ups.md`.

## Architecture

- `core/` (planned): Rust library owning the Netgate API data model,
  response mapping, session handling, error typing, change descriptions,
  certificate pin records and expiry logic. Exposed via UniFFI.
- `android/` (planned, first): Kotlin + Jetpack Compose. UI, OS integration
  and the network transport (OkHttp: TLS, pinning, client certificates).
- `ios/` (planned, second): Swift + SwiftUI, same split (URLSession).
- No server component. Never add one.

## Non-negotiable rules

- **API logic belongs in the Rust core.** Never reimplement mapping, session
  handling or error typing in Kotlin or Swift.
- **TLS belongs in the platform layer**, because OS-installed client
  certificates have non-exportable keys. The core decides policy (which
  fingerprint is pinned); the platform enforces it on every connection.
- **Never accept a certificate the user hasn't confirmed.** No "trust all"
  code paths, not even in debug builds.
- **Never retry a rejected login automatically** (account lockout).
- **No writes without an explicit, confirmed user action.**
- **No credentials, tokens, firewall addresses, MACs, hostnames or
  configuration in logs**, in any build type. Strip logging from release
  builds.
- **Never copy a certificate's private key** out of an API response
  (`/system/certificates` returns `prv`).
- **No third-party SDKs** for analytics, ads, crash reporting, or
  attribution. Justify every new dependency.
- **Credentials live in Android Keystore / the iOS Keychain**
  (this-device-only); app data is excluded from OS backups.
- **Test fixtures are scrubbed** before commit (see CONTRIBUTING.md).
- **This repo is public.** Nothing about the maintainer's own network
  (hostnames, addresses, rule names) goes in code, docs, fixtures or commit
  messages.

## Conventions

- New source files carry the SPDX header from `LICENSING.md`.
- Commits by Claude are authored as `Claude <noreply@anthropic.com>`. Human
  contributors sign off commits (`git commit -s`), per `CONTRIBUTING.md`.
  The sign-off is a personal legal certification: never add a
  `Signed-off-by` line on anyone's behalf. The maintainer signs off on
  Claude-authored changes when merging.
- Never commit signing keys, keystores, provisioning profiles, or secrets.
