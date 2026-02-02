# Web Cache Deception via Path Delimiter Discrepancy

## Lab: Exploiting path delimiters for web cache deception

This write-up documents a Web Cache Deception vulnerability caused by inconsistent handling of path delimiter characters between the cache and the origin server.

---

## Summary

A private endpoint was cached when accessed through a URL containing a delimiter character.

The origin server served authenticated content, while the cache interpreted the request as a static resource and stored the response. This allowed an attacker to retrieve the victim’s cached account page and steal their API key.

---

## Root Cause

The cache and origin server disagreed on how certain characters act as path delimiters.

A crafted URL such as:

/my-account;asd.css

was processed differently:

- Origin server returned the real /my-account page
- Cache treated the URL as a cacheable static asset due to the .css extension

---

## Exploitation Steps

1. Identify the authenticated endpoint:

/my-account

2. Append a delimiter character:

/my-account;asd.css

3. Force the victim to visit the crafted URL using:

<img src="https://target.web-security-academy.net/my-account;asd.css">

4. Request the same URL within the cache TTL window.

5. Extract Carlos’s API key from the cached response.

---

## Impact

An attacker can leak sensitive authenticated content via the web cache, leading to exposure of API keys or personal data.

Severity: High (Sensitive Data Exposure)

---

## Mitigation

- Never cache authenticated responses
- Normalize URL parsing consistently across cache and origin
- Block delimiter characters like ';' in routing if unnecessary
- Use Cache-Control: private, no-store for sensitive pages

---

## Key Takeaway

Delimiter parsing discrepancies are another common way to trigger Web Cache Deception when caches and origin servers interpret URL boundaries differently.