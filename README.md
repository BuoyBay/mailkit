# Compose MIME messages and submit mail over SMTP

`Import mailkit` adds owned MIME messages and explicit SMTP submission. It has no
package dependencies or native bridge of its own. Copy
`packages/mailkit-0.1.0.buipkg` to a consumer's `packages/` directory. Importing,
constructing, serializing and parsing a message never connects or sends.

The [offline example](examples/mailkit/main.bui) constructs a Unicode
message with an attachment and parses its MIME representation again.

## Install

From a Buoy consumer project with BPM installed:

```sh
bpm add mailkit --git https://github.com/BuoyBay/mailkit.git --rev refs/heads/main
```

BPM records the resolved commit in its lockfile. Keep that lockfile with the
consumer project for reproducible installs.

## API at a glance

| API | Contract |
| --- | --- |
| `New MailMessage()` | Mutable message builder; set `Sender`, `Subject`, `TextBody` |
| `AddRecipient(mailbox)` | Append a visible To recipient; maximum 100 |
| `AddAttachment(attachment)` | Retain an immutable attachment; maximum 100 |
| `Recipients`, `Attachments` | Defensive array copies |
| `New MailAttachment(filename, mediaType, data, maxBytes = 8388608)` | Copy binary data; validate filename/media type and per-attachment budget |
| `attachment.Filename`, `MediaType`, `Size` | Read-only metadata |

## Build from source

With the native `build-packages` driver from [devtools](https://github.com/BuoyBay/devtools)
on your `PATH`, run from this checkout:

```sh
build-packages --root "$PWD" --compile-consumer mailkit
```

The driver checks source targets and compiles an import consumer. See the
[API guide](docs/API.md) for behavior, limits and any host prerequisites.

## Local Buoy setup

On your own machine, install Buoy and run:

```sh
buoy ai init --both
```

The generated agent files and editor settings stay local and are ignored by Git.

## Contributing

[Contributing](CONTRIBUTING.md) · [License](LICENSE)
