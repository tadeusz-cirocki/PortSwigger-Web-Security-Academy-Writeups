# Password Reset Poisoning via Middleware Header Injection

## Summary

The application’s password reset functionality is vulnerable to **password reset poisoning** due to improper trust in proxy-related HTTP headers. By injecting a malicious `X-Forwarded-Host` header, an attacker can manipulate the domain used in the password reset link sent via email. This allows interception of the victim’s reset token and ultimately leads to full account takeover.

## Root Cause

The application dynamically constructs password reset URLs based on the `Host` or `X-Forwarded-Host` header without proper validation.

Because:

* The `X-Forwarded-Host` header is trusted,
* The value is reflected into the reset email link,
* No strict server-side validation or canonical host enforcement is implemented,

an attacker can poison the reset link and redirect the victim’s token to an attacker-controlled domain.

This is a classic example of host header injection leading to authentication compromise.

## Exploitation Steps

1. Initiate a password reset for a valid account and observe the reset email structure.
2. Intercept the **POST /forgot-password** request using Burp Suite.
3. Send the request to **Burp Repeater**.
4. Add the following header:

   ```
   X-Forwarded-Host: ATTACKER-SERVER
   ```
5. Modify the `username` parameter to:

   ```
   carlos
   ```
6. Send the request.
7. Wait for the victim to click the poisoned reset link.
8. On the attacker-controlled server, observe the incoming request containing:

   ```
   temp-forgot-password-token=<victim_token>
   ```
9. Copy the captured reset token.
10. Use the legitimate reset URL and replace the token value with the stolen one.
11. Set a new password for Carlos.
12. Log in using the updated credentials.

## Impact

**Severity:** Critical (Account Takeover)

Attackers can:

* Intercept password reset tokens.
* Reset arbitrary user passwords.
* Fully compromise user accounts without email access.

This vulnerability is especially dangerous because it exploits normal user behavior (clicking reset links).

## Mitigation

* Do not trust `X-Forwarded-Host` or similar headers unless validated and set by trusted infrastructure.
* Use a hardcoded, canonical application base URL when generating password reset links.
* Validate the `Host` header against an allowlist.
* Ensure reverse proxies strip untrusted forwarding headers.
* Implement monitoring for anomalous password reset requests.

## Key Takeaway

Authentication flows must never rely on untrusted HTTP headers for security-critical URL generation. Middleware misconfiguration and host header trust issues can escalate into full account takeover.
