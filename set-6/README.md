# Set 6

| S.No. | Question                                                                                                                                                                                            |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is the purpose of `package.json` in an Express.js project?](#question-1-what-is-the-purpose-of-packagejson-in-an-expressjs-project)                                                           |
| 2.    | [How do you install Express.js in a Node.js application?](#question-2-how-do-you-install-expressjs-in-a-nodejs-application)                                                                         |
| 3.    | [What is the difference between Express.js and the built-in Node.js `http` module?](#question-3-what-is-the-difference-between-expressjs-and-the-built-in-nodejs-http-module)                       |
| 4.    | [How do you define multiple route handlers for the same route in Express.js?](#question-4-how-do-you-define-multiple-route-handlers-for-the-same-route-in-expressjs)                                |
| 5.    | [What is the purpose of `req.method` in Express.js?](#question-5-what-is-the-purpose-of-reqmethod-in-expressjs)                                                                                     |
| 6.    | [What does `req.originalUrl` contain?](#question-6-what-does-reqoriginalurl-contain)                                                                                                                |
| 7.    | [How can you access request headers in Express.js?](#question-7-how-can-you-access-request-headers-in-expressjs)                                                                                    |
| 8.    | [How do you send HTML responses from Express.js?](#question-8-how-do-you-send-html-responses-from-expressjs)                                                                                        |
| 9.    | [What is the purpose of template engines in Express.js?](#question-9-what-is-the-purpose-of-template-engines-in-expressjs)                                                                          |
| 10.   | [How do you configure EJS or Pug with Express.js?](#question-10-how-do-you-configure-ejs-or-pug-with-expressjs)                                                                                     |
| 11.   | [What is the difference between server-side rendering and API-only Express.js applications?](#question-11-what-is-the-difference-between-server-side-rendering-and-api-only-expressjs-applications) |
| 12.   | [How do you define wildcard routes in Express.js?](#question-12-how-do-you-define-wildcard-routes-in-expressjs)                                                                                     |
| 13.   | [What are optional route parameters in Express.js?](#question-13-what-are-optional-route-parameters-in-expressjs)                                                                                   |
| 14.   | [How do you chain multiple middleware functions for a route?](#question-14-how-do-you-chain-multiple-middleware-functions-for-a-route)                                                              |
| 15.   | [What is the purpose of `res.status()`?](#question-15-what-is-the-purpose-of-resstatus)                                                                                                             |
| 16.   | [What is the purpose of `res.set()`?](#question-16-what-is-the-purpose-of-resset)                                                                                                                   |
| 17.   | [How do you handle 404 errors in Express.js?](#question-17-how-do-you-handle-404-errors-in-expressjs)                                                                                               |
| 18.   | [What happens if two routes match the same request path?](#question-18-what-happens-if-two-routes-match-the-same-request-path)                                                                      |
| 19.   | [How can you access cookies sent by the client?](#question-19-how-can-you-access-cookies-sent-by-the-client)                                                                                        |
| 20.   | [What is the difference between `npm install --save` and `npm install --save-dev`?](#question-20-what-is-the-difference-between-npm-install---save-and-npm-install---save-dev)                      |

## Question 1. What is the purpose of `package.json` in an Express.js project?

### Direct Answer

`package.json` is the **manifest file** of a Node.js/Express.js project. It stores metadata about the project, manages dependencies, defines scripts, specifies the project's entry point, and contains configuration that npm and other tools use to build, run, and distribute the application.

---

# Detailed Explanation

In an Express.js project, `package.json` is one of the most important files. Whenever you initialize a project using:

```bash
npm init
```

or

```bash
npm init -y
```

npm creates a `package.json` file.

A typical Express project looks like this:

```
express-app/
│
├── node_modules/
├── package.json
├── package-lock.json
├── app.js
├── routes/
├── controllers/
└── public/
```

---

# Main Purposes of `package.json`

## 1. Project Metadata

It describes the project.

```json
{
  "name": "express-api",
  "version": "1.0.0",
  "description": "REST API using Express",
  "author": "John Doe",
  "license": "MIT"
}
```

This information is useful when publishing packages or identifying the project.

---

## 2. Dependency Management

One of its primary purposes is listing project dependencies.

Example:

```json
{
  "dependencies": {
    "express": "^5.1.0",
    "dotenv": "^17.0.0",
    "mongoose": "^8.0.0"
  }
}
```

Then anyone can install all required packages using:

```bash
npm install
```

instead of installing each package individually.

---

## 3. Development Dependencies

Packages needed only during development go under `devDependencies`.

Example:

```json
{
  "devDependencies": {
    "nodemon": "^3.0.0",
    "eslint": "^9.0.0",
    "jest": "^30.0.0"
  }
}
```

Install them using:

```bash
npm install --save-dev nodemon
```

Examples include:

- Nodemon
- ESLint
- Prettier
- Jest
- TypeScript

These are generally not required in production.

---

## 4. Project Scripts

The `scripts` section defines shortcuts for common tasks.

Example:

```json
{
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js",
    "test": "jest"
  }
}
```

Run them with:

```bash
npm start
```

```bash
npm run dev
```

```bash
npm test
```

This standardizes project commands across the team.

---

## 5. Entry Point

The `main` field specifies the application's main file.

```json
{
  "main": "app.js"
}
```

For many Express applications, this is `app.js`, `server.js`, or `index.js`.

---

## 6. Version Management

```json
{
  "version": "1.2.0"
}
```

This follows **Semantic Versioning (SemVer)**:

- Major → Breaking changes
- Minor → New features
- Patch → Bug fixes

Example:

```
1.0.0
1.1.0
1.1.1
2.0.0
```

---

## 7. Engine Requirements

You can specify the supported Node.js version.

```json
{
  "engines": {
    "node": ">=20"
  }
}
```

This helps prevent running the application on unsupported versions.

---

## 8. Module Type

Modern Node.js projects can specify the module system.

CommonJS:

```json
{
  "type": "commonjs"
}
```

ES Modules:

```json
{
  "type": "module"
}
```

With ES Modules:

```javascript
import express from "express";
```

Without it (CommonJS):

```javascript
const express = require("express");
```

---

## 9. Custom Configuration

Many tools store configuration in `package.json`.

Example:

```json
{
  "eslintConfig": {
    "extends": "eslint:recommended"
  }
}
```

Other tools that may use it include:

- Babel
- Prettier
- Jest
- Husky

---

# Example `package.json` for an Express Application

```json
{
  "name": "express-rest-api",
  "version": "1.0.0",
  "description": "Sample Express API",
  "main": "app.js",
  "type": "module",
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^5.1.0",
    "dotenv": "^17.0.0",
    "mongoose": "^8.0.0"
  },
  "devDependencies": {
    "nodemon": "^3.1.0",
    "eslint": "^9.0.0",
    "jest": "^30.0.0"
  },
  "engines": {
    "node": ">=20"
  },
  "license": "MIT"
}
```

---

# Relationship with `package-lock.json`

Interviewers often ask about the difference.

| `package.json`                       | `package-lock.json`                                         |
| ------------------------------------ | ----------------------------------------------------------- |
| Declares project dependencies        | Locks the exact dependency versions installed               |
| Editable by developers               | Automatically generated by npm                              |
| Uses version ranges (e.g., `^5.1.0`) | Stores exact resolved versions and dependency tree          |
| Shared with the project              | Also committed to version control for reproducible installs |

Example:

```json
"express": "^5.1.0"
```

`package-lock.json` might resolve this to:

```
express 5.1.2
```

ensuring every developer and deployment uses the same dependency versions.

---

# Best Practices

- Commit both `package.json` and `package-lock.json` to version control.
- Separate runtime dependencies from development dependencies.
- Use meaningful npm scripts (e.g., `start`, `dev`, `test`, `lint`).
- Keep dependencies up to date and remove unused packages.
- Specify a compatible Node.js version using the `engines` field when appropriate.
- Avoid manually editing `package-lock.json` unless absolutely necessary.

---

# Common Interview Pitfalls

- **Confusing `package.json` with `package-lock.json`:** The former defines dependency requirements; the latter locks exact installed versions.
- **Placing development tools in `dependencies`:** Tools like `nodemon` or `eslint` should typically be in `devDependencies`.
- **Ignoring npm scripts:** Defining consistent scripts improves developer experience and CI/CD workflows.
- **Not understanding `type: "module"`:** This setting changes how Node.js interprets JavaScript modules (`import`/`export` vs. `require`/`module.exports`).

---

# Interview-Ready Summary

> **`package.json` is the central configuration file of a Node.js/Express.js project. It defines project metadata, manages dependencies and development dependencies, provides npm scripts, specifies the application's entry point and module type, and can include engine requirements and tool configurations. Together with `package-lock.json`, it ensures the project can be installed, run, and maintained consistently across development, testing, and production environments.**

## Question 2. How do you install Express.js in a Node.js application?

## Question 3. What is the difference between Express.js and the built-in Node.js `http` module?

## Question 4. How do you define multiple route handlers for the same route in Express.js?

## Question 5. What is the purpose of `req.method` in Express.js?

## Question 6. What does `req.originalUrl` contain?

## Question 7. How can you access request headers in Express.js?

## Question 8. How do you send HTML responses from Express.js?

## Question 9. What is the purpose of template engines in Express.js?

## Question 10. How do you configure EJS or Pug with Express.js?

## Question 11. What is the difference between server-side rendering and API-only Express.js applications?

## Question 12. How do you define wildcard routes in Express.js?

## Question 13. What are optional route parameters in Express.js?

## Question 14. How do you chain multiple middleware functions for a route?

## Question 15. What is the purpose of `res.status()`?

## Question 16. What is the purpose of `res.set()`?

## Question 17. How do you handle 404 errors in Express.js?

## Question 18. What happens if two routes match the same request path?

## Question 19. How can you access cookies sent by the client?

## Question 20. What is the difference between `npm install --save` and `npm install --save-dev`?
