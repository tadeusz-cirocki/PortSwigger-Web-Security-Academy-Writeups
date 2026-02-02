# Web Cache Deception via Origin Server Normalization

## Lab: Exploiting origin server normalization for web cache deception

This write-up documents a Web Cache Deception vulnerability caused by inconsistent URL decoding and path normalization between the caching layer and the origin server.

---

## Summary

The application contained a private authenticated endpoint:

/my-account

The caching layer also applied a static directory caching rule for:

/resources/

By using an encoded path traversal sequence inside this cached directory, it was possible to trick the cache into storing the victim’s account page as if it were a public static resource.

This allowed retrieval of Carlos’s API key from the cached response.

---

## Root Cause

The cache and origin server interpreted the same URL differently.

The crafted payload was:

/resources/..%2fmy-account

Key discrepancy:

- The cache did not decode or normalize the path.
- The origin server decoded and normalized it before routing.

As a result:

- Cache believed the request was for a cacheable resource under `/resources/`
- Origin resolved the request to `/my-account` and returned sensitive content

---

## Why Encoding Was Required

A direct traversal such as:

/resources/../my-account

is often detected or normalized by caches and therefore not cached.

However, using the encoded slash:

%2f = /

allowed the traversal to bypass cache normalization:

/resources/..%2fmy-account
→ cache sees literal path under /resources/
/resources/../my-account
→ origin decodes and normalizes to /my-account

This difference enabled the deception attack.

---

## Exploitation Steps

### 1. Identify the private endpoint

The authenticated endpoint containing sensitive data:

/my-account

### 2. Find a cached static directory

The cache stored responses for paths under:

/resources/

### 3. Craft a normalization discrepancy payload

Escape the static directory using encoded traversal:

/resources/..%2fmy-account

Full target URL:

https://TARGET.web-security-academy.net/resources/..%2fmy-account

### 4. Deliver the URL to the victim

Force Carlos to request the crafted URL via an exploit payload:

<img src="https://TARGET.web-security-academy.net/resources/..%2fmy-account">

### 5. Retrieve the cached response

Within the cache TTL window, request the same URL again.

The response contained Carlos’s account page, including his API key.

---

## Impact

An attacker can cause sensitive authenticated pages to be cached and served publicly, resulting in:

- API key leakage
- Exposure of private user data
- Potential account compromise

Severity: High

---

## Mitigation

To prevent normalization-based Web Cache Deception:

- Never cache authenticated responses
- Apply consistent decoding and normalization across cache and origin
- Avoid caching rules based purely on URL prefixes (e.g. `/resources/`)
- Use strict cache directives for private content:

Cache-Control: private, no-store

- Block encoded traversal sequences such as `..%2f`

---

## Key Takeaway

Web Cache Deception can be exploited when the cache fails to decode or normalize URLs, while the origin server does, allowing private dynamic content to be stored under cacheable static paths.
