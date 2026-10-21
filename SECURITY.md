# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability, please report it responsibly:

- **Do NOT open a public GitHub issue.**
- Email: security@meetmind.example (or DM a maintainer).
- Include a clear description, reproduction steps, and potential impact.

We will acknowledge your report within 48 hours and aim to resolve critical
issues within 7 days.

## Security Measures

MeeyMind implements defense-in-depth across every layer:

| Layer          | Controls                                                       |
|---------------|---------------------------------------------------------------|
| **Network**    | CORS allow-lists, HTTPS everywhere, CSP headers                |
| **API**        | Auth required on all routes, per-request RBAC checks           |
| **Input**      | Zod schema validation at every boundary                        |
| **Rate limit** | Auth endpoints + AI-heavy routes rate-limited                  |
| **Secrets**    | Environment variables only — never committed to source control   |
| **Media**      | WebRTC traffic encrypted (DTLS-SRTP)                            |
| **Storage**    | Recordings & transcripts encrypted at rest (AES-256)            |
| **Sessions**   | Secure, httpOnly, SameSite cookies; short-lived + refresh tokens |

## What We Ask

- Keep secret keys secret — never push `.env.local` or share credentials.
- Rotate keys periodically and immediately after any suspected exposure.
- Report suspicious activity or potential bypasses promptly.
