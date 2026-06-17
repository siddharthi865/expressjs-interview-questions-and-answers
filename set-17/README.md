# Set 17

| S.No. | Question                                                                                                                                                          |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is the purpose of HTTP response headers like `Content-Type`?](#question-1-what-is-the-purpose-of-http-response-headers-like-content-type)                   |
| 2.    | [How do you prevent accidental exposure of sensitive configuration values?](#question-2-how-do-you-prevent-accidental-exposure-of-sensitive-configuration-values) |
| 3.    | [What is the role of `req.headers` in Express.js?](#question-3-what-is-the-role-of-reqheaders-in-expressjs)                                                       |
| 4.    | [How do you make middleware run globally for all routes?](#question-4-how-do-you-make-middleware-run-globally-for-all-routes)                                     |
| 5.    | [What is API testing and why is it important for Express.js applications?](#question-5-what-is-api-testing-and-why-is-it-important-for-expressjs-applications)    |
| 6.    | [How do you dynamically register routes in Express.js?](#question-6-how-do-you-dynamically-register-routes-in-expressjs)                                          |
| 7.    | [What are middleware factories in Express.js?](#question-7-what-are-middleware-factories-in-expressjs)                                                            |
| 8.    | [How do higher-order middleware functions work?](#question-8-how-do-higher-order-middleware-functions-work)                                                       |
| 9.    | [How would you implement request sanitization in Express.js?](#question-9-how-would-you-implement-request-sanitization-in-expressjs)                              |
| 10.   | [What is input canonicalization and why is it important?](#question-10-what-is-input-canonicalization-and-why-is-it-important)                                    |
| 11.   | [How do you prevent parameter pollution attacks in Express.js APIs?](#question-11-how-do-you-prevent-parameter-pollution-attacks-in-expressjs-apis)               |
| 12.   | [What are common security headers recommended for APIs?](#question-12-what-are-common-security-headers-recommended-for-apis)                                      |
| 13.   | [How do you securely handle API secrets in CI/CD pipelines?](#question-13-how-do-you-securely-handle-api-secrets-in-cicd-pipelines)                               |
| 14.   | [What is the purpose of nonce values in web security?](#question-14-what-is-the-purpose-of-nonce-values-in-web-security)                                          |
| 15.   | [How do you implement API usage quotas for customers?](#question-15-how-do-you-implement-api-usage-quotas-for-customers)                                          |
| 16.   | [What are the trade-offs between API keys and OAuth authentication?](#question-16-what-are-the-trade-offs-between-api-keys-and-oauth-authentication)              |
| 17.   | [How do OAuth access tokens differ from refresh tokens?](#question-17-how-do-oauth-access-tokens-differ-from-refresh-tokens)                                      |
| 18.   | [What is the purpose of scopes in OAuth-based APIs?](#question-18-what-is-the-purpose-of-scopes-in-oauth-based-apis)                                              |
| 19.   | [How do you implement tenant-aware authorization in Express.js?](#question-19-how-do-you-implement-tenant-aware-authorization-in-expressjs)                       |
| 20.   | [What are the risks of storing JWTs in browser local storage?](#question-20-what-are-the-risks-of-storing-jwts-in-browser-local-storage)                          |

## Question 1. What is the purpose of HTTP response headers like `Content-Type`?

### Direct Answer

HTTP response headers like `Content-Type` provide metadata about the response being sent from the server to the client. The `Content-Type` header specifically tells the client **what type of data is in the response body** (such as JSON, HTML, CSS, or an image), allowing the browser or API client to correctly interpret and process the content.

---

# Detailed Explanation

HTTP responses consist of three parts:

1. **Status Line** (e.g., `HTTP/1.1 200 OK`)
2. **Response Headers**
3. **Response Body**

Example:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 52

{
  "id": 1,
  "name": "John"
}
```

Here, the `Content-Type` header tells the client that the body contains JSON.

---

# Why `Content-Type` is Important

Without `Content-Type`, the client would not know how to interpret the response.

For example:

| Content-Type             | Meaning        |
| ------------------------ | -------------- |
| `application/json`       | JSON data      |
| `text/html`              | HTML page      |
| `text/plain`             | Plain text     |
| `text/css`               | CSS stylesheet |
| `application/javascript` | JavaScript     |
| `image/png`              | PNG image      |
| `application/pdf`        | PDF document   |
| `application/xml`        | XML data       |

The browser or API client behaves differently depending on this header.

---

# Express.js Example

### Returning JSON

```javascript
app.get("/user", (req, res) => {
  res.json({
    id: 1,
    name: "Alice",
  });
});
```

Express automatically sets:

```http
Content-Type: application/json; charset=utf-8
```

---

### Returning HTML

```javascript
app.get("/", (req, res) => {
  res.send("<h1>Welcome</h1>");
});
```

Express sets:

```http
Content-Type: text/html; charset=utf-8
```

---

### Returning Plain Text

```javascript
app.get("/status", (req, res) => {
  res.type("text/plain");
  res.send("Server is running");
});
```

Response:

```http
Content-Type: text/plain; charset=utf-8
```

---

# Setting Content-Type Manually

You can explicitly set the header:

```javascript
app.get("/data", (req, res) => {
  res.set("Content-Type", "application/json");
  res.send(JSON.stringify({ success: true }));
});
```

Or use the helper method:

```javascript
res.type("application/json");
```

or

```javascript
res.type("json");
```

---

# Other Common HTTP Response Headers

Besides `Content-Type`, response headers provide additional metadata:

| Header                                     | Purpose                                                                             |
| ------------------------------------------ | ----------------------------------------------------------------------------------- |
| `Content-Type`                             | Type of response body                                                               |
| `Content-Length`                           | Size of the response body                                                           |
| `Cache-Control`                            | Controls client and proxy caching                                                   |
| `ETag`                                     | Supports conditional requests and caching                                           |
| `Set-Cookie`                               | Sends cookies to the client                                                         |
| `Location`                                 | Redirect target or URI of a newly created resource                                  |
| `Authorization` (less common in responses) | Authentication information in specific protocols                                    |
| `Access-Control-Allow-Origin`              | Controls Cross-Origin Resource Sharing (CORS)                                       |
| `Content-Encoding`                         | Compression used (e.g., `gzip`, `br`)                                               |
| `Content-Disposition`                      | Indicates whether content should be displayed inline or downloaded as an attachment |
| `X-Content-Type-Options`                   | Prevents MIME type sniffing (`nosniff`)                                             |

---

# How Express Handles Content-Type Automatically

Express intelligently sets the `Content-Type` based on the response method:

```javascript
res.json({ name: "John" });
```

→ `application/json`

```javascript
res.send("<h1>Hello</h1>");
```

→ `text/html`

```javascript
res.send("Hello");
```

→ `text/html` (for string responses)

```javascript
res.send(Buffer.from("abc"));
```

→ `application/octet-stream`

This automatic behavior reduces the need to manually manage MIME types in most cases.

---

# Security Considerations

Correct `Content-Type` values are important for security. Browsers may attempt **MIME sniffing** if the header is missing or incorrect, which can lead to security issues such as unintended script execution.

A common best practice is to send:

```http
X-Content-Type-Options: nosniff
```

This instructs browsers to trust the declared `Content-Type` and not guess the content type.

---

# Common Interview Pitfalls

- Forgetting that `Content-Type` describes the **response body**, not the HTTP method.
- Confusing the request's `Content-Type` (which tells the server what format the client is sending) with the response's `Content-Type` (which tells the client what format it is receiving).
- Manually setting `Content-Type` when using `res.json()` or `res.send()`, even though Express usually handles it automatically.
- Returning JSON with an incorrect `Content-Type` such as `text/plain`, which can cause API clients to parse the response incorrectly.

---

# Best Practices

- Use `res.json()` for JSON APIs so Express automatically sets the correct `Content-Type`.
- Let Express infer the header where possible instead of setting it manually.
- Set the appropriate `Content-Type` when serving custom file types or raw data.
- Include security headers like `X-Content-Type-Options: nosniff` to prevent MIME type sniffing.
- Ensure the declared `Content-Type` accurately matches the response body to avoid client-side parsing issues.

---

## Interview Summary

**`Content-Type` is an HTTP response header that tells the client the format of the response body (e.g., JSON, HTML, or an image). This allows browsers and API clients to correctly parse and render the content. In Express.js, methods like `res.json()`, `res.send()`, and `res.type()` automatically or explicitly set the appropriate `Content-Type`, making response handling simpler and more reliable.**

## Question 2. How do you prevent accidental exposure of sensitive configuration values?

## Question 3. What is the role of `req.headers` in Express.js?

## Question 4. How do you make middleware run globally for all routes?

## Question 5. What is API testing and why is it important for Express.js applications?

## Question 6. How do you dynamically register routes in Express.js?

## Question 7. What are middleware factories in Express.js?

## Question 8. How do higher-order middleware functions work?

## Question 9. How would you implement request sanitization in Express.js?

## Question 10. What is input canonicalization and why is it important?

## Question 11. How do you prevent parameter pollution attacks in Express.js APIs?

## Question 12. What are common security headers recommended for APIs?

## Question 13. How do you securely handle API secrets in CI/CD pipelines?

## Question 14. What is the purpose of nonce values in web security?

## Question 15. How do you implement API usage quotas for customers?

## Question 16. What are the trade-offs between API keys and OAuth authentication?

## Question 17. How do OAuth access tokens differ from refresh tokens?

## Question 18. What is the purpose of scopes in OAuth-based APIs?

## Question 19. How do you implement tenant-aware authorization in Express.js?

## Question 20. What are the risks of storing JWTs in browser local storage?
