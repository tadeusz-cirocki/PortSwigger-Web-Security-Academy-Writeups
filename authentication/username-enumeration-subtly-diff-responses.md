# Username enumeration via subtly different responses

## Signal Used
Subtle difference in error message content – identical text visually, but one response contained a trailing whitespace instead of a full stop.

## Tooling
Burp Intruder with Grep-Extract.

## Exploitation Flow
- Sent POST /login request to Intruder with username as payload position.
- Used Grep-Extract on the error message "Invalid username or password".
- Identified valid username based on a subtle variation (trailing space).
- Replaced payload position with password parameter.
- Brute-forced password and identified correct one via 302 redirect.

## Result
Successful authentication and access to the user account page.

## One-Line Takeaway
Even visually identical error messages can leak credentials through subtle content differences like whitespace.
