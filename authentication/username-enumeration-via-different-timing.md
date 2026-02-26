# Username enumeration via response timing

## Summary
The login functionality is vulnerable to username enumeration based on response timing discrepancies. Additionally, the application implements IP-based brute-force protection that can be bypassed by spoofing the X-Forwarded-For header. By identifying a valid username through timing analysis and bypassing rate limiting, it is possible to brute-force the password and gain account access.

---

## Root Cause
The application performs additional processing when a valid username is supplied (e.g., password verification logic), which results in longer response times compared to invalid usernames. This timing difference allows attackers to distinguish valid usernames.

Furthermore, the application relies on the client-controlled `X-Forwarded-For` header for IP-based rate limiting without verifying that it originates from a trusted reverse proxy. This enables bypassing brute-force protection.

---

## Exploitation Steps
1. Intercept a POST `/login` request using Burp Suite.
2. Send the request to Repeater and observe that repeated failed attempts result in IP blocking.
3. Add the `X-Forwarded-For` header manually and change its value between requests to bypass IP-based rate limiting.
4. Set a very long password value (e.g., ~100 characters) to amplify timing differences.
5. Send the request to Burp Intruder.
6. Select **Pitchfork** attack type.
7. Add payload positions for:
   - `X-Forwarded-For`
   - `username`
8. Configure:
   - Payload 1 (IP spoofing): Numbers (1–100)
   - Payload 2: Candidate usernames list
9. Enable the **Response received** and **Response completed** columns.
10. Identify the username with significantly higher response time.
11. Repeat the request to confirm consistent timing difference.
12. Create a new Intruder attack:
   - Keep `X-Forwarded-For` payload.
   - Set the identified valid username.
   - Add password payload position.
13. Use candidate password list.
14. Identify the response returning HTTP 302.
15. Log in with discovered credentials and access the account page.

---

## Impact
Severity: High (Account Compromise via Timing Enumeration)

An attacker can enumerate valid usernames through response timing analysis and bypass IP-based brute-force protection. This enables targeted password brute-forcing, ultimately leading to full account takeover.

---

## Mitigation
- Ensure constant-time authentication responses for both valid and invalid usernames.
- Avoid exposing timing differences in password verification logic.
- Do not rely on client-controlled headers (e.g., `X-Forwarded-For`) for security decisions.
- Implement rate limiting based on trusted network-layer IP information.
- Add account lockout mechanisms or multi-factor authentication.

---

## Key Takeaway
Authentication mechanisms must use constant-time comparisons and must not rely on client-controlled headers for rate limiting, as both can lead to account compromise.
