# iProov Web SDK 5.4.3 — Unauthenticated ECDH session + forgeable liveness verdict (active MITM)

| | |
|---|---|
| **Vendor** | iProov Ltd |
| **Product** | iProov Web SDK (remote biometric liveness / identity assurance) |
| **Version** | 5.4.3 (assets served from `cdn.iproov.app/iproov-assets/5.4.3/`); earlier/later versions with the same WebSocket design are likely affected |
| **CWE** | [CWE-322](https://cwe.mitre.org/data/definitions/322.html) — Key Exchange without Entity Authentication |
| **CVSS 3.1 (preliminary)** | 7.4 High — `AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:N` (to be confirmed with the vendor) |
| **Attack type** | Remote — active network MITM |
| **Status** | CVE requested (MITRE CNA-LR, 2026-10-29). Coordinated disclosure in progress. |

## Summary

The SDK establishes its biometric session channel **before** the liveness gate and performs an
**unauthenticated ECDH P-256 key exchange**. An active network attacker can terminate the
transport, perform the key exchange with both the client and the genuine server, and decrypt,
alter and re-encrypt frames in transit — including the liveness verdict (`EdgeResult`).

## Technical detail

- The WebSocket session opens before the face/liveness gate; the SDK accepts any endpoint (no
  server authentication / pinning on the WS channel).
- ECDH P-256 is performed without authenticating the server; per-session material (one-time
  token, ephemeral ECDH keys) is readable from the client runtime.
- Frame envelope: `c2 7e <tipo>`; data frames carry an AES-GCM-encrypted blob with a protobuf
  trail (`campo5=1`, `campo6=TAG 16B`).
- The verdict frame (`EdgeResult.passed`) can be rewritten from `rejected` to `success`.
- Assurance is negotiable client-side: the integration requested `["genuine_presence","liveness"]`
  and was granted `liveness` with `flags.emc=false` and `fs_params {efs:false,fpc:0,noc:0}` —
  all additional checks disabled.
- The anti-bot PoW (`utils_wasm`, SHA3/Keccak solver) is dormant in this flow (`emc=false`) and
  is not a barrier to the MITM.

## Impact

Bypass of biometric liveness/identity assurance:
- **Active interception** — the biometric stream is exposed to the attacker in transit.
- **Verdict spoofing** — a failed or absent liveness check is reported as passed.
- **Silent assurance downgrade** — the granted assurance differs from the requested one without
  detection by the relying party.
- **Replay** — genuine sessions can be reused.

## Reproduction

Six attack paths were implemented and validated against 5.4.3 in a controlled test environment:

1. socat redirect of the SDK WebSocket,
2. BoringSSL certificate-acceptance patch (`libwebviewchromium`),
3. CDP `Network.webSocketFrame*` capture (12 frames validated),
4. mitmproxy addon rewriting the verdict frame (`rejected → success`),
5. `WebSocket.prototype.send` hook (full capture),
6. full double-ECDH MITM proxy (two keys: reads/modifies/rejects any frame).

The verdict frame was modified from rejected to success and the relying application accepted the
session. Evidence (stream captures, transcripts, keys) available to the assigner or vendor on
request.

## Recommendations

- Authenticate the WebSocket endpoint (server authentication / certificate pinning on the WS
  channel) before exchanging keys.
- Bind the liveness verdict to a server-authenticated channel; do not allow the client to
  negotiate the assurance level — enforce the requested assurance server-side (enable EMC).
- Treat the biometric stream as sensitive data at the transport layer (mutually authenticated,
  forward-secret channel).

## Timeline

- 2026-09-09/11 — discovery, reverse engineering and POC
- 2026-10-29 — CVE requested from MITRE (CNA-LR); vendor notification in progress

## Evidence (deterministic)

- Assets captured from `cdn.iproov.app/iproov-assets/5.4.3/` (17 artifacts; main bundle 893.js).
- WS envelope `c2 7e <tipo>`; data head `a9 00 a3 0b 5c f4 08 01 10 02 22`; AES-GCM per frame with protobuf trail.
- Verdict frame `EdgeResult.passed` rewritten from `rejected` to `success`.
- Assurance requested `["genuine_presence","liveness"]`, granted `liveness` with `flags.emc=false` and
  `fs_params {efs:false,fpc:0,noc:0}`.
- Per-session ECDH shared secret and AES-GCM session key captured during the MITM.

## Credits

Bruno Marlan — RedTrust (contato@redtrustcorp.com)
