# 2FA Broken Logic via User Parameter Manipulation

## Summary

The application implements two-factor authentication (2FA), but the verification process contains a logic flaw that allows attackers to brute-force another user’s 2FA code. The vulnerability arises from improper trust in a client-controlled parameter that determines which account the verification applies to. By manipulating this parameter, an attacker can generate and brute-force a valid 2FA code for a victim account and gain unauthorized access.

## Root Cause

During the 2FA process, the application uses a client-supplied parameter (`verify`) to identify the account for which the verification code should be validated. This parameter is not bound to the authenticated session and can be modified by the attacker.

As a result:

* The attacker can request a 2FA challenge for another user.
* The attacker can brute-force the victim’s 2FA code without possessing their primary authentication session.

This represents a broken authentication logic flaw caused by missing server-side binding between session state and verification context.

## Exploitation Steps

1. Log in using a valid account (`wiener:peter`) while intercepting traffic with Burp Suite.
2. Observe the **POST /login2** request and note the `verify` parameter used to determine the account for 2FA validation.
3. Log out.
4. Send the **GET /login2** request to Burp Repeater and modify:

   ```
   verify=carlos
   ```

   This triggers generation of a 2FA code for the victim account.
5. Start a new login attempt using the attacker account (`wiener:peter`) and submit any invalid 2FA code.
6. Send the resulting **POST /login2** request to Burp Intruder.
7. Configure the attack:

   * Set `verify=carlos`.
   * Add a payload position to the `mfa-code` parameter.
   * Load a numeric brute-force payload list (e.g., 0000–9999).
8. Execute the attack and identify the response returning **HTTP 302** (successful verification).
9. Load the successful response in the browser and access the account page to complete the lab.

## Impact

**Severity:** High (Account Takeover)

Attackers can bypass the intended user-to-session binding in the 2FA process and brute-force verification codes for arbitrary accounts, leading to full account compromise.

## Mitigation

* Bind the 2FA verification process strictly to the authenticated session.
* Do not rely on client-controlled parameters to identify the user during verification.
* Regenerate and validate verification context server-side.
* Implement rate limiting and attempt counters for 2FA submissions.
* Invalidate verification challenges after a small number of failed attempts.

## Key Takeaway

Two-factor authentication mechanisms must securely bind verification steps to the authenticated session. Trusting client-controlled parameters in authentication workflows can completely break otherwise strong security controls.
