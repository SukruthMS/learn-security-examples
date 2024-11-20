# Privilege Escalation

The example demonstrates a privilege escalation vulnerability and how to exploit it.

## Steps to reproduce

1. Install all dependencies

   `$ npm install`

2. Start the **insecure.ts** server

   `$ npx ts-node insecure.ts`

3. In the browser, send a GET request

   ```
       http://localhost:3000/send-form
   ```

4. Try different UserIds and see which one gives you authorized access to change the role of that user.

## For you to do

Answer the following:

1. Briefly explain the potential vulnerabilities in **insecure.ts**

Answer:

The insecure.ts application contains a critical vulnerability due to improper authentication and authorization mechanisms in the /update-role route. Here's a breakdown of the vulnerabilities:

<li>Trusting User-Provided Identification:

The application relies on the userId provided in the request body to identify the user making the request:

```
const { userId, newRole } = req.body;
const user = users.find(u => u.id === Number(userId));
```

This means that any client can specify any userId, effectively impersonating any user in the system, including administrators.</li>

<li>Insecure Authentication:

The application lacks a proper authentication system. There's no login process or session management to verify the identity of the requester. Instead, it assumes that the userId in the request body is trustworthy.</li>

<li>Flawed Authorization Logic:

The authorization check is based on the role of the user identified by the userId from the request:

```
if (user.role !== 'admin') {
  return res.status(403).json({ error: 'Unauthorized' });
}
```

Since the userId is supplied by the client, an attacker can specify the userId of an admin user to bypass this check.</li>

<li>Privilege Escalation via Parameter Manipulation:

After the (ineffective) authorization check, the application allows the role of any user to be updated without verifying if the requester has the necessary permissions:

```
user.role = newRole;
res.json({ message: 'User role updated successfully' });
```

This code modifies the role of the user identified by userId, which, as established, can be manipulated by the client.

Summary of Vulnerabilities:

<li>Lack of Proper Authentication: The server trusts the userId provided by the client, allowing impersonation.</li>
<li>Improper Authorization: Authorization checks are based on untrusted data, enabling unauthorized actions.</li>
<li>Privilege Escalation: Attackers can elevate their privileges by altering request parameters.</li>

</br>
2. Briefly explain how a malicious attacker can exploit them.

</br>

Answer:

An attacker can exploit these vulnerabilities by manipulating the userId and newRole parameters in the POST request to the /update-role endpoint. Here's how:

<li>Impersonate an Admin User:

The attacker sends a POST request with userId set to the admin's ID (e.g., 1), and the server believes the request is from the admin:

```
{
  "userId": "1",
  "newRole": "admin"
}
```

</li>

<li>Bypass Authorization Checks:

Since the server uses the userId from the request to perform the authorization check, specifying the admin's userId passes the user.role !== 'admin' condition.</li>

<li>Change Roles of Other Users:

The attacker can change the role of any user by setting userId to that user's ID and newRole to any desired role:

```
{
  "userId": "2",
  "newRole": "admin"
}
```

</li>

<li>Escalate Their Own Privileges:

If the attacker knows or guesses their own userId, they can elevate their role to admin:

```
{
  "userId": "3",
  "newRole": "admin"
}
```

</li>

<li>No Authentication Barrier:

Without authentication mechanisms like sessions or tokens, the server has no way to verify the true identity of the requester.</li>

3. Briefly explain the defensive techniques used in **secure.ts** to prevent the privilege escalation vulnerability?

Answer:

In secure.ts, several defensive strategies are implemented to address the vulnerabilities:

<li>Session-Based Authentication:

The application uses express-session middleware to manage user sessions:

```
app.use(session({
  secret: 'your_secret_key',
  cookie: { httpOnly: true, sameSite: 'strict' },
  resave: false,
  saveUninitialized: true,
}));
```

This allows the server to maintain a session for each logged-in user, storing the userId securely on the server side.</li>

<li>Server-Side User Identification:

Instead of trusting the userId from the client, the server relies on req.session.userId to identify the authenticated user:

```
if (!req.session.userId) {
  return res.status(401).json({ error: 'Unauthorized' });
}
```

The userId in the session is set during a proper login process (not shown in the code), ensuring that it represents the true identity of the requester.</li>

<li>Proper Authorization Checks:

The server retrieves the logged-in user's details and checks their role:

```
const loggedInUser = users.find(u => u.id === Number(req.session.userId));
if (!loggedInUser || loggedInUser.role !== 'admin') {
  return res.status(403).json({ error: 'Forbidden' });
}
```

This ensures that only users with the admin role can perform the role update operation.</li>

<li>Separating User Identification from Target User:

The userId in the request body is now only used to identify which user's role needs to be updated, not to authenticate the requester.

```
const userToUpdate = users.find(u => u.id === userId);
if (!userToUpdate) {
  return res.status(404).json({ error: 'User not found' });
}

userToUpdate.role = newRole;
res.json({ message: 'User role updated successfully' });
```

This prevents attackers from manipulating the userId to impersonate other users.</li>

<li>Session Security Enhancements:

The session cookies are configured with security attributes:

`cookie: { httpOnly: true, sameSite: 'strict' }`

`httpOnly`: Prevents client-side scripts from accessing the cookie, mitigating XSS attacks.

`sameSite: 'strict'`: Prevents the browser from sending the cookie along with cross-site requests, reducing CSRF risks.</li>

<li>Error Handling and Response Codes:

The application returns appropriate HTTP status codes (401 Unauthorized, 403 Forbidden, 404 Not Found), providing clear feedback and enhancing security.</li>

By addressing both authentication and authorization properly, secure.ts prevents privilege escalation attacks that were possible in insecure.ts.
