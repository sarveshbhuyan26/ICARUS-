# Security and privacy status

PrepPilot is a local hackathon prototype. Accounts and guest identities now protect
learner sessions, preferences and owned papers. They do not make it production-ready.

## Implemented

- Salted PBKDF2-SHA256 password hashes; SQLite account uniqueness and transactions.
- Random 30-day authentication tokens stored only as SHA-256 digests in SQLite.
- HttpOnly, SameSite=Lax authentication cookies; logout revokes the current token.
- Ownership checks for learner data and papers, including resumed session IDs.
- Cross-origin mutation requests with a mismatched Origin header are rejected.
- Frontend display caches do not establish identity. API keys stay on the server.

`PREPPILOT_SECURE_COOKIES=1` enables Secure cookies for HTTPS deployments. Local HTTP
uses non-Secure cookies for development. One worker remains required because learner
objects and mutation locks are cached in-process. No new public deployment is implied.

## Private data

`preppilot.sqlite3` and its WAL/SHM files contain account and learning data. They are
Git-ignored, as are old account/paper files and `.env.local`. Back up the database using
the migration guide. Old unowned papers are not silently assigned or served to accounts.
Guest progress is accessible through the guest cookie and cannot be recovered through
an account if that cookie is lost or expires.

Selected Gemini/Groq requests can include question context, chosen answers and doubt
text. Avoid real student personal information in evaluation. Scripts and fonts may
load from CDNs. The provider selector never exposes API keys.

## Remaining work

Rate limiting, account recovery, a retention/deletion policy, administrator controls,
comprehensive security auditing and public-service privacy documentation remain
unfinished. Public catalog/simulation routes are not a production abuse boundary.

## Reporting

Do not post keys, account databases or exploit details in public issues. Use private
vulnerability reporting if enabled, or arrange a private channel with the repository
owner. No response SLA or monitored security address is claimed.
