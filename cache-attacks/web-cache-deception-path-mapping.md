# Web Cache Deception via Path Mapping Discrepancy

## Lab: Exploiting path mapping for web cache deception

This write-up documents a **Web Cache Deception** vulnerability caused by inconsistent path mapping, allowing sensitive authenticated content to be cached and later accessed by an attacker.

---

## Summary

The application exposed a private endpoint (`/my-account`) through an alternative URL path containing a static file extension.

Because the caching layer treated requests ending in `.css` as static assets, it stored the authenticated response. This allowed an attacker to retrieve sensitive user information (API key) directly from the cache.

---

## Root Cause

The authenticated endpoint:

- `/my-account`

was also accessible through:

- `/my-account/<arbitrary>.css`

This created a **path mapping discrepancy**, where the origin server returned the same dynamic account page, but the caching layer classified the URL as a static resource due to the `.css` extension.

As a result, private user-specific responses were cached.

---

## Cache Behavior

The response was stored by the cache with a short TTL:

Cache-Control: max-age=30

This meant that for ~30 seconds, subsequent requests to the same URL could receive the cached private content, even without authentication.

---

## Exploitation Steps

1. Identify the private endpoint: `/my-account`
2. Discover an alternative cached path: `/my-account/asd.css`
3. Trigger the victim to populate the cache using:

   `<img src="https://target.web-security-academy.net/my-account/asd.css">`

4. Retrieve the cached sensitive data within the TTL window.
5. Extract the API key from the cached HTML response.

---

## Impact

This vulnerability allows an attacker to leak sensitive user-specific data from cached responses, bypass authentication indirectly, and steal API keys or personal information.

Severity: High (Sensitive Data Exposure)

---

## Mitigation

- Never cache authenticated or user-specific responses
- Configure caching rules to ignore file extensions in dynamic routes
- Ensure consistent routing between cache and origin
- Add appropriate headers for private content:

Cache-Control: no-store, private

- Use strict allowlists for cacheable static resources

---

## Key Takeaway

Web Cache Deception occurs when caches and origin servers interpret URL paths differently, allowing private dynamic content to be stored and served as if it were a public static asset.
