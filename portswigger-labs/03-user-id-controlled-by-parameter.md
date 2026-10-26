# IDOR: Unauthorized Access to Another User's Account Information

## Training Environment

- Platform: PortSwigger Web Security Academy
- Lab: User ID Controlled by Request Parameter
- Difficulty: Apprentice
- Vulnerability: Insecure Direct Object Reference (IDOR)
- Category: Broken Access Control / Horizontal Privilege Escalation
- Status: Solved

**Lab URL:**
https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter

> This report documents a vulnerability reproduced in an intentionally vulnerable training environment. It is not a real-world bug bounty finding.

---

## 1. Summary

The `/my-account` endpoint is vulnerable to Insecure Direct Object Reference (IDOR).

The application allows an authenticated user to modify the `id` request parameter and retrieve another user's private account information without proper object-level authorization.

During testing, the authenticated user `wiener` was able to access account information belonging to `carlos`, including his private API key.

The vulnerability results from insufficient server-side authorization checks on the requested account.

---

## 2. Steps to Reproduce

### Step 1: Authenticate

Log in to the application using the provided training account:

- Username: `wiener`
- Password: `[LAB-PROVIDED PASSWORD]`

Navigate to the account page.

### Step 2: Capture the Account Request

Using Burp Suite, capture the request to the `/my-account` endpoint.

The original request contains the authenticated user's identifier:

```http
GET /my-account?id=wiener HTTP/2
Cookie: session=[REDACTED]
```

### Step 3: Modify the User Identifier

Send the request to Burp Repeater.

Change the `id` parameter from `wiener` to `carlos`.

Keep the original authenticated session cookie unchanged.

```http
GET /my-account?id=carlos HTTP/2
Cookie: session=[SAME_SESSION]
```

### Step 4: Examine the Response

Send the modified request and inspect the server's response.

The application returns account information belonging to `carlos`, including his private API key.

The request remains authenticated as `wiener`.

---

## 3. Expected Behavior

The server should verify whether the authenticated user is authorized to access the requested account.

If `wiener` attempts to access `carlos`'s account without permission, the application should deny access to the protected information.

Authorization must be enforced on the server side for each requested resource.

---

## 4. Observed Behavior

The application accepts the modified `id` parameter and returns another user's private account information.

The application fails to enforce object-level authorization on the `/my-account` endpoint, allowing an authenticated user to retrieve account information belonging to another user.

The returned information includes the target user's private API key.

**Evidence:**

- Authenticated user: `wiener`
- Requested account: `carlos`
- Session cookie: Unchanged
- Result: Unauthorized disclosure of `carlos`'s account information and private API key

The actual API key is intentionally excluded from this public report.

---

## 5. Security Impact

### Confirmed Impact

An authenticated user can access another user's private account information by modifying the `id` request parameter.

Testing confirmed unauthorized disclosure of another user's private API key.

### Potential Impact

Depending on the permissions associated with the exposed API key, additional unauthorized actions might be possible.

However, the API key's permissions and potential misuse were not tested.

Therefore, no additional account access or API-level compromise is claimed.

---

## 6. Root Cause

The application relies on a user-controlled request parameter to identify the account being accessed without adequately verifying whether the authenticated user has permission to access that specific account.

Authentication confirms the identity of the requesting user, but authentication alone does not establish authorization to access another user's resources.

---

## 7. Remediation

1. Enforce server-side object-level authorization checks on every request that accesses user-specific resources.

2. Where appropriate, derive the account identity directly from the authenticated session instead of relying on a client-controlled identifier.

3. If account identifiers must be accepted, verify that the authenticated user has explicit permission to access the requested account.

4. Apply authorization checks consistently across all endpoints that expose private user data.

5. Implement automated authorization tests to verify that authenticated users cannot access resources belonging to other users without permission.

---

## 8. Tools Used

- Burp Suite Community Edition
- Burp Repeater
- Web Browser
- PortSwigger Web Security Academy

---

## 9. Key Takeaways

This lab demonstrated that:

- Authentication and authorization are separate security controls.
- A valid session does not automatically grant access to every user resource.
- User-controlled identifiers must not be trusted as proof of authorization.
- Object-level access control must be enforced on the server.
- A successful HTTP response must be examined to verify what information was actually disclosed.

---

## 10. References

- [PortSwigger: Access Control Vulnerabilities](https://portswigger.net/web-security/access-control)
- [OWASP: Insecure Direct Object Reference Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html)
