# Password Brute-Force via Password Change Logic Flaw

## Summary

The application’s password change functionality is vulnerable to brute-force attacks due to inconsistent validation logic and distinguishable error messages. By manipulating request parameters and analyzing server responses, an attacker can enumerate the correct current password for another user without triggering account lockout mechanisms.

## Root Cause

The password change endpoint contains multiple logic flaws:

* The `username` parameter is **client-controlled** and not bound to the authenticated session.
* The application returns **different error messages** depending on validation outcomes:

  * Incorrect current password → *“Current password is incorrect”*
  * Correct current password but mismatched new passwords → *“New passwords do not match”*
* Account lockout is only triggered when **new passwords match**, allowing attackers to bypass lockout protections by intentionally submitting mismatched new passwords.

This creates a side-channel that enables attackers to identify valid passwords based on response differences.

## Exploitation Steps

1. Log in using valid credentials (`wiener:peter`).
2. Navigate to the password change functionality and observe request structure:

   ```
   username=<user>&current-password=<value>&new-password-1=<value>&new-password-2=<value>
   ```
3. Test different scenarios to understand response behavior:

   * Incorrect current password → error message A
   * Correct current password + mismatched new passwords → error message B
4. Send the **POST /my-account/change-password** request to Burp Intruder.
5. Modify the request:

   ```
   username=carlos
   current-password=§payload§
   new-password-1=123
   new-password-2=abc
   ```
6. Load the candidate password list as payloads.
7. Configure a **Grep Match** rule for:

   ```
   New passwords do not match
   ```
8. Launch the attack and analyze responses.
9. Identify the payload that triggers the *“New passwords do not match”* message — this indicates a **correct current password**.
10. Log out and authenticate as `carlos` using the discovered password.
11. Access the account page to complete the lab.

## Impact

**Severity:** High (Account Takeover)

Attackers can:

* Enumerate valid passwords without triggering lockouts.
* Bypass brute-force protections.
* Fully compromise user accounts.

## Mitigation

* Bind the `username` parameter to the authenticated session; do not trust client input.
* Return consistent error messages for all password validation failures.
* Apply account lockout and rate limiting regardless of input combinations.
* Validate current password before processing any new password logic.
* Implement centralized authentication and validation controls.

## Key Takeaway

Authentication-related features must avoid leaking information through response differences. Even subtle discrepancies in error messages can enable efficient brute-force attacks and completely bypass security controls.
