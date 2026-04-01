# SSRF with Blacklist-Based Input Filter Bypass

## Summary

The application’s stock check functionality is vulnerable to Server-Side Request Forgery (SSRF). Although basic defenses are implemented using a blacklist to block access to internal resources, these protections are insufficient and can be bypassed באמצעות URL obfuscation techniques. This allows attackers to access the internal admin interface and perform privileged actions.

## Root Cause

The application attempts to prevent SSRF by applying **blacklist-based filtering**, blocking:

* Explicit references to `127.0.0.1`
* Sensitive paths such as `/admin`

However, this approach is flawed because:

* Blacklists are inherently incomplete.
* Input validation does not account for alternative IP representations.
* URL decoding is inconsistently applied, allowing encoded payloads to bypass filters.

As a result, attackers can evade detection using obfuscation techniques such as:

* Alternative IP formats (`127.1`)
* Double URL encoding (`%2561` → `%61` → `a`)

## Exploitation Steps

1. Navigate to a product page and click **Check stock**.
2. Intercept the request and send it to **Burp Repeater**.
3. Identify the vulnerable parameter:

   ```
   stockApi=<url>
   ```
4. Attempt a basic SSRF payload:

   ```
   http://127.0.0.1/
   ```

   → Request is blocked.
5. Bypass IP filtering using an alternative representation:

   ```
   http://127.1/
   ```
6. Attempt to access the admin panel:

   ```
   http://127.1/admin
   ```

   → Blocked due to path filtering.
7. Bypass path filtering via double URL encoding:

   ```
   http://127.1/%2561dmin
   ```

   (`%2561` → `%61` → `a`)
8. Access the admin interface and identify the delete endpoint:

   ```
   /admin/delete?username=carlos
   ```
9. Deliver the final payload:

   ```
   http://127.1/%2561dmin/delete?username=carlos
   ```
10. Send the request to delete the target user.

## Impact

**Severity:** High (Internal Access + Privilege Abuse)

Attackers can:

* Bypass SSRF protections.
* Access internal-only services.
* Perform administrative actions.
* Potentially pivot deeper into internal infrastructure.

## Mitigation

* Replace blacklist filtering with **strict allowlist validation**.
* Normalize and fully decode input before validation.
* Block access to internal IP ranges and loopback addresses.
* Use URL parsing libraries to prevent obfuscation bypasses.
* Enforce network-level restrictions (e.g., deny outbound requests to internal services).

## Key Takeaway

Blacklist-based defenses are ineffective against SSRF. Attackers can easily bypass them using encoding tricks and alternative representations. Robust SSRF protection requires strict allowlisting and proper input normalization.
