# Baikal OIDC authorization server — PKCE optional + legacy grants advertised

| | |
|---|---|
| **Vendor** | Baikal |
| **Product** | Baikal OIDC authorization server (multi-tenant) |
| **Version** | latest observed (2026-09); server-side version not exposed |
| **CWE** | [CWE-287](https://cwe.mitre.org/data/definitions/287.html) — Improper Authentication · [CWE-306](https://cwe.mitre.org/data/definitions/306.html) — Missing Authentication for Critical Function |
| **CVSS 3.1 (preliminary)** | 7.4 High — `AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:N` (to be confirmed with the vendor) |
| **Attack type** | Remote |
| **Status** | CVE requested (MITRE CNA-LR, 2026-10-29). Coordinated disclosure in progress. |

## Summary

The authorization server treats **PKCE as optional**: the S256 `code_challenge` is enforced only
by the client application, and direct authorization requests without a `code_challenge` are
accepted by the server. The discovery document also advertises legacy grant types.

## Technical detail

- Authorization requests without `code_challenge` are accepted by the server (verified live on
  the production tenant).
- PKCE S256 is enforced client-side only — the server does not require it.
- The discovery metadata advertises legacy grants: `password`, `jwt-bearer`, CIBA/device
  (200+ scopes; public JWKS/discovery).
- Consequences: with stolen credentials, tokens can be minted without the PKCE binding; an
  intercepted authorization code can be redeemed without the `code_verifier`.

## Impact

Weakened protection against authorization-code interception and credential theft in multi-tenant
deployments; token acquisition outside the PKCE binding (account takeover surface).

## Reproduction

Tested against the production tenant: authorization requests without `code_challenge` accepted;
legacy grants confirmed in the discovery metadata.

## Recommendations

- Enforce PKCE S256 server-side for public clients; reject authorization requests without
  `code_challenge`.
- Disable legacy grant types (or restrict them to explicitly authorized confidential clients).
- Apply per-tenant client policy (allowed grants, scopes, redirects).

## Timeline

- 2026-08-11 — platform pivot / discovery captured
- 2026-09-08 — grant matrix completed
- 2026-10-29 — CVE requested from MITRE (CNA-LR); vendor notification in progress

## Evidence (deterministic)

- `/.well-known/openid-configuration` returns 200 (200+ scopes; legacy grants advertised:
  `password`, `jwt-bearer`, CIBA/device).
- Authorization requests without `code_challenge` are accepted server-side (verified live on the
  production tenant, 2026-09).

## Credits

Bruno Marlan — RedTrust (contato@redtrustcorp.com)
