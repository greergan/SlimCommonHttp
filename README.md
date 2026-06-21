<a href="https://codeberg.org/greergan/SlimTS">
  <img src="https://raw.githubusercontent.com/greergan/SlimTS/master/assets/slimts_logo.png" width="75" alt="SlimTS Logo">
</a>   

# SlimCommonHttp

Shared HTTP support code for the [SlimCommon](https://codeberg.org/greergan/SlimCommon) library — error reporting types used by the cookie and header parsing libraries built on top of it.  
Dependency of [SlimCommonHttpCookie](https://codeberg.org/greergan/SlimCommonHttpCookie), [SlimCommonHttpCookieStore](https://codeberg.org/greergan/SlimCommonHttpCookieStore), and [SlimCommonHttpHeaders](https://codeberg.org/greergan/SlimCommonHttpHeaders).  
Built using [SlimLibraryPackager](https://codeberg.org/greergan/SlimLibraryPackager).  
CI/CD supplied by unified workflows provided by [SlimLibraryPackager](https://codeberg.org/greergan/SlimLibraryPackager).

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Core API](#core-api)
  - [ErrorStatus enum](#errorstatus-enum)
  - [HttpHeaderException class](#httpheaderexception-class)
- [Building](#building)
- [Dependencies](#dependencies)
- [Examples](#examples)

## Overview

This library is currently a single header, [`error_codes.h.in`](include/slim/common/http/error_codes.h.in), providing the shared vocabulary of error conditions used across SlimCommon's HTTP-related libraries:

- A single `ErrorStatus` enum covering cookie validation, cookie attribute parsing, and generic HTTP header parsing failures
- A `constexpr` lookup table mapping every `ErrorStatus` value to a human-readable description
- `HttpHeaderException`, a `std::logic_error`-derived exception type that carries an `ErrorStatus` alongside its message

[↑ Top](#table-of-contents)

## Features

| Feature | Description |
|--------|-------------|
| Unified error enum | Single `ErrorStatus` type shared by cookie and header parsing code, avoiding divergent per-library error types |
| Human-readable lookup | `error::status::to_string()` resolves any `ErrorStatus` to a descriptive string at compile time |
| Structured exceptions | `HttpHeaderException` pairs a thrown error with its originating `ErrorStatus` for programmatic handling |
| Version macros | `SLIMCOMMONHTTP_VERSION` and `SLIMCOMMONHTTP_GIT_HASH` are injected at build time |

[↑ Top](#table-of-contents)

## Core API

### ErrorStatus enum

All of `error_codes.h.in`'s lookup machinery is keyed off a single `ErrorStatus`. The full set of values is:

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

[↑ Top](#table-of-contents)
