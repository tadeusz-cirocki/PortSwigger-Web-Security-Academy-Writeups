# 2FA Simple Bypass via Missing Authorization Check

## Summary

The application implements two-factor authentication (2FA) during login, but the protection is flawed due to missing authorization checks on post-authentication endpoints. By directly navigating to the account page after submitting valid credentials (without completing the 2FA step), an attacker can bypass the verification process and gain unauthorized access.

## Root Cause

The application fails to properly enforce the 2FA state before granting access to authenticated resources. After the primary authentication step (username and password), the server does not verify whether the user has successfully completed the 2FA challenge before allowing access to `/my-account`.

This results in a logic flaw where 2FA is implemented at the UI flow level rather than being strictly enforced server-side.

## Exploitation Steps

1. Log in using a valid account (e.g., `wiener:peter`) to observe the authentication flow.
2. Note the URL of the authenticated account page (`/my-account`).
3. Log out.
4. Log in using the victim's credentials (`carlos:montoya`).
5. When prompted for the 2FA verification code, do **not** complete the challenge.
6. Manually modify the browser URL and navigate directly to:

   ```
   /my-account
   ```
7. The application grants access to the account page without verifying the 2FA code.

## Impact

**Severity:** High (Authentication Bypass / Account Takeover)

Attackers with valid primary credentials can bypass the second authentication factor and fully compromise user accounts.

## Mitigation

* Enforce 2FA server-side by validating a completed 2FA state before granting access to protected endpoints.
* Use session flags (e.g., `2fa_verified=true`) and validate them on every sensitive request.
* Restrict direct access to authenticated endpoints until the full authentication flow is completed.
* Implement centralized authorization middleware to enforce authentication state consistently.

## Key Takeaway

Two-factor authentication must be enforced at the server authorization layer, not only in the application flow. Logic flaws in authentication state validation can completely undermine otherwise strong security controls.
