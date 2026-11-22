# Netgate API: what we know

Findings from running a Home Assistant integration
([nolsen311/Pfsense-REST](https://github.com/nolsen311/Pfsense-REST), MIT,
at `cce2e81`) against a live Netgate 6100 on pfSense Plus 26.07, plus
read-only checks made on 2026-11-17. Its `CLAUDE.md` ("Netgate API backend")
and `pypfsense/netgate_map.py` hold the details and the response mapping this
summary draws on. Anything not verified live is marked **LIVE-CHECK**.

Specification: https://github.com/Netgate/pfsense-api (`pfapi_openapi.yml`,
about 486 paths). It has no `servers` or `securitySchemes`, and some field
shapes differ from real responses, so hand-written models checked against
recorded responses are safer than a generated client.

## Where it runs

- The controller is Netgate's Go-based "Nexus" (`pfnet-controller`), serving
  the new web UI and the API on its own HTTPS port (8443 on the test box).
- Base path: `/api`. Paths are unversioned and flat, for example
  `/system/status`, `/firewall/rules/interface/{if}`, `/diag/states`.
- From 26.10, pfSense CE and Plus reconverge and every install gets this API.

## Authentication

- **Login:** `POST /api/login` with `{"username": ..., "password": ...}`,
  **both values base64-encoded**. Returns `{token, user, version, alerts}`
  and sets a `pfsense-refresh-token` cookie
  (`Secure; HttpOnly; Path=/api/login/refresh`, 24 hours).
- **Access token:** a JWT sent as `Authorization: Bearer`. `exp` is login
  time **+300 seconds**; there is no `iat`. Claims: `addr, auth, exp, iss,
  keyid, username`; `auth.roles` lists the account's roles.
- **Refresh:** `POST /api/login/refresh` with `{"username": ...}`, the Bearer
  token and the refresh cookie, returns a new 300-second token. Refresh about
  60 seconds before `exp`; if refresh fails, log in again.
- Cookie handling may need to be manual: some HTTP stacks drop cookies set by
  bare-IP hosts.

## Errors

- Shape: `{errcode, errlevel, errmsg}`.
- An invalid or expired session is **HTTP 400** `"session invalid,
  reauthenticate (...)"`, not 401. Classify errors from `errmsg` first.
- Bad credentials: 400 `"invalid credentials"`.
- A user without API access: `"unauthorized login"` (treat as a privilege
  problem, not bad credentials).
- An unknown top-level path: 400 `"invalid namespace"`.
- **Lockout safety:** never retry a rejected login automatically.

## Reads verified live

- **Rules:** `GET /firewall/rules/interface` lists interfaces and groups
  (`interface_mappings`); `GET /firewall/rules/interface/{if}` returns
  `entries[]`, each either `{rule: {...}}` or `{separator: ...}`. Fetch per
  interface identity from `/system/interfaces/status` (the listing omits
  disabled interfaces), plus groups and `floating`. Drop `readonly` rules and
  system trackers like `"10000-10999"`.
  - `disabled` and `log` are reported **correctly** (pfrest gets both wrong).
  - Each rule carries live pf counters in `rule.state`: `evaluations`,
    `packets`, `bytes`, `states`, `state_creations`. Hit counts need no shell.
  - `tracker` matches pf's `ridentifier`; one rule can expand to several pf
    lines.
  - `created`/`updated` carry `{time, username}`.
- **Status:** `/system/status` (identity `status.ndi`, platform, `osver`
  version string, memory). `/system/setup` (hostname, domain).
- **Monitoring:** `POST /system/monitoring {scope: system, dataset: ...}`
  returns 60-second averages (CPU from `processor`, temperatures from
  `sensors`, mbufs). The `memory` dataset returns zeros. Load averages come
  from `GET /controller/stats` (`cpu_load`, last 60 one-minute samples).
- **Gateways:** status `Online` etc.; delay and loss are strings with units.
- **DHCP leases:** `/services/dhcp/leases` (MACs upper-case). **ARP:**
  `/diag/arp` takes about 5.6 seconds (reverse DNS, no way to skip).
- **Services:** `/services/status`.
- **Firmware:** `GET /system/update` (`curr_base_system`,
  `latest_base_system`, `firmware_branch`, ...), about 1 second.
  `/system/firmware` is the board firmware (coreboot), not pfSense.
- **Packages:** `GET /system/package/installed`.
- **Certificates:** `GET /system/certificates` returns every certificate
  **including its private key** (`prv`). Only ever copy the public fields
  (refid, name, type, `in_use`, start, expiry, issuer, `can_renew`). Expiry
  looks like `"2027-01-18 09:17:33 +0000 UTC"`.
- **State table size:** `GET /diag/pfinfo` (fields base64-encoded text).
- **Latency:** rules take about 1.5 seconds per interface; a full refresh of
  everything is about 9 seconds. The box partly serializes requests.

## Writes verified live

- **Rule toggle:** `POST /firewall/rules/interface/{if}/toggle/{id}
  {"value": true}` disables, `false` enables. The firewall reloads at once.
  `POST /system/config/dirty {"apply": true}` afterwards is harmless.
- **Aliases:** `PUT /aliases` creates, `POST /aliases/{id}` updates
  (`id` = list index, as a string), `DELETE /aliases/{id}` deletes.
- **Package install:** `POST /system/package/install
  {"packages": [{"name": "pfSense-pkg-<x>"}]}`; returns in about 0.5 s.
- **Diagnostics** (change nothing): `POST /diag/ping`, `POST /diag/traceroute`
  (needs `proto`), `POST /diag/testport` (closed port = HTTP 400 after about
  10 s), `GET /diag/dns?host=`.
- **States:** `DELETE /diag/states` needs both `srcip` and `dstip`;
  `GET /diag/states?filter_str=` is a substring match. **LIVE-CHECK:** the
  pair DELETE against a real state.
- **Shell:** `POST /diag/cmd/shell {cmd: <base64>, timeout}`; the response is
  base64 too. Needs the shell privilege. Avoid in Moatkeeper where an API
  endpoint exists.

## Not yet verified

- **LIVE-CHECK:** whether certificates can be renewed through the API
  (`can_renew` is reported; no renew call has been tested).
- **LIVE-CHECK:** creating users and groups and assigning API privileges (the
  least-privilege onboarding depends on it), and the exact privilege names
  each feature needs.
- **LIVE-CHECK:** firmware upgrade (`POST /system/update`), reboot
  (`POST /diag/control`).
- **LIVE-CHECK:** behaviour on 26.10 CE once released.

## TLS observed in the field

- The controller's certificate is typically issued by a CA on the firewall
  itself, so it's untrusted by phones by default.
- One box served a valid leaf certificate with an **expired intermediate CA**
  in the chain, while another port on the same box served a renewed one.
  Standard validation rejects that; pinning the leaf works. This is why
  Moatkeeper pins and diagnoses rather than relying on CA validation.
