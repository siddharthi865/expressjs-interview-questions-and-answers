# Set 1

| S.No. | Question                                                                                                                                                                                        |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is Express.js and why is it commonly used with Node.js?](#question-1-what-is-expressjs-and-why-is-it-commonly-used-with-nodejs)                                                           |
| 2.    | [What are the main differences between Node.js and Express.js?](#question-2-what-are-the-main-differences-between-nodejs-and-expressjs)                                                         |
| 3.    | [How do you create a basic Express server?](#question-3-how-do-you-create-a-basic-express-server)                                                                                               |
| 4.    | [What is the purpose of `app.listen()` in Express.js?](#question-4-what-is-the-purpose-of-applisten-in-expressjs)                                                                               |
| 5.    | [What is middleware in Express.js?](#question-5-what-is-middleware-in-expressjs)                                                                                                                |
| 6.    | [What is the difference between application-level middleware and router-level middleware?](#question-6-what-is-the-difference-between-application-level-middleware-and-router-level-middleware) |
| 7.    | [How does routing work in Express.js?](#question-7-how-does-routing-work-in-expressjs)                                                                                                          |
| 8.    | [What is the purpose of `req` and `res` objects in Express.js?](#question-8-what-is-the-purpose-of-req-and-res-objects-in-expressjs)                                                            |
| 9.    | [What does `next()` do in middleware?](#question-9-what-does-next-do-in-middleware)                                                                                                             |
| 10.   | [What happens if `next()` is not called inside middleware?](#question-10-what-happens-if-next-is-not-called-inside-middleware)                                                                  |
| 11.   | [How do you handle GET requests in Express.js?](#question-11-how-do-you-handle-get-requests-in-expressjs)                                                                                       |
| 12.   | [How do you handle POST requests in Express.js?](#question-12-how-do-you-handle-post-requests-in-expressjs)                                                                                     |
| 13.   | [What is the difference between `req.params`, `req.query`, and `req.body`?](#question-13-what-is-the-difference-between-reqparams-reqquery-and-reqbody)                                         |
| 14.   | [How do you serve static files in Express.js?](#question-14-how-do-you-serve-static-files-in-expressjs)                                                                                         |
| 15.   | [What is the role of `express.json()` middleware?](#question-15-what-is-the-role-of-expressjson-middleware)                                                                                     |
| 16.   | [What is the purpose of `express.urlencoded()`?](#question-16-what-is-the-purpose-of-expressurlencoded)                                                                                         |
| 17.   | [How do you create route parameters in Express.js?](#question-17-how-do-you-create-route-parameters-in-expressjs)                                                                               |
| 18.   | [What is the difference between `app.use()` and `app.get()`?](#question-18-what-is-the-difference-between-appuse-and-appget)                                                                    |
| 19.   | [How can you organize routes in separate files?](#question-19-how-can-you-organize-routes-in-separate-files)                                                                                    |
| 20.   | [What are the common HTTP methods used in Express.js applications?](#question-20-what-are-the-common-http-methods-used-in-expressjs-applications)                                               |

## Question 1. What is Express.js and why is it commonly used with Node.js?

## What is Express.js?

Express.js is a minimal and flexible web application framework built on top of Node.js. It provides a thin layer of features for building web servers and APIs, making backend development in Node.js much easier and more structured.

At its core, Express.js simplifies tasks such as:

- Routing HTTP requests
- Handling middleware
- Processing requests and responses
- Building REST APIs
- Serving static files
- Error handling
- Managing sessions and authentication integrations

---

# Why is Express.js commonly used with Node.js?

Node.js provides the runtime and low-level HTTP capabilities, but building applications directly with Node’s native `http` module can become verbose and difficult to maintain.

Express.js solves this by adding a clean and developer-friendly abstraction layer.

---

# Without Express.js (Pure Node.js)

Using only the built-in `http` module:

```js
const http = require("http");

const server = http.createServer((req, res) => {
  if (req.url === "/" && req.method === "GET") {
    res.writeHead(200, { "Content-Type": "text/plain" });
    res.end("Hello World");
  }
});

server.listen(3000);
```

This works, but as the application grows:

- Routing becomes messy
- Middleware handling is manual
- Error handling is repetitive
- Request parsing is cumbersome

---

# With Express.js

```js
const express = require("express");

const app = express();

app.get("/", (req, res) => {
  res.send("Hello World");
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

This is:

- Cleaner
- More readable
- Easier to scale
- Easier to maintain

---

# Key Features of Express.js

## 1. Routing System

Express provides powerful routing methods:

```js
app.get("/users", getUsers);
app.post("/users", createUser);
app.put("/users/:id", updateUser);
app.delete("/users/:id", deleteUser);
```

This makes REST API development straightforward.

---

## 2. Middleware Architecture

Middleware is one of Express’s biggest strengths.

Middleware functions can:

- Execute logic before request completion
- Modify request/response objects
- Authenticate users
- Validate input
- Log requests
- Handle errors

Example:

```js
app.use(express.json());

app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});
```

---

## 3. Simplified Request & Response Handling

Express enhances Node’s native objects:

```js
req.params;
req.query;
req.body;
req.headers;
```

Response helpers:

```js
res.json();
res.send();
res.status();
res.redirect();
```

Example:

```js
app.get("/users/:id", (req, res) => {
  res.json({
    userId: req.params.id,
  });
});
```

---

## 4. REST API Development

Express is widely used for building RESTful APIs because it:

- Is lightweight
- Has minimal boilerplate
- Supports JSON naturally
- Integrates easily with databases

Typical stack:

- Express.js
- MongoDB / PostgreSQL / MySQL
- JWT Authentication
- Validation libraries like Joi/Zod

---

## 5. Huge Ecosystem

Express has a massive ecosystem of middleware and community packages:

Examples:

- `cors`
- `helmet`
- `morgan`
- `passport`
- `jsonwebtoken`
- `express-validator`

This accelerates development significantly.

---

# Why Express Fits Node.js So Well

## 1. Same Language Everywhere

Both frontend and backend can use JavaScript.

This enables:

- Faster development
- Shared validation logic
- Easier onboarding

---

## 2. Non-Blocking & Event-Driven

Node.js uses an asynchronous, event-driven architecture.

Express naturally works with async operations:

```js
app.get("/users", async (req, res) => {
  const users = await User.find();
  res.json(users);
});
```

This is ideal for:

- APIs
- Real-time apps
- Streaming
- Microservices

---

## 3. Lightweight & Unopinionated

Express does not force:

- Folder structures
- ORM choices
- Authentication methods
- Architecture patterns

Developers can structure applications however they want.

This flexibility is a major reason for its popularity.

---

# Common Use Cases

Express.js is commonly used for:

- REST APIs
- Microservices
- Backend for SPAs (React/Angular/Vue)
- Authentication servers
- Real-time applications (with Socket.IO)
- GraphQL servers
- Server-side rendered apps

---

# Common Interview Follow-Up: “Is Express.js still relevant?”

Yes. Even with newer frameworks like:

- NestJS
- Fastify
- Koa

Express remains:

- Extremely popular
- Stable
- Production-proven
- Widely adopted in industry

Many modern frameworks are also inspired by Express concepts.

---

# Best Practices When Using Express.js

## Use Middleware Properly

Keep middleware modular and reusable.

## Handle Errors Centrally

```js
app.use((err, req, res, next) => {
  res.status(500).json({
    message: err.message,
  });
});
```

## Use Async Error Handling

Avoid unhandled promise rejections.

## Validate Incoming Data

Use libraries like:

- Joi
- Zod
- express-validator

## Secure Applications

Use:

- Helmet
- CORS configuration
- Rate limiting
- Input sanitization

---

# Summary

Express.js is a lightweight web framework built on top of Node.js that simplifies backend development by providing:

- Routing
- Middleware support
- Request/response utilities
- API development tools

It is commonly used with Node.js because it:

- Reduces boilerplate
- Improves scalability
- Supports asynchronous programming naturally
- Makes REST API development fast and maintainable
- Has a massive ecosystem and community support

In interviews, a strong answer should emphasize:

- Express extends Node’s HTTP capabilities
- Middleware is a core concept
- Express is minimal yet flexible
- It is widely used for scalable API and backend development

## Question 2. What are the main differences between Node.js and Express.js?

## Question 3. How do you create a basic Express server?

## Question 4. What is the purpose of `app.listen()` in Express.js?

## Question 5. What is middleware in Express.js?

## Question 6. What is the difference between application-level middleware and router-level middleware?

## Question 7. How does routing work in Express.js?

## Question 8. What is the purpose of `req` and `res` objects in Express.js?

## Question 9. What does `next()` do in middleware?

## Question 10. What happens if `next()` is not called inside middleware?

## Question 11. How do you handle GET requests in Express.js?

## Question 12. How do you handle POST requests in Express.js?

## Question 13. What is the difference between `req.params`, `req.query`, and `req.body`?

## Question 14. How do you serve static files in Express.js?

## Question 15. What is the role of `express.json()` middleware?

## Question 16. What is the purpose of `express.urlencoded()`?

## Question 17. How do you create route parameters in Express.js?

## Question 18. What is the difference between `app.use()` and `app.get()`?

## Question 19. How can you organize routes in separate files?

## Question 20. What are the common HTTP methods used in Express.js applications?
