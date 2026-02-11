# Username enumeration via subtly different responses

## Summary

The application is vulnerable to username enumeration due to a subtle
discrepancy in the login error message. Although the visible response
appears identical ("Invalid username or password."), one valid username
triggers a slightly different response containing a trailing space
instead of a period. This allows an attacker to distinguish valid
usernames and subsequently brute-force the password.

---

## Root Cause

The backend login logic generates slightly different error messages
depending on whether the username exists. While the application attempts
to standardize responses, an inconsistency in formatting (a trailing
space instead of a period) reveals whether a username is valid.

---

## Exploitation Steps

1.  Intercept a POST /login request in Burp.
2.  Send the request to Burp Intruder and place the payload position on
    the username parameter.
3.  Use the provided candidate username wordlist.
4.  In Intruder → Settings → Grep - Extract, highlight the full error
    message text "Invalid username or password." to extract it from
    responses.
5.  Run the attack and sort results by the extracted error message
    column.
6.  Identify the response that differs subtly (trailing space instead of
    period) --- this reveals a valid username.
7.  Replace the username with the identified valid username.
8.  Place the payload position on the password parameter.
9.  Use the candidate password wordlist and run the attack.
10. Identify the request that returns HTTP 302 (redirect) --- this
    reveals the correct password.
11. Log in with the valid credentials to solve the lab.

---

## Impact

An attacker can enumerate valid usernames and brute-force passwords.
This significantly lowers the complexity of authentication attacks and
can lead to full account compromise.

---

## Mitigation

-   Ensure authentication responses are completely identical for both
    valid and invalid usernames.
-   Normalize all error messages at the final response stage.
-   Implement account lockout or rate limiting.
-   Add CAPTCHA or progressive delays for repeated login attempts.

---

## Key Takeaway

Even a single extra whitespace character in an authentication response
can enable full username enumeration.
