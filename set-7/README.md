# Set 7

| S.No. | Question                                                                                                                                                                |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you restart an Express.js server automatically during development?](#question-1-how-do-you-restart-an-expressjs-server-automatically-during-development)        |
| 2.    | [What is Nodemon and why is it useful?](#question-2-what-is-nodemon-and-why-is-it-useful)                                                                               |
| 3.    | [What is the purpose of `.env` files in backend applications?](#question-3-what-is-the-purpose-of-env-files-in-backend-applications)                                    |
| 4.    | [How do you read environment variables in Node.js?](#question-4-how-do-you-read-environment-variables-in-nodejs)                                                        |
| 5.    | [What is the difference between synchronous and asynchronous code in Node.js?](#question-5-what-is-the-difference-between-synchronous-and-asynchronous-code-in-nodejs)  |
| 6.    | [How does Express.js match incoming requests to routes internally?](#question-6-how-does-expressjs-match-incoming-requests-to-routes-internally)                        |
| 7.    | [What are dynamic middleware patterns in Express.js?](#question-7-what-are-dynamic-middleware-patterns-in-expressjs)                                                    |
| 8.    | [How would you create reusable middleware across multiple services?](#question-8-how-would-you-create-reusable-middleware-across-multiple-services)                     |
| 9.    | [How do you implement request validation middleware using schemas?](#question-9-how-do-you-implement-request-validation-middleware-using-schemas)                       |
| 10.   | [What is the purpose of dependency injection in Express.js applications?](#question-10-what-is-the-purpose-of-dependency-injection-in-expressjs-applications)           |
| 11.   | [How do you separate controller, service, and repository layers in Express.js?](#question-11-how-do-you-separate-controller-service-and-repository-layers-in-expressjs) |
| 12.   | [What are fat controllers and why are they problematic?](#question-12-what-are-fat-controllers-and-why-are-they-problematic)                                            |
| 13.   | [How do you implement API documentation for Express.js services?](#question-13-how-do-you-implement-api-documentation-for-expressjs-services)                           |
| 14.   | [What are the benefits of OpenAPI or Swagger in Express.js projects?](#question-14-what-are-the-benefits-of-openapi-or-swagger-in-expressjs-projects)                   |
| 15.   | [How do you implement request logging with correlation IDs?](#question-15-how-do-you-implement-request-logging-with-correlation-ids)                                    |
| 16.   | [How do you propagate request context across async operations?](#question-16-how-do-you-propagate-request-context-across-async-operations)                              |
| 17.   | [What is the difference between optimistic and pessimistic locking in APIs?](#question-17-what-is-the-difference-between-optimistic-and-pessimistic-locking-in-apis)    |
| 18.   | [How do you handle database transactions in Express.js applications?](#question-18-how-do-you-handle-database-transactions-in-expressjs-applications)                   |
| 19.   | [How would you implement soft deletes in REST APIs?](#question-19-how-would-you-implement-soft-deletes-in-rest-apis)                                                    |
| 20.   | [What are common REST API anti-patterns in Express.js projects?](#question-20-what-are-common-rest-api-anti-patterns-in-expressjs-projects)                             |

## Question 1. How do you restart an Express.js server automatically during development?

### Direct Answer

During development, you can automatically restart an Express.js server whenever source files change by using tools like **`nodemon`** (most common), **`node --watch`** (built into modern Node.js), or alternatives like **`tsx`**. These tools watch your project files and restart the server automatically, eliminating the need to manually stop and start the application after every code change.

---

## Why Automatic Restart is Needed

Normally, when you start an Express application:

```bash
node app.js
```

or

```bash
node server.js
```

Node.js loads the application into memory. If you modify any source file, the running process does **not** detect those changes. You must manually restart the server.

Automatic restart tools solve this by:

- Watching project files for changes
- Stopping the current Node.js process
- Starting a new process with the updated code

This significantly improves the development workflow.

---

# 1. Using Nodemon (Most Popular)

### Install

Local installation (recommended):

```bash
npm install --save-dev nodemon
```

Or globally:

```bash
npm install -g nodemon
```

---

### Add a Script

In `package.json`:

```json
{
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js"
  }
}
```

Run:

```bash
npm run dev
```

Output:

```
[nodemon] starting `node app.js`
Server running on port 3000
```

---

### When You Save a File

```
Saving app.js...
```

Nodemon automatically displays:

```
[nodemon] restarting due to changes...
[nodemon] starting `node app.js`
```

No manual restart is needed.

---

# 2. Using Node.js Built-in Watch Mode

Modern versions of Node.js include a built-in watch mode.

Run:

```bash
node --watch app.js
```

Or in `package.json`:

```json
{
  "scripts": {
    "dev": "node --watch app.js"
  }
}
```

Advantages:

- No extra dependency
- Lightweight
- Officially supported by Node.js

Limitations compared to `nodemon`:

- Fewer configuration options
- Less mature feature set
- Doesn't provide some advanced ignore/watch behaviors that `nodemon` supports

---

# 3. Example Express Server

```javascript
const express = require("express");

const app = express();

app.get("/", (req, res) => {
  res.send("Hello World");
});

app.listen(3000, () => {
  console.log("Server running...");
});
```

Start with:

```bash
nodemon app.js
```

Now modifying:

```javascript
res.send("Updated!");
```

and saving the file immediately restarts the server.

---

# 4. Configure Nodemon

You can customize its behavior with a `nodemon.json` file:

```json
{
  "watch": ["src"],
  "ext": "js,json",
  "ignore": ["node_modules", "logs"],
  "delay": "500"
}
```

Options:

- `watch` – directories to monitor
- `ext` – file extensions to watch
- `ignore` – files/directories to exclude
- `delay` – wait before restarting

---

# 5. Monitor Specific Files

Example:

```bash
nodemon --watch routes --watch controllers app.js
```

Only changes in those folders trigger a restart.

---

# 6. Ignore Files

Example:

```bash
nodemon --ignore uploads --ignore logs app.js
```

Useful when log files or uploaded files change frequently.

---

# 7. Using Environment Variables

You can combine `nodemon` with environment variable tools.

Example (cross-platform):

```json
{
  "scripts": {
    "dev": "cross-env NODE_ENV=development nodemon app.js"
  }
}
```

---

# 8. Development vs Production

Automatic restart tools are intended for **development only**.

Development:

```bash
npm run dev
```

Production:

```bash
node app.js
```

or use a production process manager such as PM2, which is designed for running Node.js applications reliably in production with features like automatic recovery, clustering, and monitoring.

---

# Common Interview Follow-up

### Why shouldn't `nodemon` be used in production?

Because it is designed for developer convenience rather than production reliability. It continuously watches the file system, which adds unnecessary overhead and doesn't provide production-oriented features like load balancing, clustering, graceful reloads, or robust process management. Production environments are better served by process managers like PM2 or by container orchestration platforms.

---

# Best Practices

- Install `nodemon` as a **development dependency** (`devDependencies`).
- Add a dedicated `dev` script in `package.json`.
- Use **`node --watch`** if you prefer a dependency-free solution and its feature set is sufficient.
- Exclude directories such as `node_modules`, `logs`, and `uploads` from file watching to avoid unnecessary restarts.
- Keep separate scripts for development and production.
- Use a production-grade process manager (e.g., PM2) instead of development watch tools in production.

---

## Interview Summary

> To automatically restart an Express.js server during development, you typically use **`nodemon`**, which watches your project files and restarts the Node.js process whenever changes are detected. Modern Node.js also offers a built-in **`node --watch`** mode that provides basic restart functionality without extra dependencies. These tools improve developer productivity by eliminating manual restarts after code changes, while production environments should use dedicated process managers instead of development watch tools.

## Question 2. What is Nodemon and why is it useful?

## Question 3. What is the purpose of `.env` files in backend applications?

## Question 4. How do you read environment variables in Node.js?

## Question 5. What is the difference between synchronous and asynchronous code in Node.js?

## Question 6. How does Express.js match incoming requests to routes internally?

## Question 7. What are dynamic middleware patterns in Express.js?

## Question 8. How would you create reusable middleware across multiple services?

## Question 9. How do you implement request validation middleware using schemas?

## Question 10. What is the purpose of dependency injection in Express.js applications?

## Question 11. How do you separate controller, service, and repository layers in Express.js?

## Question 12. What are fat controllers and why are they problematic?

## Question 13. How do you implement API documentation for Express.js services?

## Question 14. What are the benefits of OpenAPI or Swagger in Express.js projects?

## Question 15. How do you implement request logging with correlation IDs?

## Question 16. How do you propagate request context across async operations?

## Question 17. What is the difference between optimistic and pessimistic locking in APIs?

## Question 18. How do you handle database transactions in Express.js applications?

## Question 19. How would you implement soft deletes in REST APIs?

## Question 20. What are common REST API anti-patterns in Express.js projects?
