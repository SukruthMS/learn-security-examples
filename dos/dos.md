# Denial-of-Service (DoS)

This example demonstrates DoS vulnerabilities and how they can be exploited.

## Steps to reproduce

1. Install all dependencies

   `$ npm install`

2. Ignore if you have already done this once. Insert test data in the MongoDB database. Make sure the mongod is up and running by typing the `mongosh` command in the termainal. If mongod process is up then you will see that the connection was successful. Command to insert test data:

   `$ npx ts-node insert-test-users.ts`

This will create a database in MongoDB called **infodisclosure**. Verify its presence by connecting with mongosh and running the command `show dbs;`.

2. Start the **insecure.ts** server

   `$ npx ts-node insecure.ts`

3. In the browser, pretend to be a hacker and type a malicious request

   ```
       http://localhost:3000/userinfo?id[$ne]=
   ```

4. Do you see the server crashing?

## For you to do

Answer the following:

1. Briefly explain the potential vulnerabilities in **insecure.ts** that can lead to a DoS attack.

Answer:
</br>
The primary vulnerability in insecure.ts stems from the lack of input validation on the id query parameter received from the client. The code directly uses this parameter in a MongoDB query without any sanitization.

This approach makes the application susceptible to NoSQL injection attacks. An attacker can inject malicious query operators into the id parameter, altering the structure of the query object. For instance, by injecting $ne (not equal) operators or other MongoDB query selectors, the attacker can manipulate the query to perform unintended operations.

Such manipulated queries can lead to expensive database operations, like full collection scans, which consume excessive server resources (CPU, memory, I/O). If the server cannot handle the load, it may slow down significantly or crash, resulting in a Denial-of-Service (DoS) condition.

2. Briefly explain how a malicious attacker can exploit them.

Answer:
</br>
An attacker can exploit this vulnerability by crafting a special HTTP GET request that includes a malicious id parameter. By injecting MongoDB query operators into the id parameter, the attacker changes the intended query logic. For example:

`http://localhost:3000/userinfo?id[$ne]=`

In this request, id[$ne]= translates to an id parameter where id is an object containing the $ne operator:

`{ _id: { $ne: '' } }`

This query asks MongoDB to find a user whose \_id is not equal to an empty string, which effectively returns the first user it finds. However, if the database contains a large number of records, MongoDB may perform a full collection scan to satisfy this query. This operation can be resource-intensive and degrade server performance.

By repeatedly sending such requests or crafting even more complex queries, the attacker can overload the server's resources, causing it to become unresponsive or crash, thereby achieving a Denial-of-Service attack.

3. Briefly explain the defensive techniques used in **secure.ts** to prevent the DoS vulnerability?

Answer:
</br>
In secure.ts, the application implements a rate-limiting middleware using the express-rate-limit package.

Defensive Techniques:

<li>
Rate Limiting: By limiting each IP address to a maximum of one request every five seconds, the server reduces the risk of being overwhelmed by rapid, repeated requests from a single source. This helps mitigate DoS attacks by controlling the flow of incoming traffic and ensuring that server resources are not exhausted by malicious actors.
</li>
</br>
<li>
Error Handling with Try-Catch: The route handler is wrapped in a try-catch block to gracefully handle any errors that occur during the database query. This prevents the server from crashing due to unhandled exceptions, which could be triggered by malicious input or unexpected runtime errors.
</li>
