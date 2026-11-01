---
title: "mailkit typed API"
summary: "Source-derived public signatures, properties, parameters, defaults, and return values."
category: "Reference"
order: 21
types: ["MailAttachment", "MailMessage", "SMTPException", "SMTPOptions"]
dependencies: []
symbols: ["MailAttachment", "MailMessage", "SMTPException", "SMTPOptions"]
related: ["api.md", "getting-started.md", "index.md"]
---

# mailkit typed API

Generated from the package source declarations. Platform guards are shown where present; inherited members are documented on the base type. See the [API contracts](api.md) for behavior and limitations.

## MailAttachment

`Public Class MailAttachment`

An owned immutable attachment; Data returns a defensive copy.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `Filename` | `String` | Yes | No | — | — |
| `MediaType` | `String` | Yes | No | — | — |
| `Size` | `Integer` | Yes | No | — | — |
| `Data` | `Bytes` | Yes | No | — | — |

### Constructor

`Public Sub Constructor(filename As String, mediaType As String, data As Bytes, maxBytes As Integer = 8388608)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `filename` | `String` | `ByVal` | Required |
| `mediaType` | `String` | `ByVal` | Required |
| `data` | `Bytes` | `ByVal` | Required |
| `maxBytes` | `Integer` | `ByVal` | `8388608` |

Returns: no value.

## MailMessage

`Public Class MailMessage`

A bounded UTF-8 text/plain message with optional binary attachments.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `Sender` | `String` | No | No | `""` | — |
| `Subject` | `String` | No | No | `""` | — |
| `TextBody` | `String` | No | No | — | — |
| `Recipients` | `Array Of String` | Yes | No | — | — |
| `Attachments` | `Array Of MailAttachment` | Yes | No | — | — |

### AddRecipient

`Public Sub AddRecipient(mailbox As String)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `mailbox` | `String` | `ByVal` | Required |

Returns: no value.

### AddAttachment

`Public Sub AddAttachment(attachment As MailAttachment)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `attachment` | `MailAttachment` | `ByVal` | Required |

Returns: no value.

### Serialize

`Public Function Serialize(maxBytes As Integer = 16777216) As Bytes`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `maxBytes` | `Integer` | `ByVal` | `16777216` |

#### Returns

| Type |
| --- |
| `Bytes` |

## MailKit functions

### ParseMailMessage

`Public Function ParseMailMessage(data As Bytes, maxBytes As Integer = 16777216) As MailMessage`

Parse the documented ASCII-wire, UTF8/base64 flat MIME subset; no files are written.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `data` | `Bytes` | `ByVal` | Required |
| `maxBytes` | `Integer` | `ByVal` | `16777216` |

#### Returns

| Type |
| --- |
| `MailMessage` |

### SMTPDataBlock

`Public Function SMTPDataBlock(wire As Bytes, maxBytes As Integer = 16777216) As Bytes`

Validate CRLF SMTP data, dot-stuff it and append the terminating period line.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `wire` | `Bytes` | `ByVal` | Required |
| `maxBytes` | `Integer` | `ByVal` | `16777216` |

#### Returns

| Type |
| --- |
| `Bytes` |

### SubmitMail

`Public Function SubmitMail(message As MailMessage, options As SMTPOptions) As String`

Submit one message. Returns the final DATA acceptance reply; failures after DATA can be ambiguous.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `message` | `MailMessage` | `ByVal` | Required |
| `options` | `SMTPOptions` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `String` |

## SMTPException

`Public Class SMTPException Inherits Exception`

SMTP response errors retain the numeric server code without exposing credentials.

### Constructor

`Public Sub Constructor(message As String, responseCode As Integer = 0)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `message` | `String` | `ByVal` | Required |
| `responseCode` | `Integer` | `ByVal` | `0` |

Returns: no value.

## SMTPOptions

`Public Class SMTPOptions`

Explicit SMTP transport configuration. TLS is required unless AllowInsecure is opted into.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `IPv4Address` | `String` | No | No | `"127.0.0.1"` | — |
| `Port` | `Integer` | No | No | `465` | — |
| `ImplicitTLS` | `Boolean` | No | No | `True` | — |
| `AllowInsecure` | `Boolean` | No | No | `False` | — |
| `ServerName` | `String` | No | No | `""` | — |
| `TrustedCaPath` | `String` | No | No | `""` | — |
| `EHLOName` | `String` | No | No | `"localhost"` | — |
| `TimeoutMs` | `Integer` | No | No | `10000` | — |
| `MaxMessageBytes` | `Integer` | No | No | `16777216` | — |
| `Username` | `String` | No | No | `""` | — |
| `Password` | `String` | No | No | `""` | — |
