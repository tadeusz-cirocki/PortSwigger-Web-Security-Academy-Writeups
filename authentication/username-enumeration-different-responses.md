# Username Enumeration via Different Responses

## Summary

The application discloses whether a username exists by returning different error messages and response lengths during authentication. This allows an attacker to enumerate valid usernames and subsequently brute-force the corresponding password.

## Root Cause

The login endpoint returns different responses depending on whether the username exists:

- Non-existent usernames trigger an "Invalid username" message.

- Valid usernames with incorrect passwords trigger an "Incorrect password" message.

These differences are reflected both in the response content and in the response length, enabling reliable username enumeration.

## Exploitation Steps

1. Intercept a login request and send it to Burp Intruder.

2. Configure a Sniper attack on the username parameter while keeping the password static.

3. Load the provided candidate username list and start the attack.

4. Identify the valid username by observing:

    - A different error message (Incorrect password instead of Invalid username)

    - A distinct response length compared to other requests

5. Configure a second Sniper attack, this time targeting the password parameter while using the enumerated valid username.

6. Load the provided password list and start the attack.

7. Identify the correct password by observing a 302 redirect, indicating a successful login.

8. Log in using the valid credentials and access the account page to solve the lab.

## Impact

An attacker can enumerate valid usernames and brute-force passwords, leading to unauthorized account access and potential compromise of user data.

## Mitigation

- Use a single, generic error message for all authentication failures.

- Ensure responses have uniform status codes, content, and timing.

- Implement account lockout or rate-limiting mechanisms on login attempts.

## Key Takeaway

Authentication responses must be consistent, as even subtle differences in error messages or response length can enable account enumeration attacks.