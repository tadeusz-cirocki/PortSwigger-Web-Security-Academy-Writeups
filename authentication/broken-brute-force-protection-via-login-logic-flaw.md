# Broken brute-force protection via login logic flaw

## Summary
The application implements IP-based brute-force protection but contains a logic flaw that allows resetting the failed login counter by performing a successful login to another account.

## Root Cause
The failed login counter is tied to session/login state rather than strictly enforced per account or per IP.

## Exploitation Steps
1. Send brute-force attempts for victim user `carlos`.
2. Before reaching the block threshold, log in using a valid account (`wiener`).
3. This resets the failed login counter.
4. Continue brute-forcing until a valid password is found.

## Impact
Severity: High (Account Takeover)

Attackers can bypass brute-force protection and obtain valid credentials.

## Mitigation
- Track failed attempts per account.
- Do not reset counters on successful login for different users.
- Implement proper rate limiting and anomaly detection.

## Key Takeaway
Authentication protections often fail due to logic flaws rather than missing controls.