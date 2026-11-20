# Private chat media foundation

Private chat images are stored as normalized JPEG, PNG or WebP objects in a
private Cloudflare R2 bucket. The browser never receives an R2 URL, bucket name,
credential or object key. Upload and download both traverse authenticated Konti
backend routes.

Each environment has an isolated bucket and a bucket-scoped Object Read & Write
credential. Staging uses `kontener-app-staging-chat-media`; production must use
a separate `kontener-app-production-chat-media` bucket and credential when the
release candidate is frozen. Public Development URL, custom public domains and
CORS remain disabled.

Required server variables:

- `R2_ACCOUNT_ID`
- `R2_ACCESS_KEY_ID`
- `R2_SECRET_ACCESS_KEY`
- `R2_BUCKET_NAME`
- optional `R2_ENDPOINT` (defaults to the account S3 endpoint)

The backend validates and decodes the actual image, enforces byte, dimension and
pixel limits, applies orientation, and re-encodes without optional metadata.
Only an opaque random `chat-media/v1/...` object key and normalized metadata are
stored. The normalized SHA-256 binds retries and download integrity.

Message quota consumption, Message creation and all attachment rows share one
PostgreSQL transaction. R2 puts happen only after current authority, membership,
idempotency and rate-limit checks. A later failure rolls back PostgreSQL and
triggers immediate best-effort object deletion. Residual objects are identifiable
under the versioned prefix and `cleanupOrphanedChatMedia` deletes only old keys
that have no current `MessageAttachment` reference. Scheduling that cleanup is
deliberately outside this foundation PR.
