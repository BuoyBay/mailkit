# MailKit implementation plan

Final serialization decisions: generate an RFC5322 Date in UTC, encode text and
attachments as base64 and Unicode subjects as folded UTF-8 encoded words. The API
has no Bcc field, so no Bcc header is emitted. This version does not generate a
Message-ID; submission servers may add it. Keep that interoperability limit
explicit rather than inventing identifiers from noncryptographic randomness.
The Linux verifier uses the existing BUI-301 TLS-link workaround, preserving the
original toolchain and testing a local trusted CA plus negative trust/hostname cases.

Implement a self-contained pure Buoy package: MailMessage (bare ASCII Sender and recipients, UTF-8 Subject/TextBody, immutable binary attachments), bounded RFC2047/base64 MIME serialization and a deliberately strict flat text/plain or multipart/mixed parser. Attachment filenames initially safe ASCII. Validate controls, mailbox syntax, MIME tokens and all budgets. Copy binary inputs/outputs; never implicitly send.

SMTP is an explicit submission call using TCPSocket, EHLO/envelope/DATA state transitions, multiline reply code consistency, CRLF/dot-stuffing and per-response monotonic deadlines. Default implicit TLS with certificate validation and explicit SNI hostname; an explicit insecure switch enables loopback/local relay testing. AUTH PLAIN only on TLS. No STARTTLS upgrade (no public socket upgrade primitive), DNS, POP3, IMAP, rich address syntax, arbitrary MIME tree or DKIM claims. TLS native handshake/write bounds are limited by Buoy runtime; document this.

Validation: real compiler JSON for every authored Buoy source and six desktop targets; sealed consumer native Linux execution; Python email parser oracle for generated UTF8 text/subject and binary attachments; malformed/injected/oversized MIME rejection tests; fake local SMTP transcript with split/multiline replies, recipient rejection, DATA body check, timeout and malformed reply; optional local TLS CA fixture if runtime supports it. No delivery to external servers. Package README and provenance state exact subset and native-validation boundaries.
