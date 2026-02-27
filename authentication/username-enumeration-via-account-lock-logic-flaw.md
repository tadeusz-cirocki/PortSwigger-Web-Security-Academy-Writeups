# Username Enumeration via Account Lock Logic Flaw

## Summary

The application implements account lockout protection after multiple failed login attempts. However, the implementation contains a logic flaw that enables attackers to enumerate valid usernames based on differences in server responses during the lockout process. After identifying a valid username, the attacker can brute-force the password and access the account.

## Root Cause

The authentication mechanism returns inconsistent responses depending on whether the supplied username exists. When repeated login attempts are made:

* **Valid usernames** trigger an account lock message after several failed attempts.
* **Invalid usernames** continue returning a generic authentication error.

This discrepancy allows attackers to distinguish valid accounts through response analysis.

## Exploitation Steps

1. Intercept the **POST /login** request using Burp Suite.
2. Send the request to **Intruder** and configure a **Cluster Bomb** attack:

   * Add a payload position to the `username` parameter.
   * Add a second payload position at the end of the request body.
3. Load:

   * Candidate usernames as the first payload set.
   * Null payloads (e.g., 5 repetitions) as the second payload set to trigger lockout behavior.
4. Start the attack and analyze responses:

   * Identify the username that returns a different response length or the message:
     *“You have made too many incorrect login attempts.”*
5. Note the valid username.
6. Create a new **Sniper** attack:

   * Fix the identified username.
   * Add a payload position to the `password` parameter.
7. Load the candidate password list and configure **Grep Extract** for error messages.
8. Start the attack and identify the response that does **not** contain an error message.
9. Wait for the lockout timer to reset.
10. Log in using the discovered credentials and access the user account page.

## Impact

**Severity:** High (Account Takeover)

Attackers can enumerate valid usernames and perform targeted password brute-force attacks, significantly increasing the likelihood of credential compromise.

## Mitigation

* Return identical error messages for both valid and invalid usernames.
* Apply account lockout logic uniformly without revealing account state.
* Implement rate limiting per IP and per account.
* Use CAPTCHA or adaptive authentication after repeated failures.
* Monitor authentication anomalies and brute-force patterns.

## Key Takeaway

Authentication mechanisms often leak sensitive information through subtle response differences. Even when protective controls such as account lockout exist, inconsistent error handling can enable username enumeration and facilitate account takeover attacks.
