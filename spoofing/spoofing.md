# Spoofing

This example demonstrates spoofind through two ways -- Stealing cookies programmatically and cross site request forgery (CSRF).

## Steps to reproduce the vulnerability

1. Install dependencies

   `$ npx install`

2. Start the **insecure.ts** server

   `npx ts-node insecure.ts`

3. Start the malicious server **mal.ts**

   `npx ts-node mal.ts`

4. Open **http://localhost:8000** in a browser, type a name and Submit.

5. Open the **Application** tab in the Browser's inspect pane. Find the **Cookies** under **Storage**. You should see a **connect.sid** cookie being set.

6. Open the HTML file **mal-steal-cookie.html** file in the same browser (different tab). Open inspect and view the console.

7. Click the link in the HTML file. Do you see the cookie being stolen in the console?

8. Open the HTML file **mal-csrf.html** file in the same browser (different tab). What do you see if the user has not logged out of **insecure.ts**? What do you see if the user has logged out?

## For you to answer

1. Briefly explain the spoofing vulnerability in **insecure.ts**.

Answer:

The insecure.ts application contains significant spoofing vulnerabilities due to improper session cookie configuration and lack of protections against Cross-Site Request Forgery (CSRF) and session hijacking. Here are the key issues:

1. **Session Cookie Not Marked as HttpOnly**:

   - **Issue**: The session cookie is configured with httpOnly: false:

     ```
     app.use(
       session({
         secret: "SOMESECRET",
         cookie: { httpOnly: false },
         resave: false,
         saveUninitialized: false,
       })
     );
     ```

     This setting allows client-side scripts (JavaScript) to access the session cookie via document.cookie.

   - **Impact**: An attacker can exploit this by injecting malicious JavaScript code into the user's browser to steal the session cookie. Once obtained, the attacker can hijack the user's session and impersonate them on the server.

2. **Lack of CSRF Protection**:

   - **Issue**: The session cookie does not have the SameSite attribute set, meaning it defaults to None or an insecure default, depending on the browser.
   - **Impact**: Without the SameSite attribute, the browser will include the session cookie in cross-origin requests. This makes the application vulnerable to CSRF attacks, where an attacker can trick a user's browser into making unintended requests to the server, leveraging the user's authenticated session.

3. **No CSRF Tokens or Additional Protections**:

   - **Issue**: The application does not implement CSRF tokens or other mechanisms to verify the legitimacy of POST requests.
   - **Impact**: Attackers can craft malicious forms or requests that the user's browser will automatically submit, performing actions on behalf of the user without their knowledge or consent.

4. **Summary of the Vulnerabilities**:
   - Session Hijacking Risk: Due to httpOnly: false, the session cookie can be accessed and stolen via JavaScript, leading to session hijacking.
   - CSRF Vulnerability: Without SameSite enforcement or CSRF tokens, the application is susceptible to CSRF attacks, enabling unauthorized actions.

These vulnerabilities enable attackers to spoof legitimate users, impersonating them to perform sensitive operations on the server.

</br>

2. Briefly explain different ways in which vulnerability can be exploited.

Answer:

The vulnerabilities in insecure.ts can be exploited through several attack vectors:

1. **Session Hijacking via Cookie Theft**:

   - **Method**:

     - An attacker creates a malicious web page (mal-steal-cookie.html) that the user visits.
     - The page contains a link to a malicious server (http://localhost:8001/malhome) served by mal.ts.
     - When the user clicks the link, the malicious server responds with a script that accesses document.cookie:

       ```
       <script>
         console.log("Session = " + document.cookie);
       </script>
       ```

     - Since httpOnly is set to false, the session cookie (connect.sid) is accessible via document.cookie.

   - **Impact**:
     - The attacker can read and steal the session cookie from the user's browser.
     - With the stolen cookie, the attacker can impersonate the user by injecting the cookie into their own browser, gaining unauthorized access to the user's session.

2. **Cross-Site Request Forgery (CSRF) Attack**:

   - **Method**:

     - An attacker hosts a malicious form (`mal-csrf.html`) that auto-submits a request to `/sensitive`:

       ```
       <form
         action="http://localhost:8000/sensitive"
         method="POST"
         id="csrfForm"
       >
         <input type="hidden" name="username" value="attacker" />
       </form>
       <script>
         document.getElementById("csrfForm").submit();
       </script>
       ```

     - When the user visits this page, their browser automatically submits the form to the vulnerable server (insecure.ts), including their session cookie.

   - **Impact**:
     - If the user is logged in (i.e., has an active session), the server processes the request as if it came directly from the user.
     - The attacker can perform sensitive operations on behalf of the user without their knowledge, such as triggering actions that only the user should be able to perform.

3. **Cross-Site Scripting (XSS)**:

   - **Method**:

     - If other parts of the application are vulnerable to XSS, an attacker could inject malicious scripts that access document.cookie.
     - This could occur through user input fields that are not properly sanitized.

   - **Impact**:
     - Similar to the cookie theft method, the attacker can steal session cookies and hijack user sessions.

4. Exploitation Scenarios:

   - Unauthorized Access to Sensitive Operations: By exploiting CSRF, an attacker can invoke the /sensitive endpoint, performing actions reserved for privileged users (e.g., 'Admin').
   - Session Impersonation: Through session hijacking, attackers can impersonate users, accessing their personal data, and performing actions on their behalf.

5. Consequences:

   - Data Breach: Unauthorized access to sensitive user information.
   - Privilege Escalation: Attackers performing actions beyond their authorization level.
   - Account Compromise: Full control over user accounts leading to potential fraud or misuse.

</br>

3. Briefly explain why **secure.ts** does not have the spoofing vulnerability in **insecure.ts**.

Answer:

The secure.ts application addresses the spoofing vulnerabilities by implementing proper security measures in the session cookie configuration:

1. **Session Cookie Marked as HttpOnly**:

   ```
   app.use(
   session({
       secret: `${secret}`,
       cookie: {
           httpOnly: true,
           sameSite: true,
       },
       resave: false,
       saveUninitialized: false
   })
   );
   ```

- **Effect**:
  - Setting httpOnly: true means the session cookie cannot be accessed via client-side JavaScript (document.cookie).
  - This prevents attackers from stealing the session cookie through XSS attacks or malicious scripts.

2. **SameSite Attribute Set to Protect Against CSRF**:

   - **Configuration**:

     - The sameSite attribute is set to true, which enforces SameSite=Lax policy (or SameSite=Strict depending on the implementation).

   - **Effect**:
     - The browser will not include the session cookie in cross-site requests initiated by third-party sites.
     - This effectively mitigates CSRF attacks, as malicious forms or scripts on external sites cannot cause the user's browser to send authenticated requests to the server.

3. **Use of a Secret Passed as an Argument**:

   - **Configuration**: The session secret is passed as an argument when starting the server:

     ```
     const secret: string = process.argv[2];
     ```

   - **Effect**:
     - This ensures that the session secret is not hard-coded and can be securely managed, reducing the risk of session hijacking through secret compromise.

_How These Measures Mitigate Spoofing Vulnerabilities:_

- Prevents Session Cookie Theft: By marking the cookie as HttpOnly, even if an attacker manages to inject malicious JavaScript into the page, they cannot access the session cookie, protecting against session hijacking.

- Blocks CSRF Attacks: With SameSite enforcement, the browser ignores cross-origin requests that attempt to use the session cookie, ensuring that only requests originating from the same site can access authenticated endpoints.

- Enhances Session Security: Secure cookie configurations increase the overall resilience of the application against common web vulnerabilities that lead to spoofing.

_Consequences of the Secure Configuration:_

- Users are Protected Against Impersonation: Attackers cannot easily hijack user sessions or perform actions on their behalf.

- Sensitive Operations Are Secured: The /sensitive endpoint cannot be triggered by unauthorized parties through CSRF attacks.

**Conclusion**:
By appropriately configuring session cookies with httpOnly and SameSite attributes in secure.ts, the application effectively mitigates the spoofing vulnerabilities present in insecure.ts. These settings ensure that:

- Session cookies are inaccessible to client-side scripts, preventing theft.
- Cross-origin requests cannot exploit authenticated sessions, blocking CSRF attacks.

This demonstrates the importance of secure session management in protecting web applications from spoofing threats.
