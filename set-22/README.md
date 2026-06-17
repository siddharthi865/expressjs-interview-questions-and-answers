# Set 22

| S.No. | Question                                                                                                                                                             |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you test a simple Express.js route manually?](#question-1-how-do-you-test-a-simple-expressjs-route-manually)                                                 |
| 2.    | [What is API mocking and why is it useful?](#question-2-what-is-api-mocking-and-why-is-it-useful)                                                                    |
| 3.    | [How do you configure custom response headers globally?](#question-3-how-do-you-configure-custom-response-headers-globally)                                          |
| 4.    | [What are the differences between APIs and traditional MVC applications?](#question-4-what-are-the-differences-between-apis-and-traditional-mvc-applications)        |
| 5.    | [How do you expose API documentation endpoints in Express.js?](#question-5-how-do-you-expose-api-documentation-endpoints-in-expressjs)                               |
| 6.    | [How do you implement reusable async wrappers for route handlers?](#question-6-how-do-you-implement-reusable-async-wrappers-for-route-handlers)                      |
| 7.    | [What are middleware side effects and why are they dangerous?](#question-7-what-are-middleware-side-effects-and-why-are-they-dangerous)                              |
| 8.    | [How do you avoid hidden dependencies between middleware functions?](#question-8-how-do-you-avoid-hidden-dependencies-between-middleware-functions)                  |
| 9.    | [How would you implement request-scoped services in Express.js?](#question-9-how-would-you-implement-request-scoped-services-in-expressjs)                           |
| 10.   | [What is inversion of control and how does it improve backend architecture?](#question-10-what-is-inversion-of-control-and-how-does-it-improve-backend-architecture) |
| 11.   | [How do you separate business logic from HTTP transport logic?](#question-11-how-do-you-separate-business-logic-from-http-transport-logic)                           |
| 12.   | [What are anti-corruption layers in backend architectures?](#question-12-what-are-anti-corruption-layers-in-backend-architectures)                                   |
| 13.   | [How do you expose consistent API error codes across services?](#question-13-how-do-you-expose-consistent-api-error-codes-across-services)                           |
| 14.   | [What is the purpose of RFC standards in REST API design?](#question-14-what-is-the-purpose-of-rfc-standards-in-rest-api-design)                                     |
| 15.   | [How do you implement conditional requests using `If-None-Match` headers?](#question-15-how-do-you-implement-conditional-requests-using-if-none-match-headers)       |
| 16.   | [What is optimistic caching in APIs?](#question-16-what-is-optimistic-caching-in-apis)                                                                               |
| 17.   | [How do you design APIs for mobile clients with unstable networks?](#question-17-how-do-you-design-apis-for-mobile-clients-with-unstable-networks)                   |
| 18.   | [What are batching endpoints and when should they be used?](#question-18-what-are-batching-endpoints-and-when-should-they-be-used)                                   |
| 19.   | [How do you design APIs for partial failures in batch operations?](#question-19-how-do-you-design-apis-for-partial-failures-in-batch-operations)                     |
| 20.   | [What are the risks of deeply nested REST endpoints?](#question-20-what-are-the-risks-of-deeply-nested-rest-endpoints)                                               |

## Question 1. How do you test a simple Express.js route manually?

## Direct Answer

A simple Express.js route can be tested manually by starting the server and sending HTTP requests using a web browser (for GET requests), tools like **Postman** or **Insomnia**, or command-line tools such as **cURL**. You then verify that the response status, headers, and body match the expected behavior.

---

# Detailed Explanation

Manual testing is the quickest way to verify that an Express.js route behaves correctly before writing automated tests. It helps ensure that routing, middleware, request handling, and responses are functioning as expected.

## Example Express Route

```javascript
const express = require("express");

const app = express();
const PORT = 3000;

app.get("/hello", (req, res) => {
  res.status(200).json({
    message: "Hello, Express!",
  });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

Start the server:

```bash
node app.js
```

Output:

```text
Server running on port 3000
```

---

# Method 1: Test Using a Browser

For simple **GET** routes, open:

```
http://localhost:3000/hello
```

Expected response:

```json
{
  "message": "Hello, Express!"
}
```

This works only for routes that don't require a request body.

---

# Method 2: Test Using cURL

cURL is available on most operating systems.

### GET Request

```bash
curl http://localhost:3000/hello
```

Output:

```json
{ "message": "Hello, Express!" }
```

### View Response Headers

```bash
curl -i http://localhost:3000/hello
```

Example output:

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 28

{"message":"Hello, Express!"}
```

---

# Method 3: Test Using Postman

1. Open Postman.
2. Create a new request.
3. Select **GET**.
4. Enter:

```
http://localhost:3000/hello
```

5. Click **Send**.

Verify:

- Status Code → **200 OK**
- Response Body
- Response Headers
- Response Time

Postman is especially useful for testing:

- POST requests
- PUT requests
- PATCH requests
- DELETE requests
- Authentication
- Cookies
- Custom headers

---

# Method 4: Test POST Routes

Example:

```javascript
app.use(express.json());

app.post("/users", (req, res) => {
  res.status(201).json(req.body);
});
```

Using cURL:

```bash
curl -X POST http://localhost:3000/users \
-H "Content-Type: application/json" \
-d '{"name":"Alice"}'
```

Expected response:

```json
{
  "name": "Alice"
}
```

---

# What Should You Verify?

When manually testing a route, check:

- ✅ Correct HTTP status code (200, 201, 400, 404, etc.)
- ✅ Response body
- ✅ Response headers (e.g., `Content-Type`)
- ✅ Route parameters
- ✅ Query parameters
- ✅ Request body handling
- ✅ Error responses
- ✅ Middleware execution
- ✅ Authentication and authorization (if applicable)

---

# Testing Route Parameters

Example:

```javascript
app.get("/users/:id", (req, res) => {
  res.json({
    id: req.params.id,
  });
});
```

Request:

```
GET /users/42
```

Response:

```json
{
  "id": "42"
}
```

---

# Testing Query Parameters

Example:

```javascript
app.get("/search", (req, res) => {
  res.json(req.query);
});
```

Request:

```
GET /search?name=John&age=30
```

Response:

```json
{
  "name": "John",
  "age": "30"
}
```

---

# Common Manual Testing Tools

| Tool     | Best For                                   |
| -------- | ------------------------------------------ |
| Browser  | Simple GET requests                        |
| cURL     | Quick command-line testing and scripting   |
| Postman  | Full API testing with GUI                  |
| Insomnia | REST and GraphQL API testing               |
| Bruno    | Offline API testing with local collections |
| HTTPie   | Human-friendly command-line HTTP client    |

---

# Common Pitfalls

- Forgetting to start the server.
- Using the wrong HTTP method (e.g., `GET` instead of `POST`).
- Sending JSON without the `Content-Type: application/json` header.
- Omitting `express.json()` middleware when parsing JSON request bodies.
- Testing the wrong port or URL.
- Ignoring HTTP status codes and checking only the response body.
- Not testing error paths (such as invalid input or missing resources).

---

# Best Practices

- Use manual testing during development for quick feedback.
- Test both successful and failure scenarios.
- Verify status codes, headers, and response payloads—not just the body.
- Use API clients like Postman or Insomnia for complex requests involving authentication, headers, or different payload formats.
- Complement manual testing with automated tests using tools like **Supertest**, **Jest**, or **Mocha** to ensure consistent regression testing.

---

## Interview Tip

In an interview, you could answer:

> "To manually test an Express.js route, I start the server and send HTTP requests using a browser for simple GET endpoints, cURL from the command line, or API clients like Postman or Insomnia. I verify the response status code, headers, and body, and I test both successful and error scenarios. While manual testing is useful during development, I typically complement it with automated tests using Supertest and a test runner like Jest to ensure reliability and prevent regressions."

## Question 2. What is API mocking and why is it useful?

## Question 3. How do you configure custom response headers globally?

## Question 4. What are the differences between APIs and traditional MVC applications?

## Question 5. How do you expose API documentation endpoints in Express.js?

## Question 6. How do you implement reusable async wrappers for route handlers?

## Question 7. What are middleware side effects and why are they dangerous?

## Question 8. How do you avoid hidden dependencies between middleware functions?

## Question 9. How would you implement request-scoped services in Express.js?

## Question 10. What is inversion of control and how does it improve backend architecture?

## Question 11. How do you separate business logic from HTTP transport logic?

## Question 12. What are anti-corruption layers in backend architectures?

## Question 13. How do you expose consistent API error codes across services?

## Question 14. What is the purpose of RFC standards in REST API design?

## Question 15. How do you implement conditional requests using `If-None-Match` headers?

## Question 16. What is optimistic caching in APIs?

## Question 17. How do you design APIs for mobile clients with unstable networks?

## Question 18. What are batching endpoints and when should they be used?

## Question 19. How do you design APIs for partial failures in batch operations?

## Question 20. What are the risks of deeply nested REST endpoints?
