# MailKit provenance

Independent implementation in Buoy, developed from the public language/runtime
APIs and protocol specifications. No third-party email-library source or native
binaries are copied into this package.

Primary references checked on 2026-10-31:

- [RFC 5321](https://www.rfc-editor.org/rfc/rfc5321): SMTP reply framing,
  EHLO, envelope, DATA transparency and command/reply limits.
- [RFC 2045](https://www.rfc-editor.org/rfc/rfc2045): MIME headers and
  base64 content-transfer encoding.
- [RFC 2046](https://www.rfc-editor.org/rfc/rfc2046): multipart/mixed structure.
- [RFC 2047](https://www.rfc-editor.org/rfc/rfc2047): UTF-8 base64 encoded
  Subject words and folding.
- [RFC 4616](https://www.rfc-editor.org/rfc/rfc4616): SASL PLAIN payload shape.
- Installed Buoy 0.72 documentation: `bytes.md`, `tcpsocket.md`,
  `streamsocket.md`, `microseconds.md`, `datetime.md`, `command-line-args.md`.

These inform the documented subset, not a claim to full RFC conformance.
Buoy owns TLS, trust-store lookup, sockets and base64 primitives. Package tests
use Python's standard-library email parser as an independent MIME oracle and
Python ssl/socket local servers as protocol peers. Test credentials are fixed,
public fixture values. TLS certificates/private keys are generated temporarily by
OpenSSL for each verifier run and removed with the temporary directory. They are
never installed or retained as trust material.
