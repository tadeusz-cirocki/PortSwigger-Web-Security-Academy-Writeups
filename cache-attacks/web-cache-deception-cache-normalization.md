# Exploiting Cache Server Normalization for Web Cache Deception

## Lab: Exploiting cache server normalization for web cache deception

## Summary
A web cache deception vulnerability allowed an attacker to trick the cache into storing a sensitive, user-specific response under a cacheable static path. By abusing discrepancies in URL normalization and delimiter handling between the cache server and the origin server, it was possible to retrieve another user's API key.

---

## Root Cause
The cache server and the origin server normalized and interpreted the request URL differently:

- The cache server decoded and normalized encoded path segments and delimiters, applying a cache rule based on a static directory prefix.
- The origin server did not fully normalize the same encoded segments and delimiters, and therefore treated the request as a dynamic endpoint.

As a result, a sensitive response from a dynamic endpoint was cached as if it were a static resource.

---

## Exploitation Steps
1. Log in as a normal user (wiener:peter) and identify a sensitive dynamic endpoint:
   - `/my-account` returns the user's API key.

2. Identify cacheable static directories by observing responses with cache-related headers.
   - Requests under the `/resources` path are cached.

3. Test delimiter handling on the origin server and identify encoded characters that act as delimiters.
   - `%23` (encoded `#`) is interpreted as a delimiter by the origin server.

4. Test normalization behavior:
   - The origin server does not normalize encoded dot-segments such as `..%2f`.
   - The cache server *does* decode and normalize these segments.

5. Construct a malicious URL that:
   - Appears to the cache as a request for a static resource under `/resources`.
   - Is interpreted by the origin server as a request to `/my-account`.

   Payload used:
   https://0a6a001104e278a880dd08b000910029.web-security-academy.net/my-account%23%2f..%2fresources?asd

6. Deliver the URL to the victim user using an HTML element:
   - <img src="https://0a6a001104e278a880dd08b000910029.web-security-academy.net/my-account%23%2f..%2fresources?asd">

7. When the victim visits the URL, their account page is cached under the normalized `/resources` path.

8. Request the same URL as the attacker and retrieve the cached response containing the victim's API key.

---

## Impact
An attacker can steal sensitive, user-specific data such as API keys or personal information from other users without authentication, leading to account compromise and further attacks.

---

## Mitigation
- Do not cache responses from authenticated or user-specific endpoints.
- Ensure consistent URL normalization between cache servers and origin servers.
- Avoid cache rules that rely solely on file extensions or directory prefixes.
- Use appropriate cache-control headers such as `Cache-Control: private, no-store` on sensitive responses.

---

## Key Takeaway
Web cache deception can occur when caches and origin servers normalize and interpret encoded paths and delimiters differently, allowing sensitive dynamic content to be cached as static resources.
