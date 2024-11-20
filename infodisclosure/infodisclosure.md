# Tampering

This example demonstrates information disclosure by injecting malicious query objects to a NoSQL database.

## Steps to reproduce

1. Install all dependencies

   `$ npm install`

2. Insert test data in the MongoDB database. Make sure the mongod is up and running by typing the `mongosh` command in the termainal. If mongod process is up then you will see that the connection was successful. Command to insert test data:

   `$ npx ts-node insert-test-users.ts`

This will create a database in MongoDB called **infodisclosure**. Verify its presence by connecting with mongosh and running the command `show dbs;`.

2. Start the **insecure.ts** server

   `$ npx ts-node insecure.ts`

3. In the browser, pretend to be a hacker and type a malicious request

   ```
       http://localhost:3000/userinfo?username[$ne]=
   ```

4. Do you see user information being displayed despite the malicious request not having a valid username in the request?

## For you to do

Answer the following:

1. Briefly explain the potential vulnerabilities in **insecure.ts**

Answer:

The primary vulnerability in insecure.ts is the lack of input validation and sanitization for the username query parameter. The application directly uses the user-provided username in a MongoDB query without any checks:

```
const { username } = req.query;
const user = await User.findOne({ username: username as string }).exec();
```

This approach makes the application susceptible to NoSQL injection attacks. An attacker can inject MongoDB query operators into the username parameter to manipulate the query's behavior. Specifically, by injecting operators like $ne (not equal), the attacker can alter the query to return unintended results.

Furthermore, the application responds by sending the entire user object back to the client:
`res.send(`User: ${user}`);`

This can lead to information disclosure, as sensitive user data including hashed passwords or other confidential information may be exposed in the response.

2. Briefly explain how a malicious attacker can exploit them.

Answer:

An attacker can exploit this vulnerability by crafting a malicious HTTP GET request that injects a MongoDB query operator into the username parameter:
`http://localhost:3000/userinfo?username[$ne]=`

In this request, username[$ne]= translates to the following query:
`{ username: { $ne: '' } }`

This query tells MongoDB to find a user whose username is not equal to an empty string. Since usernames are generally not empty, MongoDB will return the first user it finds that meets this condition.

As a result, the attacker can:

<li>Bypass authentication checks: Retrieve user information without knowing a valid username.</li>
<li>Access sensitive data: Receive the entire user object in the response, which may include confidential information like passwords, email addresses, or personal details.</li>
<li>Harvest multiple user records: By manipulating the query further, the attacker could iterate over all user records.</li>
</br>
3. Briefly explain the defensive techniques used in **secure.ts** to prevent the information disclosure vulnerability?

</br>
Answer:

</br>

In secure.ts, several defensive measures are implemented to mitigate the vulnerability:

<li>Input Validation:

The application first checks if the username parameter is of type string. This ensures that the input is in the expected format before proceeding.</li>

<li>Input Sanitization:

The username input is sanitized to remove any non-alphanumeric characters. By stripping out special characters, the application prevents injection of MongoDB query operators like $ne, $gt, $lt, etc.</li>

<li>Secure Database Query:

The sanitized username is used in the database query. This ensures that the query executes as intended, without being manipulated by injected operators.</li>

<li>Error Handling with Try-Catch:

The query operation is wrapped in a try-catch block. This prevents the server from crashing due to unhandled exceptions and provides a generic error message to the client without revealing sensitive details.</li>

<li>Avoiding Sensitive Information Disclosure:

The application modifies the response to avoid sending sensitive user information. Instead of sending the entire user object, it sends only the necessary information, mitigating the risk of exposing confidential data.</li>

By implementing these measures, secure.ts effectively mitigates the NoSQL injection vulnerability and protects against unauthorized data access and information disclosure.
