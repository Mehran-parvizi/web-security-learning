# Username Enumeration via Subtle Differences in Login Error Messages

## Training Environment

- **Platform:** PortSwigger Web Security Academy
- **Lab:** Username Enumeration via Subtly Different Responses
- **Difficulty:** Practitioner
- **Vulnerability:** Username Enumeration
- **Category:** Authentication Vulnerabilities
- **Status:** Solved

**Lab URL:** https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-subtly-different-responses

> This report documents an intentionally vulnerable training environment. It is not a real-world bug bounty finding.

---

## 1. Summary

The `/login` endpoint is vulnerable to username enumeration because the application returns slightly different error messages depending on whether the submitted username exists.

During testing, the difference was identified through the presence or absence of a trailing period (`.`) in the login error message.

Using Burp Suite Intruder and the username wordlist provided by PortSwigger, the valid username `ag` was identified.

Subsequent password testing in the authorized training environment resulted in successful authentication.

---

## 2. Steps to Reproduce

### Step 1: Capture the Login Request

Navigate to the login page and intercept a login request using Burp Suite.

Example request:

```http
POST /login HTTP/2
Content-Type: application/x-www-form-urlencoded

username=testuser123&password=123456
```

### Step 2: Configure Burp Intruder

Send the login request to Burp Intruder.

Select the `username` parameter as the payload position:

```text
username=§testuser123§&password=123456
```

Keep the password parameter unchanged during username enumeration.

Load the username wordlist provided by the training lab.

### Step 3: Identify the Response Difference

Use Burp Intruder's response analysis features to compare login error messages.

During testing, most invalid usernames produced:

```text
Invalid username or password.
```

However, the username `ag` produced:

```text
Invalid username or password
```

The difference is the missing trailing period.

This response discrepancy provided evidence that the application processed the username `ag` differently.

### Step 4: Identify the Valid Username

The username `ag` was identified as a valid account based on the observed response difference.

The successful authentication performed later in the lab confirmed that this username existed.

### Step 5: Test Passwords in the Training Environment

Keep the identified username constant and configure the password field as the Intruder payload position:

```text
username=ag&password=§123456§
```

Use the password wordlist supplied by PortSwigger.

The correct lab password was identified as:

```text
biteme
```

### Step 6: Verify Successful Authentication

Submit the identified credentials through the login form.

The application authenticates the user and provides access to the account.

The PortSwigger lab displays the `Lab Solved` confirmation.

---

## 3. Expected Behavior

The application should not reveal whether a username exists through differences in authentication error messages.

Invalid usernames and incorrect passwords should produce consistent, generic responses.

For example:

```text
Invalid username or password.
```

Appropriate protections against automated authentication attempts should also be implemented.

---

## 4. Observed Behavior

The application returned distinguishable login error messages.

| Username Condition | Error Message |
|--------------------|---------------|
| Invalid username | `Invalid username or password.` |
| Valid username (`ag`) with incorrect password | `Invalid username or password` |

The difference in punctuation allowed the valid username to be identified.

Subsequent password testing using the lab-provided wordlist resulted in successful authentication.

---

## 5. Security Impact

### Confirmed Impact

An unauthenticated user can distinguish valid usernames from invalid ones by comparing login error messages.

Testing identified the valid username `ag`.

In the authorized training environment, additional password testing resulted in successful authentication to the identified account.

### Potential Impact

Username enumeration can make targeted password guessing, credential stuffing, and phishing attempts more effective.

However, username enumeration alone does not provide access to an account.

Successful authentication requires valid credentials or another independently exploitable authentication weakness.

---

## 6. Root Cause

The application exposes different authentication error messages depending on whether a submitted username exists.

Even a single-character difference can reveal information about the application's internal account validation process.

The underlying issue is inconsistent observable behavior during authentication failures.

---

## 7. Remediation

1. Return consistent, generic authentication error messages for invalid usernames and incorrect passwords.
2. Ensure login failure responses do not reveal account existence through response content or other observable differences.
3. Implement appropriate rate limiting to reduce automated username enumeration and password guessing attempts.
4. Monitor repeated failed authentication attempts.
5. Review the login flow for additional enumeration signals, including response timing, status codes, and redirects.

---

## 8. Tools Used

- Burp Suite Community Edition
- Burp Intruder
- Web Browser
- PortSwigger Web Security Academy

---

## 9. Key Takeaways

- Small differences in authentication responses can reveal valid usernames.
- A single missing punctuation character can provide a useful testing signal.
- Burp Intruder can help identify subtle response differences.
- Suspicious responses should be reproduced before drawing security conclusions.
- Username enumeration and successful authentication are separate security outcomes.
- HTTP response behavior should be analyzed using multiple signals rather than status codes alone.

---

## 10. References

- [PortSwigger: Authentication Vulnerabilities](https://portswigger.net/web-security/authentication)
- [OWASP: Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
