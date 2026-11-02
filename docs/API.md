# MailKit 0.1.0

`Import mailkit` adds owned MIME messages and explicit SMTP submission. It has no
package dependencies or native bridge of its own. Copy
`packages/mailkit-0.1.0.buipkg` to a consumer's `packages/` directory. Importing,
constructing, serializing and parsing a message never connects or sends.

The [offline example](../examples/mailkit/main.bui) constructs a Unicode
message with an attachment and parses its MIME representation again.

## Message API

| API | Contract |
| --- | --- |
| `New MailMessage()` | Mutable message builder; set `Sender`, `Subject`, `TextBody` |
| `AddRecipient(mailbox)` | Append a visible To recipient; maximum 100 |
| `AddAttachment(attachment)` | Retain an immutable attachment; maximum 100 |
| `Recipients`, `Attachments` | Defensive array copies |
| `New MailAttachment(filename, mediaType, data, maxBytes = 8388608)` | Copy binary data; validate filename/media type and per-attachment budget |
| `attachment.Filename`, `MediaType`, `Size` | Read-only metadata |
| `attachment.Data` | Defensive byte copy; embedded NUL and arbitrary bytes are valid |
| `message.Serialize(maxBytes = 16777216)` | Return MIME `Bytes`, including a current UTC Date header |
| `ParseMailMessage(data, maxBytes = 16777216)` | Parse the supported subset into a fresh message; never extract files |
| `SMTPDataBlock(wire, maxBytes = 16777216)` | Validate ASCII/CRLF and 998-octet data lines, dot-stuff, terminate with a period line |

Sender and recipients must be bare ASCII dot-atom mailboxes with DNS-style domain
labels. Display names, comments, quoted local parts, address literals, Cc/Bcc and
SMTPUTF8 addresses are unsupported. Attachment filenames allow 1–200 ASCII
letters, digits, spaces, dots, underscores and hyphens, excluding `.` and `..`;
media types use a restricted type/subtype token of at most 127 characters. These
are attachment metadata; the package never turns a filename into a disk path.

Subjects are UTF-8 with no control characters and at most 4096 encoded bytes.
Serialization emits folded UTF-8/base64 encoded words at scalar boundaries.
Assigning `TextBody` normalizes CR, LF and CRLF line endings to CRLF, the
canonical MIME text representation before base64. Generated MIME wire lines also
use CRLF. A message with attachments uses flat `multipart/mixed`,
with text/plain first and base64 attachments after it. The fixed boundary contains
characters unavailable in base64 payload lines, so generated bodies cannot collide
with it. No attachment is executed, loaded or rendered.

The parser accepts ASCII-wire MIME-Version 1.0 messages with a UTF-8/base64
text/plain body, or a flat multipart/mixed container with that text part followed
by base64 attachments. Subjects may be plain ASCII or whitespace-separated
UTF-8/base64 encoded words. It accepts simple quoted/unquoted charset and boundary
parameters, but escaped quotes, parameter continuations, RFC2231 filenames,
quoted-printable, other charsets, nested/alternative MIME, nonempty preambles and
nonempty epilogues are outside this version. Duplicate headers and relevant
parameters, malformed/noncanonical base64, missing/mismatched boundaries and bare
CR/LF reject. Decoded textual NUL rejects because the current Buoy String runtime
cannot preserve it; attachment NUL remains valid. An attachment `Content-Type` must
be a bare `type/subtype`; common real-world forms such as
`application/pdf; name="a.pdf"` reject, so arbitrary received mail may not parse.

Parsing retains the supported message fields and attachments. Other headers,
original Date, wire folding and MIME metadata are discarded; serializing a parsed
message creates a new Date. This is a focused message API, not a lossless email
archive or a general incoming-mail parser.

## SMTP submission

`SubmitMail(message, options As SMTPOptions) As String` opens one connection,
reads the greeting, sends EHLO, optionally authenticates, submits the envelope and
DATA, and closes. Every recipient must return 250/251 before DATA is sent. The
returned String is the final 250 DATA acceptance reply; it means the SMTP server
accepted responsibility, not that a recipient received or read the message.
A lost connection or failure while awaiting DATA acceptance can leave delivery
ambiguous. The caller owns retry policy; this package never retries automatically.

| SMTPOptions property | Default / behavior |
| --- | --- |
| `IPv4Address` | `127.0.0.1`; a literal IPv4 address, as required by Buoy TCPSocket |
| `Port` | `465`; valid range 1–65535 |
| `ImplicitTLS` | `True`; TLS begins when connecting, with certificate verification enabled |
| `ServerName` | Empty; must be set for TLS SNI and certificate hostname verification |
| `TrustedCaPath` | Empty uses OS trust; otherwise a PEM trust path passed to TCPSocket |
| `AllowInsecure` | `False`; must be explicitly True for plaintext relay/test use |
| `EHLOName` | `localhost`; ASCII letters, digits, dots and hyphens, 1–253 characters |
| `TimeoutMs` | `10000`; 1–300000, per-reply monotonic deadline and TCP connect/plain write bound |
| `MaxMessageBytes` | `16777216`; MIME input/output budget |
| `Username`, `Password` | Empty means no auth; up to 120 UTF-8 bytes each, without controls |

Authentication uses an initial AUTH PLAIN response only when implicit TLS is
active and EHLO advertised PLAIN. Plaintext authentication is rejected before
connecting. There is no certificate-verification bypass setting. Credential
normalization is the caller's responsibility; SASLprep, AUTH challenge flows,
LOGIN and OAuth mechanisms are not implemented. Protocol failures throw
`SMTPException`; server rejections retain their SMTP code in `ErrorCode` without
including the response text or credentials. Socket and TLS failures propagate as
Buoy `SocketException` / `TLSException`. Validation and unsupported MIME forms
throw `InvalidArgumentException`; malformed UTF-8 may throw `EncodingException`.

Replies are limited to 512 octets per line and 64 lines per response; multiline
codes must agree. Replies are read in order and are not checked against the
command they answer, so a peer that sends replies early (for example `250` for DATA
before the body) is accepted. Over an `AllowInsecure` plaintext connection a
successful result therefore cannot be authenticated; use implicit TLS when
acceptance matters. Submission is synchronous. Use a worker for UI applications,
and keep each builder/options object on one thread or synchronize externally.

**Transport limits:** Buoy's public socket API has no STARTTLS upgrade primitive,
so STARTTLS is unsupported; choose an implicit-TLS endpoint. There is no DNS
resolution, SMTP pooling, queue, POP3, IMAP, DKIM or delivery-status implementation.
The runtime bounds TCP connect and plaintext writes but does not bound TLS
handshake/writes/cleanup with `TimeoutMs`. MailKit's reply deadline bounds its
own polling only; it is not an overall TLS-operation deadline. The native runtime
also buffers incoming bytes in its background reader before the package drains
and validates them, so response limits are parser limits, not a hard socket-memory
cap. See Buoy's `tcpsocket.md` and `streamsocket.md` contracts.

## Budgets and native prerequisites

Limits must be 1–33554432 bytes. Serialization checks raw bytes before base64
work and final MIME size before returning. Parsing checks input size before
decoding and bounds total decoded data. Header blocks are capped at 32768 bytes
and 100 fields. This is whole-message processing with bounded copies, not a
streaming or hard peak-memory allocator.

Attachment limits apply to each copied attachment. SMTPDataBlock can expand its
input through dot-stuffing, a missing final CRLF and the three-byte terminator.

On affected Linux Buoy 0.72 installations, SMTP consumers must explicitly link
the installed TLS/SSL/Crypto archives and matching runtime stubs. Provision those
toolchain components for the deployment host; they are separate from MailKit's
source-only MIME and SMTP protocol code.
