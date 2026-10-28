# Username Enumeration via Response Timing

## Training Environment

- **Platform:** PortSwigger Web Security Academy
- **Lab:** Username Enumeration via Response Timing
- **Difficulty:** Practitioner
- **Category:** Authentication Vulnerabilities
- **Vulnerability:** Timing-Based Username Enumeration
- **Status:** Solved

**Lab URL:** https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-response-timing

> This report documents security testing performed in an intentionally vulnerable PortSwigger training environment. It is not a real-world bug bounty finding.

---

## 1. Summary

The `POST /login` endpoint is vulnerable to username enumeration through differences in HTTP response times.

During testing, login requests containing invalid usernames returned relatively quickly, even when long passwords were supplied.

In contrast, requests containing valid usernames and similarly long passwords produced noticeably longer response times.

The candidate username `arlington` produced elevated response times across repeated tests. Its validity was subsequently confirmed through successful authentication using the password `chelsea`.

A related weakness was also observed in the application's IP-based login restriction. Changing the `X-Forwarded-For` header allowed additional login attempts to proceed despite a previously triggered restriction.

---

## 2. Steps to Reproduce

### Step 1: Capture the Login Request

Navigate to the login page and capture the authentication request using Burp Suite.

The application uses the following endpoint:

```http
POST /login HTTP/2
Content-Type: application/x-www-form-urlencoded
```

### Step 2: Establish a Timing Baseline

Send the login request to Burp Repeater.

First, submit a request using an invalid username and a short, incorrect password.

Record the response time.

Repeat the request using the same invalid username and a password approximately 100 characters long.

During testing, the following results were observed:

| Username | Password Type | Response Time |
|----------|---------------|---------------|
| `not_a_real_user_8392` | Short | 184 ms |
| `not_a_real_user_8392` | Long | 155 ms |

The response times remained relatively low despite the increased password length.

### Step 3: Compare Against a Known-Valid Username

Submit a login request using the lab's known-valid username `wiener` and a similarly long, incorrect password.

Observed response time:

```text
839 ms
```

This difference suggested that the application performed different processing depending on whether the username existed.

### Step 4: Identify a Candidate Username

Use Burp Intruder to test candidate usernames with a long, incorrect password.

The following candidate produced noticeably longer response times:

```text
arlington
```

Recorded measurements:

| Test | Username | Response Time |
|------|----------|---------------|
| 1 | arlington | 1343 ms |
| 2 | arlington | 1464 ms |
| 3 | arlington | 2469 ms |

Although the exact response times varied, the recorded measurements were substantially higher than the invalid-username baseline.

The candidate was rechecked using additional requests.

These timing differences suggested that `arlington` was a valid username, but the timing measurements alone were not considered conclusive proof.

### Step 5: Observe the IP-Based Login Restriction

After multiple unsuccessful login attempts, the application returned:

```text
You have made too many incorrect login attempts.
Please try again in 30 minute(s).
```

The response used:

```http
HTTP/2 200 OK
```

This demonstrated that the HTTP status code alone did not indicate the application's authentication outcome.

During the authorized lab test, modifying the `X-Forwarded-For` header allowed additional requests to proceed despite the previously observed restriction.

Example header used during testing:

```http
X-Forwarded-For: 200
```

This behavior suggested that the application's IP-based restriction could be influenced by a client-controlled header.

### Step 6: Discover the Password

After identifying the candidate username, Burp Intruder was used to test the password candidates provided by PortSwigger.

The username remained fixed:

```text
username=arlington
```

The password parameter was configured as the payload position.

Where necessary, the `X-Forwarded-For` value was varied during the authorized lab test to continue testing despite the previously observed IP-based restriction.

The correct lab password was identified as:

```text
chelsea
```

### Step 7: Verify Successful Authentication

Log in using the identified lab credentials.

The application returned a redirect response:

```http
HTTP/2 302 Found
```

Successful authentication was confirmed by accessing the account page.

The PortSwigger lab displayed:

```text
Congratulations, you solved the lab!
```

The account page displayed the username:

```text
arlington
```

---

## 3. Expected Behavior

The application should not reveal whether usernames exist through measurable response-time differences.

Authentication processing should be designed to minimize externally observable timing discrepancies between valid and invalid usernames.

Login restrictions should also rely on trustworthy client identification rather than blindly trusting headers supplied by untrusted clients.

---

## 4. Observed Behavior

The application exhibited different response times depending on whether the submitted username was valid.

Long incorrect passwords caused noticeably longer response times for valid usernames, while invalid usernames returned comparatively quickly.

This timing discrepancy enabled the identification of a valid username.

The application's login restriction could also be influenced by changes to the `X-Forwarded-For` header during the lab test.

Successful authentication with the identified credentials confirmed that the candidate username existed.

---

## 5. Security Impact

### Confirmed Impact: Username Enumeration

The login endpoint disclosed information about username validity through response-time differences.

The valid username `arlington` was identified through timing analysis and subsequently confirmed through successful authentication.

### Related Weakness: IP-Based Rate-Limit Bypass

During testing, changing the `X-Forwarded-For` header allowed further login requests to proceed after an IP-based restriction had been triggered.

This behavior indicated that the restriction could be influenced by client-controlled request data.

### Potential Impact

Username enumeration can help attackers identify accounts for targeted password guessing, credential stuffing, or phishing.

Weak IP-based login restrictions may also reduce the effectiveness of defenses against automated authentication attempts.

However, username enumeration alone does not provide account access.

In this educational lab, successful authentication required a separate password-discovery step using the provided password wordlist.

---

## 6. Root Cause

The authentication process exhibited measurable timing differences when processing valid and invalid usernames.

Requests using valid usernames and long passwords required noticeably more time than comparable requests using invalid usernames.

This created a timing side channel that leaked information about account existence.

Additionally, the observed login restriction behavior suggested insufficient trust validation of client-supplied IP information.

---

## 7. Remediation

1. Minimize timing differences between valid and invalid username processing paths.

2. Perform comparable password-hashing operations for nonexistent usernames, such as using a dummy password hash.

3. Return consistent, generic authentication error messages.

4. Implement appropriate server-side rate limiting and account-based throttling.

5. Derive client IP information from trusted infrastructure and correctly configured reverse proxies.

6. Do not blindly trust user-supplied `X-Forwarded-For` headers when enforcing security controls.

7. Monitor repeated authentication failures and investigate abnormal login patterns.

8. Retest authentication behavior to verify that username validity cannot be reliably inferred from response timing.

9. Verify that changing untrusted request headers cannot bypass login restrictions.

---

## 8. Tools Used

- Burp Suite Community Edition
- Burp Repeater
- Burp Intruder
- Web Browser
- PortSwigger Web Security Academy

---

## 9. Key Takeaways

- Authentication vulnerabilities may be exposed through response timing even when error messages are identical.
- Long passwords can amplify timing differences in certain authentication implementations.
- A single slow response is insufficient evidence of username enumeration.
- Repeated measurements help distinguish timing patterns from network noise.
- HTTP status codes alone do not prove authentication success or failure.
- Client-controlled headers should not be blindly trusted for security decisions.
- Username enumeration and successful account authentication are separate outcomes.
- Successful authentication should be verified through actual access to the authenticated account.

---

## 10. References

- [PortSwigger: Authentication Vulnerabilities](https://portswigger.net/web-security/authentication)
- [OWASP: Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [OWASP: Credential Stuffing Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Credential_Stuffing_Prevention_Cheat_Sheet.html)
