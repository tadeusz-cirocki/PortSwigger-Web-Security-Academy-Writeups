# Password Reset Broken Logic via Missing Token Validation

## Summary

The application implements a password reset mechanism that uses a token delivered via email. However, due to flawed server-side logic, the token is not properly validated during the password reset process. As a result, attackers can reset arbitrary users’ passwords by modifying the `username` parameter, without possessing a valid reset token.

## Root Cause

The password reset workflow includes a reset token (`temp-forgot-password-token`) that is intended to authorize the password change request. However, the application fails to validate this token when the new password is submitted.

Instead, the server relies solely on a client-controlled parameter (`username`), allowing attackers to modify it and reset passwords for other users.

This represents a logic flaw caused by missing authorization checks in the password reset flow.

## Exploitation Steps

1. Navigate to the **Forgot your password?** functionality and submit a reset request for a valid account (`wiener`).
2. Open the email client and follow the password reset link.
3. Intercept the **POST /forgot-password** request when submitting the new password.
4. Observe that:

   * The reset token is present in the request.
   * The `username` is included as a hidden form parameter.
5. Send the request to Burp Repeater.
6. Remove the value of the `temp-forgot-password-token` parameter from both:

   * URL query string
   * Request body
7. Confirm that the password reset still succeeds, indicating the token is not validated.
8. Modify the request:

   ```
   username=carlos
   ```
9. Set a new password and send the request.
10. Log in using the updated credentials for `carlos` and access the account page.

## Impact

**Severity:** Critical (Account Takeover)

Attackers can reset passwords for arbitrary users without access to their email accounts or valid reset tokens, leading to full account compromise.

## Mitigation

* Enforce strict server-side validation of password reset tokens.
* Bind reset tokens to:

  * a specific user,
  * a short expiration time,
  * a single use.
* Do not trust client-controlled parameters such as hidden form fields for authorization decisions.
* Invalidate tokens immediately after successful password reset.
* Log and monitor suspicious password reset activity.

## Key Takeaway

Password reset mechanisms are part of the authentication boundary and must be treated with the same security rigor as login flows. Missing validation of reset tokens can completely undermine account security.
