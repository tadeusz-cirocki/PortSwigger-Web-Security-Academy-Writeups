# Brute-Forcing a Stay-Logged-In Cookie

## Summary

The application provides a “stay logged in” feature using a persistent cookie. However, the cookie is constructed using a predictable format that includes a weak password-derived hash. Because the hash algorithm is unsalted and based on MD5, attackers can brute-force the cookie by iterating through candidate passwords and reconstructing valid authentication tokens for other users.

## Root Cause

The persistent authentication cookie is generated using the following structure:

```
base64(username + ":" + md5(password))
```

This implementation introduces multiple security flaws:

* The password hash uses **MD5**, which is cryptographically weak and fast to brute-force.
* The hash is **unsalted**, allowing direct comparison against candidate passwords.
* The cookie is fully **predictable**, enabling attackers to reconstruct valid authentication tokens for arbitrary users.

Additionally, the server trusts the cookie value without additional verification or session binding.

## Exploitation Steps

1. Log in using valid credentials (`wiener:peter`) with the **Stay logged in** option enabled.
2. Inspect the `stay-logged-in` cookie and decode it from Base64.
3. Observe the structure:

   ```
   wiener:<md5_hash>
   ```
4. Verify that the hash corresponds to the MD5 hash of the password.
5. Log out.
6. Capture the **GET /my-account** request and send it to Burp Intruder.
7. Configure payload processing rules:

   * Hash payloads using **MD5**.
   * Add prefix:

     ```
     carlos:
     ```
   * Base64-encode the final value.
8. Load the candidate password list as the payload set.
9. Add a **Grep Match** rule for a string that appears only in authenticated responses (e.g., `Update email`).
10. Execute the attack and identify the response indicating successful authentication.
11. Use the generated cookie to access Carlos’s account.

## Impact

**Severity:** High (Account Takeover)

Attackers can brute-force persistent authentication cookies and gain unauthorized access to user accounts without needing to perform the standard login flow.

## Mitigation

* Do not store password-derived hashes in authentication cookies.
* Replace custom persistent login logic with securely generated random tokens.
* Store persistent tokens server-side and bind them to user sessions.
* Use strong cryptographic primitives and avoid MD5.
* Implement token rotation and expiration.
* Monitor for abnormal authentication patterns.

## Key Takeaway

Persistent authentication mechanisms must use securely generated random tokens instead of predictable password-derived values. Weak cryptographic design in session-related cookies can directly lead to account takeover.
