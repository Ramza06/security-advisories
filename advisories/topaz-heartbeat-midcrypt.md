# Topaz Heartbeat SDK (MidCrypt) — Derivable sync_id + hardcoded AES-128 key

| | |
|---|---|
| **Vendor** | Topaz (br.com.topaz) |
| **Product** | Topaz Heartbeat SDK (anti-fraud device fingerprint, MidCrypt) |
| **Version** | latest observed (2026-08); no version string exposed by the SDK |
| **CWE** | [CWE-321](https://cwe.mitre.org/data/definitions/321.html) — Hard-coded key material · [CWE-340](https://cwe.mitre.org/data/definitions/340.html) — Predictable identifiers |
| **CVSS 3.1 (preliminary)** | 7.7 High — `AV:L/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N` (to be confirmed with the vendor) |
| **Attack type** | Local |
| **Status** | CVE requested (MITRE CNA-LR, 2026-10-29). Coordinated disclosure in progress. |

## Summary

The SDK constructs its device-fingerprint identifier (`sync_id`) from a locally stored UUID plus a
timestamp, Base64url-encoded as `-1_|_-1_|_<UUID>|<timestamp>`, with a conditional override that
encrypts heartbeat data through the native library. The native library (`libmcrypt.so`) embeds a
**hardcoded AES-128 key** and exposes its encryption primitives via JNI without per-device keying.

## Technical detail

- `sync_id = Base64url("-1_|_-1_|_<UUID>|<token_time_diff+currentTimeMillis>")`, where the UUID
  is stored in the app's `SharedPreferences` (`ofd_hb_random_id`) and `token_time_diff` is a
  cached offset — both readable from the local app context.
- Conditional override (heartbeat active): `"1_|_" + i + "_|_" + MidCrypt(device_id+config,
  data, timestamp)`.
- `libmcrypt.so` exposes 17 JNI functions (`function14/18/24/42-56`); the crypto is injected
  pervasively (30+ call sites) and is not bound to per-device secrets.
- `FileDecryption` uses the hardcoded AES-128 key `7F4E2FD93482A3C12D47A3A10CA08888` with an IV
  from `libdigest.so`.

## Impact

A local attacker (rooted device, malware, instrumentation) can:
- read the stored UUID and reproduce the `sync_id`,
- invoke the exported JNI primitives to seal **forged heartbeat/anti-fraud payloads**,
- decrypt files protected by the hardcoded key.

This undermines the device-integrity signals the SDK is intended to provide to relying parties.

## Reproduction

Reverse engineered from the MidCrypt implementation bundled in a production Android application:
the `sync_id` construction path, the conditional heartbeat override and all 17 JNI functions were
mapped, and the AES-128 key was recovered from `FileDecryption`.

## Recommendations

- Derive per-device keys from hardware-backed storage (Android Keystore/StrongBox); remove the
  hardcoded key.
- Bind `sync_id`/heartbeat payloads to server-issued, high-entropy challenges.
- Validate heartbeat/anti-fraud telemetry server-side (freshness, expected device binding).

## Timeline

- 2026-08-19 — artifact captured
- 2026-09-22 — reverse engineering completed
- 2026-10-29 — CVE requested from MITRE (CNA-LR); vendor notification in progress

## Evidence (deterministic)

- `sync_id = Base64url("-1_|_-1_|_<UUID>|<token_time_diff+currentTimeMillis>")` built in
  `br.com.topaz.z0.b`, with conditional override `"1_|_" + i + "_|_" + MidCrypt(device_id+config, data, timestamp)`.
- `libmcrypt.so` exports 17 JNI functions (`function14/18/24/42-56`).
- `FileDecryption` hardcoded AES-128 key `7F4E2FD93482A3C12D47A3A10CA08888`; IV from `libdigest.so`.

## Credits

Bruno Marlan — RedTrust (contato@redtrustcorp.com)
