# security-advisories

Security advisories by **Bruno Marlan (RedTrust)** — independent research on third-party (vendor) products.

- Contact: contato@redtrustcorp.com
- CVE requests submitted to MITRE (CNA-LR) on 2026-10-29 — ticket MCID16024782
- Each advisory covers a vendor product only. No customer or integrator data is included.

## Advisories

| # | Vendor | Product | Version | Title |
|---|---|---|---|---|
| 1 | iProov | Web SDK (liveness) | 5.4.3 | [Unauthenticated ECDH session + forgeable liveness verdict (MITM)](advisories/iproov-web-sdk-5.4.3.md) |
| 2 | Topaz | Heartbeat SDK (MidCrypt) | latest observed 2026-08 | [Derivable sync_id + hardcoded AES-128 key (forged anti-fraud payloads)](advisories/topaz-heartbeat-midcrypt.md) |
| 3 | Baikal | OIDC authorization server | latest observed 2026-09 | [PKCE optional + legacy grants advertised](advisories/baikal-oidc-pkce.md) |
| 4 | brscan | selfie/liveness capture API | latest observed 2026-10 (client SDK 4.0.21) | [Unauthenticated capture-link retrieval + sequential link ids](advisories/brscan-capture-api.md) |
| 5 | Unico | UnicoCheck Web SDK | latest observed 2026-09 | [Client-derivable protocol key + MFA-less process flow](advisories/unico-unicoCheck.md) |
| 6 | FaceTec | SDK (liveness) | v9 | [Client-side trust of liveness result (CWE-602)](advisories/facetec-sdk-v9-cwe602.md) |

## Disclosure status

- CVE IDs requested from MITRE CNA-LR (legacy cveform) — ticket MCID16024782, under review.
- Coordinated disclosure: vendor notifications are being sent in parallel with this publication.
- No customer names, integration identifiers or customer data appear in this repository.
