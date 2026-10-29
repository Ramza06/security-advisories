# UnicoCheck Web SDK — Client-derivable protocol key + MFA-less process flow

| | |
|---|---|
| **Vendor** | Unico |
| **Product** | UnicoCheck Web SDK (biometric capture/liveness) |
| **Version** | latest observed (2026-09); UMD bundle `unico_UnicoCheck.umd.js` |
| **CWE** | [CWE-321](https://cwe.mitre.org/data/definitions/321.html) — Use of hard-coded/derivable key material |
| **CVSS 3.1 (preliminary)** | 7.7 High — `AV:L/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N` (to be confirmed with the vendor) |
| **Attack type** | Local |
| **Status** | CVE requested (MITRE CNA-LR, 2026-10-29). Coordinated disclosure in progress. |

## Summary

The SDK encrypts its protocol `dataValues` with key material derivable **entirely client-side**,
exposing the biometric motor configuration in cleartext. The biometric process can be started via
API **without MFA** and its consumption state polled.

## Technical detail

- `dataValues` key material is derivable client-side (raw PBKDF2 key observable in the WebView
  runtime), exposing `motorName`, `clientUnicoId`, `sdkKey`, `maxAttempts` in cleartext.
- The biometric process can be started via API without MFA (`generate-process` accepted with an
  empty payload).
- Consumption state is pollable: **200 alive / 404 consumed** — an oracle for flow state.

## Impact

- Forgery and enumeration of biometric sessions.
- Disclosure of motor configuration and SDK key.
- The capture result can be consumed by relying parties without independent validation.

## Reproduction

Production UMD bundle reversed; `dataValues` decrypted with runtime-captured key material; the
process API accepted `generate-process` without MFA and returned 200/404 on polling.

## Recommendations

- Move key derivation out of the client-observable context (server-provided per-session keys,
  end-to-end from the server).
- Require MFA-bound sessions to generate and consume biometric processes.
- Require independent server-side validation of capture results by relying parties.

## Timeline

- 2026-09-11 — UMD bundle captured
- 2026-09-17 — key derivation and process flow reversed
- 2026-10-29 — CVE requested from MITRE (CNA-LR); vendor notification in progress

## Evidence (deterministic)

- Production UMD bundle `unico_UnicoCheck.umd.js` reversed.
- `dataValues` decrypted with a runtime-captured PBKDF2 key (`crypto.subtle` hook), exposing
  `motorName`, `clientUnicoId`, `sdkKey`, `maxAttempts` in cleartext.
- `generate-process` accepted with empty payload (no MFA); `next-step` poll returns 200 alive /
  404 consumed.

## Credits

Bruno Marlan — RedTrust (contato@redtrustcorp.com)
