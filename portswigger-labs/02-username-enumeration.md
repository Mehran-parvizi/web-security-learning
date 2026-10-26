# Username Enumeration via Different Login Responses

## Training Environment

- Platform: PortSwigger Web Security Academy
- Lab: Username Enumeration via Different Responses
- Difficulty: Apprentice
- Vulnerability: Username Enumeration
- Category: Authentication Vulnerabilities
- Status: Solved

**Lab URL:**
https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-different-responses

> This report documents an intentionally vulnerable training application. It is not a real-world bug bounty finding.

---

## 1. Summary

The application's login endpoint reveals whether a username exists through different authentication error messages.

During testing, invalid usernames produced the message `Invalid username`, while a valid username with an incorrect password produced `Incorrect password`.

This difference allowed the identification of a valid account without knowing its password.

After identifying the valid username, the lab's provided password wordlist was used to test passwords in the authorized training environment. This resulted in successful authentication.

---

## 2. Steps to Reproduce

### Step 1: Capture the Login Request

Navigate to the login page and capture the authentication request using Burp Suite.

Example:

```http
POST /login HTTP/2
Content-Type: application/x-www-form-urlencoded

username=testuser123&password=123456
```

### Step 2: Test Invalid Usernames

Submit login requests using different invalid usernames while keeping the password unchanged.

Examples:

```text
testuser123
administrator
adm
info
```

The application responds with:

```text
Invalid username
```

The HTTP status code is `200 OK`.

### Step 3: Configure Burp Intruder

Send the login request to Burp Intruder.

Set the username as the payload position:

```text
username=§testuser123§&password=123456
```

Use the username wordlist provided by PortSwigger.

### Step 4: Identify a Different Response

After testing the supplied usernames, one account produces a different authentication error.

Username:

```text
affiliates
```

Response:

```text
Incorrect password
```

Other invalid usernames continue to produce:

```text
Invalid username
```

This indicates that `affiliates` is a valid username.

### Step 5: Confirm the Finding

Send the request to Burp Repeater.

Test the identified username with another incorrect password.

The application continues to return:

```text
Incorrect password
```

This confirms the response difference.

### Step 6: Validate Authentication in the Training Lab

Keep the identified username constant and configure the password as the Intruder payload position:

```text
username=affiliates&password=§123456§
```

Use the password wordlist supplied by the training lab.

A successful authentication attempt produces a response similar to:

```http
HTTP/2 302 Found
Location: /my-account?id=affiliates
Set-Cookie: session=[REDACTED]; Secure; HttpOnly; SameSite=None
```

Following the authenticated flow confirms access to the `affiliates` account.

The lab displays the `Lab Solved` confirmation.

---

## 3. Expected Behavior

The application should avoid revealing whether a username exists through authentication error messages.

Invalid usernames and incorrect passwords should produce consistent, generic responses.

For example:

```text
Invalid username or password.
```

Authentication defenses should also include appropriate rate limiting and monitoring.

---

## 4. Observed Behavior

The application returns distinguishable error messages:

| Condition | Response Message |
|-----------|------------------|
| Invalid username | Invalid username |
| Valid username with incorrect password | Incorrect password |

Both responses use HTTP status code `200 OK`.

The difference in the response body allows valid usernames to be identified.

---

## 5. Security Impact

### Confirmed Impact

The application's authentication responses reveal whether a username exists.

Testing confirmed that the username `affiliates` was valid.

In the authorized training environment, subsequent password testing using the supplied wordlist resulted in successful authentication.

### Potential Impact

Username enumeration can assist attackers in targeting valid accounts for password guessing, credential stuffing, or phishing attempts.

However, username enumeration alone does not grant access to an account.

---

## 6. Root Cause

The login endpoint reveals internal account-validation results through different error messages.

The application distinguishes between nonexistent usernames and incorrect passwords in responses visible to unauthenticated users.

This creates an information disclosure weakness in the authentication process.

---

## 7. Remediation

1. Use consistent authentication error messages for invalid usernames and incorrect passwords.
2. Avoid revealing account existence through response content.
3. Implement appropriate rate limiting and authentication abuse detection.
4. Monitor repeated failed login attempts.
5. Review the authentication flow for other enumeration signals, including response timing and HTTP status differences.

---

## 8. Tools Used

- Burp Suite Community Edition
- Burp Intruder
- Burp Repeater
- PortSwigger Web Security Academy

---

## 9. Key Takeaways

- HTTP status codes alone do not determine whether authentication succeeded.
- Differences in response bodies can reveal valid usernames.
- Burp Intruder can automate controlled comparisons of authentication responses.
- Burp Repeater can help confirm unusual results.
- Username enumeration and successful account authentication are separate security outcomes.
- Findings should be confirmed before being reported.

---

## 10. References

- [PortSwigger: Authentication Vulnerabilities](https://portswigger.net/web-security/authentication)
- [OWASP: Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
