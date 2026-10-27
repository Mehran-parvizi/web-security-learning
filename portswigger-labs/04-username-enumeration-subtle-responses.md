# Title:
Username Enumeration via Subtle Differences in Login Error Messages

# Steps to Reproduce:

1. Intercept a login request using Burp Suite and send it to **Burp Intruder**.

   ```http
   POST /login HTTP/2
   Content-Type: application/x-www-form-urlencoded

   username=testuser123&password=123456
   ```

2. Configure the `username` parameter as the payload position and keep the `password` parameter fixed throughout the username enumeration test.

3. Load a list of candidate usernames and run the Intruder attack. Compare the responses using the response body and, if configured, Burp Intruder's **Grep - Match** feature to identify differences in the error messages.

4. Identify `ag` as a valid username based on the observed difference in the trailing period of the error message.

   - **Invalid username:** `Invalid username or password.`
   - **Valid username (`ag`) with an incorrect password:** `Invalid username or password`

   Note: These example messages must be verified against the actual lab responses before submitting the report.

5. In the authorized PortSwigger educational lab, test the provided password candidates against the identified username `ag`, using Burp Intruder.

6. Identify the lab password `biteme` and submit the credentials through the login form.

7. Verify successful authentication by confirming that the account is accessible and the lab displays **Lab Solved**.
