# Repudiation

The example demonstrates a vulnerability that can lead to repudiation by malicious users attempting to access the services provided by a server.

## Steps to reproduce

1. Install all dependencies

   `$ npm install`

2. Run the server **insecure.ts**.

3. Pretend to be a malicous user and interact with the services by sending requests from the browser.

4. Do you think your actions can be repudiated?

## For you to do

1. Briefly explain the vulnerability.

Answer:

The primary vulnerability in the insecure.ts application is the lack of proper logging and authentication mechanisms, which can lead to repudiation attacks. Repudiation refers to a scenario where a user performs an action but later denies doing so, and the system lacks sufficient evidence to prove otherwise.

In insecure.ts, the application allows users to send and retrieve messages without any form of authentication or accountability:

<li>No Authentication: Any user can submit messages or retrieve all messages without verifying their identity. The application does not enforce any user login or verification process.</li>

<li>No Logging or Audit Trail: There is no logging mechanism to record user actions, requests, or errors. The server does not keep track of who sent messages, who retrieved them, or when these actions occurred.</li>
</br>
As a result, malicious users can:

<li>Perform Actions Anonymously: Users can send messages or access others' messages without leaving any trace. This anonymity enables them to engage in malicious activities without fear of detection.</li>

<li>Deny Responsibility: Since there's no record of user actions, malicious users can repudiate their actions. If harmful content is posted or unauthorized access occurs, the users responsible can deny involvement, and the system cannot provide evidence to hold them accountable.</li>
</br>
Implications of the Vulnerability:

<li>Security Risks: Without authentication and logging, the system cannot prevent unauthorized access or misuse of data.</li>

<li>Lack of Accountability: Users cannot be held responsible for their actions, undermining trust in the system.</li>

<li>Compliance Issues: The absence of audit trails may violate legal or regulatory requirements for data protection and user accountability.</li>
</br>

2. Briefly explain why the vulnerability is addressed in **secure.ts**.

Answer:

The secure.ts version addresses the repudiation vulnerability through the following measures:

1. Audit Logging Implementation:

<li>Request Logging Middleware: The application uses middleware to log every incoming request, capturing essential details such as the timestamp, HTTP method, URL, and client IP address:

```
app.use((req: Request, res: Response, next: NextFunction) => {
    const logEntry = `[${getCurrentDateTime()}] ${req.method} ${req.url} - ${req.ip}`;
    logStream.write(logEntry + '\n');
    next();
});
```

</li>
</br>
<li>Action-Specific Logging: When a user sends a message, the action is logged with the user's name and the message content:

```
const logEntry = `[${getCurrentDateTime()}] Message sent by ${user}: ${message}`;
logStream.write(logEntry + '\n');
```

</li>
</br>
<li>Error Logging: Any errors encountered during request processing are logged with detailed information:

```
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
    console.error(err.stack);
    const logEntry = `[${getCurrentDateTime()}] Error: ${err.message}`;
    logStream.write(logEntry + '\n');
    res.status(500).send(`Something went wrong: ${err.message}`);
});
```

</li>
</br>
2. Authentication Mechanisms:
</br></br>
<li>Although the authentication is simulated in the code (with const isAuthenticated = true;), placeholders are present for integrating a real authentication system:

```
if (!isAuthenticated) {
    return res.status(401).json({ error: 'Unauthorized access. Please login.' });
}
```

</li>
<li>This indicates an intention to restrict access to authenticated users, ensuring that only verified individuals can perform actions.</li>
</br>

3. Non-Repudiation Enforcement:

<li>By logging all actions along with user information and timestamps, the system creates an audit trail. This trail can be used to trace actions back to specific users, preventing them from denying their activities.</li>

</br>
3. Which design pattern is used in the secure version to address the vulnerability? Briefly explain how it works?

</br>
Answer:

</br>

The design pattern used in the secure version is the Audit Logging pattern, which is a fundamental security practice aimed at ensuring non-repudiation.

Explanation of the Audit Logging Pattern:

<li>Purpose: Audit logging involves recording detailed information about system events, user actions, and errors to create an immutable record that can be reviewed and analyzed.</li>
</br>
<li>
Implementation in secure.ts:

1. Middleware Usage: Logging is implemented as middleware, which intercepts requests and responses without altering the core application logic.

2. Logging Details:
<ul>
<li>Timestamp: Records the exact date and time of each action using getCurrentDateTime().</li>

<li>User Identification: Captures the user's identity (e.g., username or IP address) to associate actions with specific individuals.</li>

<li>Action Details: Logs the nature of the action performed, such as sending a message or retrieving messages.</li>

<li>Error Information: Logs any errors that occur, including stack traces and error messages.</li>
</ul>

3. Persistent Storage: Logs are written to a file (server.log) using a file stream, ensuring that the records are stored persistently and can be accessed later for analysis.

</li>

<li>How It Works:</li>

<ul>Interception of Requests: The logging middleware intercepts every incoming request and outgoing response, capturing relevant data before passing control to the next middleware or route handler.</ul>

<ul>Recording Events: All significant events, including user actions and errors, are recorded with sufficient detail to understand what happened, when, and who was involved.</ul>

<ul>Storage and Protection of Logs: Logs are stored in a secure and tamper-evident manner, preventing unauthorized modifications. In a production environment, additional measures like log rotation, encryption, and access controls may be implemented.</ul>

<li>Benefits of Audit Logging:

<ul>Non-Repudiation: Users cannot deny their actions because there is a verifiable record linking them to their activities.</ul>

<ul>Security Monitoring: Administrators can monitor logs to detect suspicious activities, such as repeated unauthorized access attempts.</ul>

<ul>Forensic Analysis: In the event of a security incident, logs provide valuable information for investigating and understanding the breach.</ul>

<ul>Compliance: Many regulatory frameworks require audit logging to ensure data security and privacy.</ul>
</li>
</br>
<li>Conclusion: By incorporating the Audit Logging design pattern, secure.ts effectively mitigates the repudiation vulnerability present in insecure.ts. It enhances the application's security by ensuring that all user actions are recorded and traceable, thereby enforcing accountability and deterring malicious behavior.</li>
