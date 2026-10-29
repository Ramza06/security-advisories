# FaceTec SDK v9 — Client-side trust of liveness result (CWE-602)

| | |
|---|---|
| **Vendor** | FaceTec |
| **Product** | FaceTec SDK (liveness) |
| **Version** | v9 |
| **CWE** | [CWE-602](https://cwe.mitre.org/data/definitions/602.html) — Client-Side Enforcement of Server-Side Security |
| **CVSS 3.1 (preliminary)** | 6.1 Medium — `AV:L/AC:L/PR:N/UI:N/S:U/C:N/I:H/A:N` (to be confirmed with the vendor) |
| **Attack type** | Local |
| **Status** | CVE requested (MITRE CNA-LR, 2026-10-29). Coordinated disclosure in progress. |

## Summary

The SDK delivers the liveness session result through **client-side callbacks** that integrators
may trust without server-side faceScan validation; the result code and challenge status are
enforceable client-side and forgeable in the app context.

## Technical detail

- The session result (`resultCode`) and the challenge status are delivered via client-side
  callbacks consumed by the integrator's UI.
- Both are forgeable in the app context: the challenge status was rewritten from `no_attempt` to
  `SUCCEEDED` and the integrator's UI accepted the forged success.
- The SDK does **not** enforce or verify the server-side proof: whether the server-side faceScan
  validation exists is left entirely to each integration.

## Impact

Integrations that rely solely on the SDK callback accept a **forged liveness result** — a local
attacker (instrumented app, hook) can have a failed or absent liveness session reported as
successful.

## Reproduction

Session result and challenge status rewritten in the app context (no_attempt → SUCCEEDED); the
integration UI accepted the session. Server-side faceScan validation is implemented by some
integrators but is not enforced by the SDK.

## Recommendations

- The SDK should require and verify the server-side faceScan proof before reporting success, or
  unambiguously enforce integrator-side server validation (default-fail, documented and enforced
  by the SDK API).

## Timeline

- 2026-09 — forge demonstrated in integration contexts
- 2026-10-29 — CVE requested from MITRE (CNA-LR); vendor notification in progress

## Evidence (deterministic)

- In the app context, the session result callback and the challenge status were rewritten
  (`no_attempt` → `SUCCEEDED`) and the integration UI accepted the session.
- The SDK (v9) does not enforce server-side faceScan validation (CWE-602).

## Credits

Bruno Marlan — RedTrust (contato@redtrustcorp.com)
