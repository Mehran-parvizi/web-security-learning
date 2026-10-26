# Broken Access Control: Unprotected Administrative Functionality

## Training Environment

- Platform: PortSwigger Web Security Academy
- Lab: Unprotected Admin Functionality
- Difficulty: Apprentice
- Vulnerability: Broken Access Control
- Category: Vertical Privilege Escalation
- Status: Solved

**Lab URL:** https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality

> This writeup documents an intentionally vulnerable training application, not a real-world bug bounty finding.

---

## 1. Summary

The application exposes an administrative interface without properly enforcing access control.

The administrative endpoint was discovered through the publicly accessible `/robots.txt` file.

By accessing the exposed administrative interface, an unauthorized user could perform administrative operations, including deleting another user's account.

During testing, the account belonging to `carlos` was successfully deleted.

---

## 2. Steps to Reproduce

### Step 1: Inspect robots.txt

Send the following request:

```http
GET /robots.txt HTTP/1.1
```

The application returns:

```text
User-agent: *
Disallow: /administrator-panel
```

This reveals the location of an administrative interface.

### Step 2: Access the Administrative Interface

Navigate to:

```text
/administrator-panel
```

The application exposes the administrative interface without requiring appropriate administrative authorization.

### Step 3: Execute an Administrative Action

Send the following request:

```http
GET /administrator-panel/delete?username=carlos HTTP/2
```

The server responds:

```http
HTTP/2 302 Found
Location: /administrator-panel
Content-Length: 0
```

### Step 4: Verify the Result

The account belonging to `carlos` was deleted.

The PortSwigger lab also displayed the `Lab Solved` confirmation.

---

## 3. Expected Behavior

Administrative functionality should only be accessible to users with appropriate privileges.

Unauthorized users should not be able to access administrative interfaces or execute administrative operations.

Authorization checks must be enforced on the server side for every privileged action.

---

## 4. Observed Behavior

The administrative interface was accessible without appropriate authorization.

The application also accepted an unauthorized request to delete the user `carlos`.

The deletion was confirmed during testing.

---

## 5. Security Impact

### Confirmed Impact

An unauthorized user was able to access administrative functionality and delete another user's account.

### Potential Impact

Other administrative actions might also be exposed if they are protected by the same insufficient access controls.

However, no additional administrative operations were tested.

---

## 6. Root Cause

The application does not adequately enforce server-side authorization checks for administrative functionality.

The `/robots.txt` file revealed the administrative path, but the information disclosure through this file was not the primary security vulnerability.

The underlying issue was the absence of effective authorization controls on administrative endpoints.

---

## 7. Remediation

1. Implement server-side role-based authorization checks for administrative endpoints.
2. Restrict administrative actions to users with explicitly assigned privileges.
3. Apply authorization checks to every sensitive administrative operation.
4. Follow the principle of least privilege.
5. Add automated tests to ensure unauthorized users cannot access or execute administrative functionality.
6. Avoid relying on hidden URLs or `robots.txt` directives as security controls.

---

## 8. Tools Used

- Web Browser
- Burp Suite
- PortSwigger Web Security Academy

---

## 9. Key Takeaways

- Hidden administrative URLs do not provide effective access control.
- The `robots.txt` file is not a security mechanism.
- Server-side authorization is necessary for privileged operations.
- HTTP `302 Found` alone does not prove that an operation succeeded.
- Security findings should be supported by verified application behavior.

---

## 10. References

- [PortSwigger: Access Control](https://portswigger.net/web-security/access-control)
- [OWASP: Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
