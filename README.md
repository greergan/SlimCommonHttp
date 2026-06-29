<a href="https://codeberg.org/greergan/SlimTS">
  <img src="https://raw.githubusercontent.com/greergan/SlimTS/master/assets/slimts_logo.png" width="75" alt="SlimTS Logo">
</a>   

# SlimCommonHttp

Shared HTTP support code for the [SlimCommon](https://codeberg.org/greergan/SlimCommon) library — error reporting and status code types used by the cookie, header, URL, and response parsing libraries built on top of it.  
Dependency of [SlimCommonHttpCookie](https://codeberg.org/greergan/SlimCommonHttpCookie), [SlimCommonHttpCookieStore](https://codeberg.org/greergan/SlimCommonHttpCookieStore), and [SlimCommonHttpHeaders](https://codeberg.org/greergan/SlimCommonHttpHeaders).  
Built using [SlimLibraryPackager](https://codeberg.org/greergan/SlimLibraryPackager).  
CI/CD supplied by unified workflows provided by [SlimLibraryPackager](https://codeberg.org/greergan/SlimLibraryPackager).

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Core API](#core-api)
  - [ErrorStatus enum](#errorstatus-enum)
  - [HttpHeaderException class](#httpheaderexception-class)
  - [UrlParseException class](#urlparseexception-class)
  - [ResponseParseException class](#responseparseexception-class)
  - [StatusCode enum](#statuscode-enum)
- [Building](#building)
- [Dependencies](#dependencies)
- [Examples](#examples)

## Overview

This library provides the shared vocabulary of error conditions and HTTP status codes used across SlimCommon's HTTP-related libraries:

- A single `ErrorStatus` enum covering cookie validation, cookie attribute parsing, generic HTTP header parsing, URL parsing, request status-line parsing, and HTTP response parsing failures, in [`error_codes.h.in`](include/slim/common/http/error_codes.h.in)
- A `constexpr` lookup table mapping every `ErrorStatus` value to a human-readable description
- `HttpHeaderException`, a `std::logic_error`-derived exception type that carries an `ErrorStatus` alongside its message
- `UrlParseException`, a specialized `HttpHeaderException` subtype for URL parsing failures
- `ResponseParseException`, a specialized `HttpHeaderException` subtype for HTTP response parsing failures
- A `StatusCode` enum covering the standard HTTP response status codes, in [`status_codes.h.in`](include/slim/common/http/status_codes.h.in)
- A `constexpr` lookup table mapping every `StatusCode` value to its standard reason phrase

[↑ Top](#table-of-contents)

## Features

| Feature | Description |
|--------|-------------|
| Unified error enum | Single `ErrorStatus` type shared by cookie, header, URL, request, and response parsing code, avoiding divergent per-library error types |
| Human-readable lookup | `error::status::to_string()` resolves any `ErrorStatus` to a descriptive string at compile time |
| Structured exceptions | `HttpHeaderException` pairs a thrown error with its originating `ErrorStatus` for programmatic handling |
| URL-specific exception | `UrlParseException` distinguishes URL parsing failures from other HTTP errors while reusing the same `ErrorStatus`/`error()` machinery |
| Response-specific exception | `ResponseParseException` distinguishes HTTP response parsing failures from other HTTP errors while reusing the same `ErrorStatus`/`error()` machinery |
| HTTP status codes | `StatusCode` enum covers the standard 1xx-5xx HTTP response status codes |
| Reason phrase lookup | `status::code::to_string()` resolves any `StatusCode` to its standard reason phrase at compile time |
| Version macros | `SLIMCOMMONHTTP_VERSION` and `SLIMCOMMONHTTP_GIT_HASH` are injected at build time |

[↑ Top](#table-of-contents)

## Core API

### ErrorStatus enum

All of [`error_codes.h.in`](include/slim/common/http/error_codes.h.in)'s lookup machinery is keyed off a single `ErrorStatus`. The full set of values is:

| Value | Meaning |
|-------|---------|
| `OK` | Success |
| `CookieDomainBareDot` | Cookie domain cannot consist of a bare dot |
| `CookieDomainEmpty` | Cookie domain name cannot be empty |
| `CookieDomainInvalidChar` | Cookie domain contains an invalid character |
| `CookieDomainLabelEmpty` | Cookie domain label cannot be empty |
| `CookieDomainLabelInvalidHyphen` | Cookie domain label has a hyphen in an invalid position |
| `CookieDomainLabelTooLong` | Cookie domain label exceeds the maximum allowed length |
| `CookieDomainNumericTld` | Cookie domain's top-level domain (TLD) must not be numeric |
| `CookieDomainTooLong` | Cookie domain name exceeds the maximum allowed length |
| `CookieDomainTrailingDot` | Cookie domain name must not end with a trailing dot |
| `CookieEmptyString` | Cookie string cannot be empty |
| `CookieExpiresInvalidFormat` | Cookie Expires attribute has an invalid date format |
| `CookieInvalidBoolean` | Cookie attribute contains an invalid boolean value |
| `CookieInvalidName` | Cookie name failed validation |
| `CookieInvalidValue` | Cookie value failed validation |
| `CookieMalformedMissingEquals` | Cookie is missing a `=` separator |
| `CookieMalformedPairMissingEquals` | Cookie pair is missing a `=` separator |
| `CookieMaxAgeEmpty` | Cookie Max-Age attribute cannot be empty |
| `CookieMaxAgeExceedsLimit` | Cookie Max-Age value exceeds the maximum allowed limit |
| `CookieMaxAgeInvalidFormat` | Cookie Max-Age attribute has an invalid format |
| `CookieMaxAgeTrailingChars` | Cookie Max-Age value has unexpected trailing characters |
| `CookieNameEmpty` | Cookie name cannot be empty |
| `CookieNameHostPrefixHasDomain` | Cookie `__Host-` prefix cannot be combined with a Domain attribute |
| `CookieNameHostPrefixInvalidPath` | Cookie `__Host-` prefix requires the Path attribute to be `/` |
| `CookieNameInvalidChar` | Cookie name contains an invalid character |
| `CookieNamePrefixRequiresSecure` | Cookie name prefix requires the Secure attribute |
| `CookiePartitionedRequiresSameSiteNone` | Partitioned cookies require `SameSite=None` |
| `CookiePartitionedRequiresSecure` | Partitioned cookies require the Secure attribute |
| `CookiePathInvalidChar` | Cookie path contains an invalid character |
| `CookiePathMissingLeadingSlash` | Cookie path must begin with a forward slash (`/`) |
| `CookieSameSiteInvalid` | Cookie SameSite attribute has an invalid value |
| `CookieSameSiteNoneRequiresSecure` | Cookie `SameSite=None` requires the Secure attribute |
| `CookieTooLarge` | Cookie exceeds the maximum allowed size |
| `CookieValueInvalidChar` | Cookie value contains an invalid character |
| `CookieValueUnmatchedQuote` | Cookie value has an unmatched quotation mark |
| `HeaderDelimiterInvalid` | Header contains an invalid delimiter |
| `HeaderNameEmpty` | Header name cannot be empty |
| `HeaderNameInvalidChar` | Header name contains an invalid character |
| `HeaderValueEmpty` | Header value cannot be empty |
| `HeaderValueInvalidChar` | Header value contains an invalid character |
| `HeaderValueInvalidFolding` | Header value contains obsolete line folding (obs-fold) |
| `UrlStringEmpty` | URL string cannot be empty |
| `UrlInvalidControlCharacter` | URL contains an invalid control character |
| `UrlSchemeInvalidCharacter` | URL scheme contains an invalid character |
| `UrlSchemeDelimiterMissing` | URL scheme delimiter not found (expected `://`) |
| `UrlSchemeUnsupported` | URL scheme is unsupported |
| `UrlHostInvalidStart` | URL host must begin with alphanumeric data |
| `UrlHostInvalidCharacter` | URL host contains an invalid character |
| `UrlHostMissing` | URL must contain a valid host[:port] |
| `UrlPortInvalidCharacter` | URL port contains an invalid character |
| `UrlBodyInvalidCharacter` | URL body contains an invalid character |
| `UrlFilePathMissing` | `file://` URL must contain a path (e.g. `file:///etc/hosts`) |
| `UrlFilePathTrailingSlash` | `file://` URL path must end with a filename, not `/` |
| `UrlUnparsable` | URL is unparsable |
| `RequestStringEmpty` | Request string cannot be empty |
| `RequestStatusLineInvalid` | Request status line is unparsable |
| `RequestStatusLineMalformed` | Request status line is malformed |
| `RequestHeadersNotTerminated` | Request header section is not terminated |
| `ResponseStorageEmpty` | Response storage is empty |
| `ResponseStatusLineInvalid` | Response status line is unparsable |
| `ResponseStatusCodeInvalid` | Response status code is invalid |
| `ResponseStatusCodeOutOfRange` | Response status code is out of range (must be 100-599) |
| `ResponseHeadersTerminatorMalformed` | Response header section terminator is malformed |
| `ResponseHeadersBareCR` | Response headers contain a bare CR |
| `ResponseHeadersNotTerminated` | Response header section is not terminated |
| `ResponseChunkedMissingCRLF` | Chunked response is missing a required CRLF |
| `ResponseChunkedSizeInvalid` | Chunked response contains an invalid chunk size |
| `ResponseChunkedTruncated` | Chunked response chunk size exceeds remaining payload |
| `ResponseChunkedMissingCRLFAfterData` | Chunked response is missing CRLF after chunk data |
| `ResponseContentLengthInvalid` | Response Content-Length value is invalid |
| `ResponseContentLengthMismatch` | Response Content-Length exceeds available body bytes |

`ErrorStatus::END` is a sentinel marking the end of the enum and is not a valid status value.

Status values can be converted to a human-readable string via:

```cpp
std::string_view msg = slim::common::http::error::status::to_string(status);
```

[↑ Top](#table-of-contents)

### HttpHeaderException class

```cpp
class HttpHeaderException : public std::logic_error {
public:
    explicit HttpHeaderException(ErrorStatus e);
    explicit HttpHeaderException(std::string msg);

    ErrorStatus error() const noexcept;
};
```

| Method | Description |
|--------|-------------|
| `HttpHeaderException(ErrorStatus e)` | Constructs the exception from an `ErrorStatus`, using its looked-up description as the `what()` message |
| `HttpHeaderException(std::string msg)` | Constructs the exception from a free-form message, with `error()` defaulting to `ErrorStatus::OK` |
| `ErrorStatus error() const noexcept` | Returns the `ErrorStatus` associated with this exception |

[↑ Top](#table-of-contents)

### UrlParseException class

```cpp
class UrlParseException : public HttpHeaderException {
public:
    explicit UrlParseException(ErrorStatus e);
    explicit UrlParseException(std::string msg);
};
```

`UrlParseException` derives from `HttpHeaderException` and inherits its constructors, `what()`, and `error()` behavior unchanged. It exists purely as a distinct type so callers can separate URL parsing failures from other HTTP errors with a dedicated `catch` clause, while code that only cares about the general case can still catch `HttpHeaderException` (or `std::logic_error`) and handle both uniformly.

| Method | Description |
|--------|-------------|
| `UrlParseException(ErrorStatus e)` | Constructs the exception from an `ErrorStatus`, using its looked-up description as the `what()` message |
| `UrlParseException(std::string msg)` | Constructs the exception from a free-form message, with `error()` defaulting to `ErrorStatus::OK` |

[↑ Top](#table-of-contents)

### ResponseParseException class

```cpp
class ResponseParseException : public HttpHeaderException {
public:
    explicit ResponseParseException(ErrorStatus e);
    explicit ResponseParseException(std::string msg);
};
```

`ResponseParseException` derives from `HttpHeaderException` and inherits its constructors, `what()`, and `error()` behavior unchanged. It exists purely as a distinct type so callers can separate HTTP response parsing failures (status line, headers, or body framing) from other HTTP errors with a dedicated `catch` clause, while code that only cares about the general case can still catch `HttpHeaderException` (or `std::logic_error`) and handle both uniformly.

| Method | Description |
|--------|-------------|
| `ResponseParseException(ErrorStatus e)` | Constructs the exception from an `ErrorStatus`, using its looked-up description as the `what()` message |
| `ResponseParseException(std::string msg)` | Constructs the exception from a free-form message, with `error()` defaulting to `ErrorStatus::OK` |

[↑ Top](#table-of-contents)

### StatusCode enum

[`status_codes.h.in`](include/slim/common/http/status_codes.h.in) provides `StatusCode`, an `enum class` covering the standard HTTP response status codes (1xx informational through 5xx server error), along with a lookup table mapping each value to its standard reason phrase:

| Range | Examples |
|-------|----------|
| 1xx Informational | `Continue`, `SwitchingProtocols`, `Processing`, `EarlyHints` |
| 2xx Success | `OK`, `Created`, `Accepted`, `NoContent`, `PartialContent` |
| 3xx Redirection | `MovedPermanently`, `Found`, `NotModified`, `TemporaryRedirect`, `PermanentRedirect` |
| 4xx Client Error | `BadRequest`, `Unauthorized`, `Forbidden`, `NotFound`, `ImATeapot`, `TooManyRequests` |
| 5xx Server Error | `InternalServerError`, `NotImplemented`, `BadGateway`, `ServiceUnavailable`, `GatewayTimeout` |

The full set of values is:

| Code | Value | Reason Phrase |
|------|-------|----------------|
| `Continue` | 100 | Continue |
| `SwitchingProtocols` | 101 | Switching Protocols |
| `Processing` | 102 | Processing |
| `EarlyHints` | 103 | Early Hints |
| `OK` | 200 | OK |
| `Created` | 201 | Created |
| `Accepted` | 202 | Accepted |
| `NonAuthoritativeInformation` | 203 | Non-Authoritative Information |
| `NoContent` | 204 | No Content |
| `ResetContent` | 205 | Reset Content |
| `PartialContent` | 206 | Partial Content |
| `MultiStatus` | 207 | Multi-Status |
| `AlreadyReported` | 208 | Already Reported |
| `IMUsed` | 226 | IM Used |
| `MultipleChoices` | 300 | Multiple Choices |
| `MovedPermanently` | 301 | Moved Permanently |
| `Found` | 302 | Found |
| `SeeOther` | 303 | See Other |
| `NotModified` | 304 | Not Modified |
| `UseProxy` | 305 | Use Proxy |
| `TemporaryRedirect` | 307 | Temporary Redirect |
| `PermanentRedirect` | 308 | Permanent Redirect |
| `BadRequest` | 400 | Bad Request |
| `Unauthorized` | 401 | Unauthorized |
| `PaymentRequired` | 402 | Payment Required |
| `Forbidden` | 403 | Forbidden |
| `NotFound` | 404 | Not Found |
| `MethodNotAllowed` | 405 | Method Not Allowed |
| `NotAcceptable` | 406 | Not Acceptable |
| `ProxyAuthenticationRequired` | 407 | Proxy Authentication Required |
| `RequestTimeout` | 408 | Request Timeout |
| `Conflict` | 409 | Conflict |
| `Gone` | 410 | Gone |
| `LengthRequired` | 411 | Length Required |
| `PreconditionFailed` | 412 | Precondition Failed |
| `PayloadTooLarge` | 413 | Payload Too Large |
| `UriTooLong` | 414 | URI Too Long |
| `UnsupportedMediaType` | 415 | Unsupported Media Type |
| `RangeNotSatisfiable` | 416 | Range Not Satisfiable |
| `ExpectationFailed` | 417 | Expectation Failed |
| `ImATeapot` | 418 | I'm a teapot |
| `MisdirectedRequest` | 421 | Misdirected Request |
| `UnprocessableEntity` | 422 | Unprocessable Entity |
| `Locked` | 423 | Locked |
| `FailedDependency` | 424 | Failed Dependency |
| `TooEarly` | 425 | Too Early |
| `UpgradeRequired` | 426 | Upgrade Required |
| `PreconditionRequired` | 428 | Precondition Required |
| `TooManyRequests` | 429 | Too Many Requests |
| `RequestHeaderFieldsTooLarge` | 431 | Request Header Fields Too Large |
| `UnavailableForLegalReasons` | 451 | Unavailable For Legal Reasons |
| `InternalServerError` | 500 | Internal Server Error |
| `NotImplemented` | 501 | Not Implemented |
| `BadGateway` | 502 | Bad Gateway |
| `ServiceUnavailable` | 503 | Service Unavailable |
| `GatewayTimeout` | 504 | Gateway Timeout |
| `HttpVersionNotSupported` | 505 | HTTP Version Not Supported |
| `VariantAlsoNegotiates` | 506 | Variant Also Negotiates |
| `InsufficientStorage` | 507 | Insufficient Storage |
| `LoopDetected` | 508 | Loop Detected |
| `NotExtended` | 510 | Not Extended |
| `NetworkAuthenticationRequired` | 511 | Network Authentication Required |

Status codes can be converted to their standard reason phrase via:

```cpp
std::string_view phrase = slim::common::http::status::code::to_string(code);
```

Codes with no defined reason phrase in the lookup table resolve to `"Unknown Status"`.

[↑ Top](#table-of-contents)

## Building

This library is built using [SlimLibraryPackager](https://codeberg.org/greergan/SlimLibraryPackager). See that repository for build instructions.

[↑ Top](#table-of-contents)

## Dependencies

External package dependencies for this library are declared in the [`required_packages`](required_packages) file at the repository root. This file is read by [SlimLibraryPackager](https://codeberg.org/greergan/SlimLibraryPackager) during the build process to resolve dependencies and install them if not present.

```
none
```

[↑ Top](#table-of-contents)

## Examples

```cpp
// Throwing and catching an HttpHeaderException
try {
    throw slim::common::http::HttpHeaderException(
        slim::common::http::ErrorStatus::HeaderValueInvalidFolding
    );
}
catch (const slim::common::http::HttpHeaderException& e) {
    std::cerr << "HTTP error: " << e.what() << '\n';
    if (e.error() == slim::common::http::ErrorStatus::HeaderValueInvalidFolding) {
        // handle obsolete line folding specifically
    }
}
```

```cpp
// Throwing and catching a UrlParseException
try {
    throw slim::common::http::UrlParseException(
        slim::common::http::ErrorStatus::UrlSchemeUnsupported
    );
}
catch (const slim::common::http::UrlParseException& e) {
    std::cerr << "URL error: " << e.what() << '\n';
    if (e.error() == slim::common::http::ErrorStatus::UrlSchemeUnsupported) {
        // handle unsupported scheme specifically
    }
}
```

```cpp
// Throwing and catching a ResponseParseException
try {
    throw slim::common::http::ResponseParseException(
        slim::common::http::ErrorStatus::ResponseContentLengthMismatch
    );
}
catch (const slim::common::http::ResponseParseException& e) {
    std::cerr << "Response error: " << e.what() << '\n';
    if (e.error() == slim::common::http::ErrorStatus::ResponseContentLengthMismatch) {
        // handle Content-Length mismatch specifically
    }
}
```

```cpp
// Looking up the reason phrase for a StatusCode
slim::common::http::StatusCode code = slim::common::http::StatusCode::NotFound;
std::string_view phrase = slim::common::http::status::code::to_string(code);
std::cout << static_cast<int>(code) << ' ' << phrase << '\n'; // "404 Not Found"
```

[↑ Top](#table-of-contents)
