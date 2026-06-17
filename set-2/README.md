# Set 2

| S.No. | Question                                                                                                                                                                  |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you send JSON responses in Express.js?](#question-1-how-do-you-send-json-responses-in-expressjs)                                                                  |
| 2.    | [What is the purpose of status codes in API responses?](#question-2-what-is-the-purpose-of-status-codes-in-api-responses)                                                 |
| 3.    | [How do you set custom headers in Express.js?](#question-3-how-do-you-set-custom-headers-in-expressjs)                                                                    |
| 4.    | [What is the difference between `res.send()` and `res.json()`?](#question-4-what-is-the-difference-between-ressend-and-resjson)                                           |
| 5.    | [How can you redirect users to another route in Express.js?](#question-5-how-can-you-redirect-users-to-another-route-in-expressjs)                                        |
| 6.    | [How does Express.js internally process middleware execution order?](#question-6-how-does-expressjs-internally-process-middleware-execution-order)                        |
| 7.    | [How do you implement custom middleware in Express.js?](#question-7-how-do-you-implement-custom-middleware-in-expressjs)                                                  |
| 8.    | [What is error-handling middleware in Express.js?](#question-8-what-is-error-handling-middleware-in-expressjs)                                                            |
| 9.    | [Why must error middleware have four arguments?](#question-9-why-must-error-middleware-have-four-arguments)                                                               |
| 10.   | [How do you handle async errors in Express.js?](#question-10-how-do-you-handle-async-errors-in-expressjs)                                                                 |
| 11.   | [What problems can arise from unhandled promise rejections in Express.js APIs?](#question-11-what-problems-can-arise-from-unhandled-promise-rejections-in-expressjs-apis) |
| 12.   | [How would you structure a large-scale Express.js project?](#question-12-how-would-you-structure-a-large-scale-expressjs-project)                                         |
| 13.   | [What is the purpose of `express.Router()`?](#question-13-what-is-the-purpose-of-expressrouter)                                                                           |
| 14.   | [How do nested routers work in Express.js?](#question-14-how-do-nested-routers-work-in-expressjs)                                                                         |
| 15.   | [What are route handlers and route chaining in Express.js?](#question-15-what-are-route-handlers-and-route-chaining-in-expressjs)                                         |
| 16.   | [How do you implement authentication middleware?](#question-16-how-do-you-implement-authentication-middleware)                                                            |
| 17.   | [How would you implement role-based authorization in Express.js?](#question-17-how-would-you-implement-role-based-authorization-in-expressjs)                             |
| 18.   | [What is CORS and how do you configure it in Express.js?](#question-18-what-is-cors-and-how-do-you-configure-it-in-expressjs)                                             |
| 19.   | [What security risks exist when enabling CORS with wildcard origins?](#question-19-what-security-risks-exist-when-enabling-cors-with-wildcard-origins)                    |
| 20.   | [How do cookies work in Express.js?](#question-20-how-do-cookies-work-in-expressjs)                                                                                       |

## Question 1. How do you send JSON responses in Express.js?

**Short answer:**
In Express.js, you send JSON responses using the `res.json()` method, which automatically serializes a JavaScript object into JSON and sets the correct `Content-Type` header.

---

## Detailed Interview-Style Explanation

In Express.js, handling HTTP responses is done through the `res` (response) object. When building REST APIs, JSON is the standard format for communication between client and server. Express provides a built-in helper method specifically for this: **`res.json()`**.

### 1. Using `res.json()` (Recommended Approach)

This is the most common and best practice way.

```javascript
const express = require("express");
const app = express();

app.get("/user", (req, res) => {
  res.json({
    id: 1,
    name: "John Doe",
    role: "developer",
  });
});
```

### What happens internally:

- The object is converted to a JSON string using `JSON.stringify()`
- The `Content-Type` header is automatically set to:

  ```
  application/json
  ```

- The response is sent to the client

---

### 2. Difference Between `res.json()` and `res.send()`

While both can send JSON, there are subtle differences:

#### `res.json()`

- Explicitly intended for JSON responses
- Always sets correct headers
- Converts non-object inputs safely into JSON

#### `res.send()`

- More generic (can send strings, buffers, objects)
- If you pass an object, Express internally converts it to JSON, but behavior is less explicit

```javascript
res.send({ message: "Hello" }); // works, but less explicit
res.json({ message: "Hello" }); // preferred
```

👉 **Best practice:** Always use `res.json()` for API responses.

---

### 3. Sending Status Codes with JSON

You can chain methods for cleaner API responses:

```javascript
app.post("/login", (req, res) => {
  res.status(200).json({
    success: true,
    message: "Login successful",
  });
});
```

Or for errors:

```javascript
res.status(400).json({
  success: false,
  error: "Invalid credentials",
});
```

---

### 4. Returning Arrays or Complex Data

Express handles arrays and nested objects seamlessly:

```javascript
app.get("/products", (req, res) => {
  res.json([
    { id: 1, name: "Laptop" },
    { id: 2, name: "Phone" },
  ]);
});
```

---

### 5. Common Pitfalls

#### ❌ Not using proper status codes

Always pair JSON responses with meaningful HTTP status codes.

#### ❌ Sending multiple responses

You can only send one response per request. Calling `res.json()` twice will throw an error.

#### ❌ Circular objects

Trying to send circular references will crash serialization.

---

### 6. Best Practices (Interview-Ready Points)

- Always use `res.json()` for APIs
- Structure responses consistently (e.g., `{ success, data, message }`)
- Use appropriate HTTP status codes
- Avoid leaking sensitive information in JSON responses
- Centralize error handling using middleware for consistent JSON error format

Example standardized response:

```javascript
res.json({
  success: true,
  data: user,
  message: "User fetched successfully",
});
```

## Question 2. What is the purpose of status codes in API responses?

## Question 3. How do you set custom headers in Express.js?

## Question 4. What is the difference between `res.send()` and `res.json()`?

## Question 5. How can you redirect users to another route in Express.js?

## Question 6. How does Express.js internally process middleware execution order?

## Question 7. How do you implement custom middleware in Express.js?

## Question 8. What is error-handling middleware in Express.js?

## Question 9. Why must error middleware have four arguments?

## Question 10. How do you handle async errors in Express.js?

## Question 11. What problems can arise from unhandled promise rejections in Express.js APIs?

## Question 12. How would you structure a large-scale Express.js project?

## Question 13. What is the purpose of `express.Router()`?

## Question 14. How do nested routers work in Express.js?

## Question 15. What are route handlers and route chaining in Express.js?

## Question 16. How do you implement authentication middleware?

## Question 17. How would you implement role-based authorization in Express.js?

## Question 18. What is CORS and how do you configure it in Express.js?

## Question 19. What security risks exist when enabling CORS with wildcard origins?

## Question 20. How do cookies work in Express.js?
