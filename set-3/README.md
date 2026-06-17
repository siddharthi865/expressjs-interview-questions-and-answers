# Set 3

| S.No. | Question                                                                                                                                                                                  |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is the purpose of session middleware?](#question-1-what-is-the-purpose-of-session-middleware)                                                                                       |
| 2.    | [What are the differences between session-based authentication and JWT authentication?](#question-2-what-are-the-differences-between-session-based-authentication-and-jwt-authentication) |
| 3.    | [How do you validate request payloads in Express.js?](#question-3-how-do-you-validate-request-payloads-in-expressjs)                                                                      |
| 4.    | [How would you prevent invalid data from reaching controllers?](#question-4-how-would-you-prevent-invalid-data-from-reaching-controllers)                                                 |
| 5.    | [What is the purpose of environment variables in Express.js applications?](#question-5-what-is-the-purpose-of-environment-variables-in-expressjs-applications)                            |
| 6.    | [How do you implement centralized logging in Express.js?](#question-6-how-do-you-implement-centralized-logging-in-expressjs)                                                              |
| 7.    | [How do you implement request tracing for debugging distributed systems?](#question-7-how-do-you-implement-request-tracing-for-debugging-distributed-systems)                             |
| 8.    | [How can you upload files in Express.js?](#question-8-how-can-you-upload-files-in-expressjs)                                                                                              |
| 9.    | [What are the risks of unrestricted file uploads?](#question-9-what-are-the-risks-of-unrestricted-file-uploads)                                                                           |
| 10.   | [How do you implement rate limiting in Express.js?](#question-10-how-do-you-implement-rate-limiting-in-expressjs)                                                                         |
| 11.   | [What is the purpose of Helmet middleware?](#question-11-what-is-the-purpose-of-helmet-middleware)                                                                                        |
| 12.   | [How would you secure sensitive API endpoints?](#question-12-how-would-you-secure-sensitive-api-endpoints)                                                                                |
| 13.   | [What are common causes of memory leaks in Express.js applications?](#question-13-what-are-common-causes-of-memory-leaks-in-expressjs-applications)                                       |
| 14.   | [How can you improve API response times in Express.js?](#question-14-how-can-you-improve-api-response-times-in-expressjs)                                                                 |
| 15.   | [What is compression middleware and when should you use it?](#question-15-what-is-compression-middleware-and-when-should-you-use-it)                                                      |
| 16.   | [How do you implement API versioning in Express.js?](#question-16-how-do-you-implement-api-versioning-in-expressjs)                                                                       |
| 17.   | [What are the trade-offs between URL versioning and header-based versioning?](#question-17-what-are-the-trade-offs-between-url-versioning-and-header-based-versioning)                    |
| 18.   | [How do you handle pagination in REST APIs built with Express.js?](#question-18-how-do-you-handle-pagination-in-rest-apis-built-with-expressjs)                                           |
| 19.   | [What strategies can be used to optimize database-heavy Express.js endpoints?](#question-19-what-strategies-can-be-used-to-optimize-database-heavy-expressjs-endpoints)                   |
| 20.   | [How would you implement request timeout handling in Express.js?](#question-20-how-would-you-implement-request-timeout-handling-in-expressjs)                                             |

## Question 1. What is the purpose of session middleware?

### ✅ Concise Answer

Session middleware in Express.js is used to **persist user-specific data across multiple HTTP requests** by storing session information on the server and associating it with a client via a session ID (usually stored in a cookie).

---

## 🧠 Interview-Ready Explanation

HTTP is a **stateless protocol**, meaning each request is independent and does not remember previous interactions. Session middleware solves this limitation by enabling **stateful communication** between the client and server.

In Express.js, session middleware (commonly using `express-session`) creates and manages a **session object per user**, allowing you to store data like:

- Authentication state (logged in / logged out)
- User preferences
- Shopping cart data
- Temporary workflow data (e.g., multi-step forms)

---

## ⚙️ How Session Middleware Works

1. **Client makes a request**
2. Server checks if a session ID exists in cookies
3. If not, server creates a new session and sends a session ID cookie
4. Session data is stored on the server (memory, Redis, DB, etc.)
5. On subsequent requests, the session ID links the client to stored data

---

## 🧾 Example using `express-session`

```javascript
const express = require("express");
const session = require("express-session");

const app = express();

app.use(
  session({
    secret: "mySecretKey",
    resave: false,
    saveUninitialized: false,
    cookie: { secure: false }, // true in HTTPS
  }),
);

app.get("/login", (req, res) => {
  req.session.user = { id: 1, name: "John" };
  res.send("User logged in");
});

app.get("/dashboard", (req, res) => {
  if (req.session.user) {
    res.send(`Welcome ${req.session.user.name}`);
  } else {
    res.send("Not authenticated");
  }
});
```

---

## 🔐 Why Sessions Are Useful

### 1. Authentication Management

Keeps track of logged-in users without requiring credentials on every request.

### 2. Temporary User State

Useful for workflows like checkout carts or form wizards.

### 3. Security Control

Sensitive data stays on the server instead of the client.

---

## ⚖️ Session vs JWT (Important Interview Comparison)

| Feature     | Session Middleware         | JWT                       |
| ----------- | -------------------------- | ------------------------- |
| Storage     | Server-side                | Client-side               |
| Scalability | Needs shared store (Redis) | Highly scalable           |
| Revocation  | Easy (destroy session)     | Hard (token invalidation) |
| Security    | Safer for sensitive data   | Depends on client storage |
| Best for    | Traditional web apps       | APIs, microservices       |

---

## ⚠️ Common Pitfalls

- ❌ Using in-memory sessions in production → breaks on server restart or scaling
- ❌ Not using secure cookies in HTTPS environments
- ❌ Storing large objects in session → memory overhead
- ❌ Not using session store like Redis in distributed systems

---

## 🚀 Best Practices

- Use **Redis or external session store** for scalability
- Enable `secure: true` and `httpOnly: true` in production
- Keep session data minimal (store IDs, not full objects)
- Set proper session expiration (`maxAge`)
- Regenerate session ID after login (prevents session fixation)

## Question 2. What are the differences between session-based authentication and JWT authentication?

## Question 3. How do you validate request payloads in Express.js?

## Question 4. How would you prevent invalid data from reaching controllers?

## Question 5. What is the purpose of environment variables in Express.js applications?

## Question 6. How do you implement centralized logging in Express.js?

## Question 7. How do you implement request tracing for debugging distributed systems?

## Question 8. How can you upload files in Express.js?

## Question 9. What are the risks of unrestricted file uploads?

## Question 10. How do you implement rate limiting in Express.js?

## Question 11. What is the purpose of Helmet middleware?

## Question 12. How would you secure sensitive API endpoints?

## Question 13. What are common causes of memory leaks in Express.js applications?

## Question 14. How can you improve API response times in Express.js?

## Question 15. What is compression middleware and when should you use it?

## Question 16. How do you implement API versioning in Express.js?

## Question 17. What are the trade-offs between URL versioning and header-based versioning?

## Question 18. How do you handle pagination in REST APIs built with Express.js?

## Question 19. What strategies can be used to optimize database-heavy Express.js endpoints?

## Question 20. How would you implement request timeout handling in Express.js?
