# Tampering

This example demonstrates tampering through script injection.

## Steps to reproduce

1. Install all dependencies

   `npm install`

2. Start the **insecure.ts** server

   `npx ts-node insecure.ts`

3. In the browser, type a potentially malicious script in the name field of the form

   ```
       <script> document.body.innerHTML = "<a href='https://google.com'> Gotcha </a>"</script>
   ```

4. Do you see the potentially malicious hyperlink being injected into the form?

## For you to do

Answer the following:

1. Briefly explain the potential vulnerabilities in **insecure.ts**

Answer:

The primary vulnerability in insecure.ts is the lack of input sanitization, leading to a Cross-Site Scripting (XSS) vulnerability. Here's how it occurs:

1.  **Unsanitized User Input Stored in Session**: In the /register route, the application accepts user input from the name field and stores it directly in the session without any sanitization:

    ```
    app.post("/register", (req: Request, res: Response) => {
    req.session.user = req.body.name.trim();
    res.send(`<p>Thank you</p> <a href="/">Back home</a>`);
    });
    ```

2.  **Unsanitized User Input Rendered in HTML**: In the root / route, the application retrieves the name from the session and injects it directly into the HTML response:

    ```
    app.get("/", (req: Request, res: Response) => {
    let name = "Guest";
    if (req.session.user) name = req.session.user;

    res.send(`
    <h1>Welcome, ${name}</h1>
    <form action="/register" method="POST">
        <input type="text" name="name" placeholder="Your name">
        <button>Submit</button>
    </form>
    <form action="/forget" method="POST">
        <button>Logout</button>
    </form>
    `);
    });
    ```

3.  **Vulnerability Impact**:
    - By embedding user-provided input directly into the HTML without any sanitization or encoding, the application allows malicious scripts to be injected into the page. This is a classic XSS vulnerability, where an attacker can execute arbitrary JavaScript code in the context of other users visiting the site.

</br>

2. Briefly explain how a malicious attacker can exploit them.

Answer:

An attacker can exploit this vulnerability through the following steps:

1. **Submit Malicious Script as Input**: The attacker enters a malicious script into the name field of the registration form. For example:

   ```
   <script>
   document.body.innerHTML = "<a href='https://malicious-site.com'>Click here</a>";
   </script>
   ```

2. **Malicious Script Stored in Session**: The unsanitized input is stored in the user's session:

   ```
   req.session.user = req.body.name.trim();
   ```

3. **Malicious Script Injected into HTML Response**: When the user (or another user sharing the same session) accesses the root / route, the application renders the HTML page with the injected script:

   ```
   <h1>Welcome, <script>...</script></h1>
   ```

4. **Execution of Malicious Code**: The browser interprets and executes the injected script. Potential consequences include:

   - Session Hijacking: The script could send the user's session cookie to the attacker.
   - Phishing Attacks: Redirecting the user to a malicious website that mimics a legitimate one.
   - Defacement: Altering the content of the website to display offensive or misleading information.
   - Malware Distribution: Initiating downloads of malicious software without user consent.

</br>

3. Briefly explain why **secure.ts** does not have the same vulnerabilties?

Answer:

`secure.ts` addresses the XSS vulnerability by implementing input sanitization:

1. **Sanitizing User Input Before Storing**: In the /register route, the application sanitizes the user's input using the escapeHTML function before storing it in the session:

```
app.post("/register", (req: Request, res: Response) => {
  const sanitizedName = escapeHTML(req.body.name.trim());
  req.session.user = sanitizedName;
  res.send(`<p>Thank you</p> <a href="/">Back home</a>`);
});
```

2. **escapeHTML Function Implementation**: The escapeHTML function replaces special HTML characters with their corresponding HTML entities:

```
function escapeHTML(input: string): string {
  return input
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#39;");
}
```

3. **Effect of Sanitization**:

   - Neutralizing Malicious Scripts: By escaping characters like <, >, ", and ', any script tags or HTML elements in the user input are rendered harmless. They appear as plain text in the browser rather than being executed or rendered as HTML.

   - Preserving Intended Display: The user's input is displayed back to them safely, without altering the functionality or appearance of the page.

4. **Example of Sanitized Output**: If the attacker inputs the malicious script:

```
<script>document.body.innerHTML = "<a href='https://malicious-site.com'>Click here</a>";</script>
```

After sanitization, it becomes:

```
&lt;script&gt;document.body.innerHTML = &quot;&lt;a href=&#39;https://malicious-site.com&#39;&gt;Click here&lt;/a&gt;&quot;;&lt;/script&gt;
```

This sanitized string is displayed as-is on the page, and the browser does not execute it as code.

**Conclusion**:
By implementing proper input sanitization in secure.ts, the application effectively prevents XSS attacks. This ensures that any user-supplied data is safely handled and cannot be used to inject malicious scripts, thereby maintaining the integrity and security of the application.
