# Content Security Policy (CSP)
- [Content Security Policy (CSP)](#content-security-policy-csp)
  - [General Description](#general-description)
  - [Control Resource Loading](#control-resource-loading)
    - [Fetch Directives](#fetch-directives)

## General Description
+ It is a series of instructions, sent from a website to a browser, which instructs the browser to place restrictions on the things that the code comprising the site is allowed to do.
+ It is delivered using the `Content-Security-Policy` HTTP response header.
+ The policy is specified as a series of directives, separated by semi-colons.
+ To do so with HTML, use the `http-equiv` attribute in the `<meta>` tag. But it does not support all the CSP features.

+ Syntax:
  ```
  Content-Security-Policy: DIRECTIVE VALUE; DIRECTIVE VALUE.....
  ```

+ Example:
  ```
  Content-Security-Policy: default-src 'self'; img-src 'self' example.com
  ```
  > The `default-src` directive tells the browser to load resources that are **same-origin** with the document.

  > The `img-src` directive tells the browser to load images that are either **same-origin** or are served from `example.com`. 

## Control Resource Loading
+ It is used to control the resources that a document is allowed to load. Acts as a defense against **XSS attacks**.
+ An XSS attack is one in which an attacker is able to execute arbitrary code in the context of the target website. 
  + This code is able to do anything that the website's own code could do.
  + Thid attack happens when a website accepts user (attacker crafted) input and includes it in the webpage without sanitizing it: that is, without ensuring that it can't be executed as JavaScript.
+ A CSP provides a complementary protection, which can protect the website even if sanitization fails.

### Fetch Directives
+ They are used to specify the category of resource(s) that a document is allowed to load.
+ There are different fetch directives for different types of resource. For example:
  + `script-src` for JavaScript sources.
  + `style-src` for CSS Stylesheets sources.
  + `img-src` for image sources.

+ Each fetch directive is specified as either the single keyword `'none'` or one or more **source expressions**, separated by spaces. When more than one source expression is listed: if any of the methods allow the resource, the resource is allowed.
+ The keyword `'self'` allows resources which are same-origin with the document.

+ Example: 
  ```
  Content-Security-Policy: default-src 'self'; img-src 'self' example.com
  ```
  > The `default-src` directive tells the browser to only load resources that are **same-origin** with the document.

  > The `img-src` directive tells the browser to load images that are either **same-origin** or are served from `example.com`. 

+ To block a resource entirely, use `'none'`
  ```
  Content-Security-Policy: object-src 'none'
  ```

+ To allow only trusted sources, use `'nonce'`
  ```
  Content-Security-Policy: script-src 'nonce-416d1177-4d12-4e3b-b7c9-f6c409789fb8'
  ```
  > A `'nonce'` refers to a **number used once**, is a randomly generated server-side value. This value is injected in the CSP and the server-side trusted scripts. When the browser receives the response, it compares the `'nonce'` value in the CSP and the scripts. Only if it is matched it executes the script.

  > Attacker injected scripts are client-side and that is why they don't get this `'nonce'` value. And it is extremely hard to guess it or brute-force it as they are generated using **Cryptographically Secure Pseudo-Random Number Generator (CSPRNG)**. And they are minimum 16-bytes (or 128-bit) long values, which are inefficient to bruteforce.

+ Using `'nonce'` means that the server can't serve static HTML, as the `'nonce'` attribute is different everytime. To achieve this, the server would use a templating engine to insert the `'nonce'`. Example:
  ```js
  function content(nonce) {
    return `
      <script nonce="${nonce}" src="/main.js"></script>
      <script nonce="${nonce}">console.log("hello!");</script>
      <h1>Hello world</h1> 
      `;
  }

  app.get("/", (req, res) => {
    const nonce = crypto.randomUUID();
    res.setHeader("Content-Security-Policy", `script-src 'nonce-${nonce}'`);
    res.send(content(nonce));
  });
  ```

+ Using hashes to maintain integrity. With this method, the server:
  1. calculates a hash of the script contents using an algorithm like SHA-256,
  2. creates a Base64 encoding of the result,
  3. appends a prefix identifying the hash algorithm used, like `sha256-`
  4. Now add this to the CSP.
     ```
     Content-Security-Policy: script-src 'sha256-cd9827ad...'
     ```
     > When the browser receives the document, it hashes the script, compares the result with the value from the header, and loads the script only if they match.
  5. External scripts must also include the `integrity` attribute for this to work.
     ```js
     const hash1 = "sha256-ex2O7MWOzfczthhKm6azheryNVoERSFrPrdvxRtP8DI=";
     const hash2 = "sha256-H/eahVJiG1zBXPQyXX0V6oaxkfiBdmanvfG9eZWSuEc=";

     const csp = `script-src '${hash1}' '${hash2}'`;
     const content = `
       <script src="./main.js" integrity="${hash2}"></script>
       <script>console.log("hello!");</script>
         <h1>Hello world</h1> 
         `;

     app.get("/", (req, res) => {
       res.setHeader("Content-Security-Policy", csp);
       res.send(content);
     });
     ```
    > Every script will have a separate hash, in contrast to every script having same `'nonce'`.

    > This can also be done for static HTML as the hashes are going to be same as long as the file is not manipulated.

    > This makes hash-based policies more suitable for static pages or websites that rely on client-side rendering.

+ Scheme-based policies to restrict resource loading from certain schemes only. Example:
  ```
  Content-Security-Policy: default-src https:
  ```

+ To allow resources from certain hosts only, like certain trusted CDN's only, we can combine the hostname with wildcards.
  ```
  Content-Security-Policy: img-src *.example.org

  Content-Security-Policy: img-src *.example.org example.org
  ```
  > This will allow resources from `.example.com` domain and sub-domains only.

# References
+ [CSP MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
+ ChatGPT
