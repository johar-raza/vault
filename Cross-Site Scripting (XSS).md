#cheatsheet #google-gruyere
# Cross-Site Scripting (XSS)
---

Cross-site scripting (XSS) is a vulnerability that permits an attacker to inject code (typically HTML or JavaScript) into contents of a website not under the attacker's control. When a victim views such a page, the injected code executes in the victim's browser. Thus, the attacker has bypassed the browser's [same origin policy](https://www.google.com/search?q=same+origin+policy) and can steal victim's private information associated with the website in question.

In a **reflected XSS** attack, the attack is in the request itself (frequently the URL) and the vulnerability occurs when the server inserts the attack in the response verbatim or incorrectly escaped or sanitized. The victim triggers the attack by browsing to a malicious URL created by the attacker. In a **stored XSS** attack, the attacker stores the attack in the application (e.g., in a snippet) and the victim triggers the attack by browsing to a page on the server that renders the attack, by not properly escaping or sanitizing the stored data.

To understand how this could happen: suppose the URL `https://www.google.com/search?q=flowers` returns a page containing the HTML fragment

`<p>Your search for 'flowers'
returned the following results:</p>`

that is, the value of the query parameter `q` is inserted verbatim into the page returned by Google. If `www.google.com` did not do any validation or escaping of `q` (it does), an attacker could craft a link that looks like this:

`https://www.google.com/search?q=flowers+%3Cscript%3Eevil_script()%3C/script%3E`

and trick a victim into clicking on this link. When a victim loads this link, the following page gets rendered in the victim's browser:

`<p>Your search for 'flowers<script>evil_script()</script>'
returned the following results:</p>`

And the browser executes `evil_script()`. And since the page comes from `www.google.com`, `evil_script()` is executed in the context of `www.google.com` and has access to all the victim's browser state and cookies for that domain.

Note that the victim does not even need to explicitly click on the malicious link. Suppose the attacker owns `www.evil.example.com`, and creates a page with an `<iframe>` pointing to the malicious link; if the victim visits `www.evil.example.com`, the attack will silently be activated.

---

- Check all input fields
- Check all URLs that contains parameters
- Check for file uploads (try to upload html files with script tags and execute them)
- 