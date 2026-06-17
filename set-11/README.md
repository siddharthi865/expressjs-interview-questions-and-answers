# Set 11

| S.No. | Question                                                                                                                                                           |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.    | [What is the purpose of the `public` directory in an Express.js application?](#question-1-what-is-the-purpose-of-the-public-directory-in-an-expressjs-application) |
| 2.    | [How do you return plain text responses in Express.js?](#question-2-how-do-you-return-plain-text-responses-in-expressjs)                                           |
| 3.    | [What is the difference between `req.path` and `req.url`?](#question-3-what-is-the-difference-between-reqpath-and-requrl)                                          |
| 4.    | [How do you access the hostname of an incoming request in Express.js?](#question-4-how-do-you-access-the-hostname-of-an-incoming-request-in-expressjs)             |
| 5.    | [What is the purpose of `req.ip`?](#question-5-what-is-the-purpose-of-reqip)                                                                                       |
| 6.    | [How can you detect whether a request is secure (HTTPS) in Express.js?](#question-6-how-can-you-detect-whether-a-request-is-secure-https-in-expressjs)             |
| 7.    | [What is the role of `app.locals` in Express.js?](#question-7-what-is-the-role-of-applocals-in-expressjs)                                                          |
| 8.    | [What is the role of `res.locals` in Express.js?](#question-8-what-is-the-role-of-reslocals-in-expressjs)                                                          |
| 9.    | [How do you define route prefixes in Express.js?](#question-9-how-do-you-define-route-prefixes-in-expressjs)                                                       |
| 10.   | [What is the difference between middleware and controllers?](#question-10-what-is-the-difference-between-middleware-and-controllers)                               |
| 11.   | [What are common MIME types returned by Express.js APIs?](#question-11-what-are-common-mime-types-returned-by-expressjs-apis)                                      |
| 12.   | [How do you send file downloads from Express.js?](#question-12-how-do-you-send-file-downloads-from-expressjs)                                                      |
| 13.   | [What is the purpose of `res.download()`?](#question-13-what-is-the-purpose-of-resdownload)                                                                        |
| 14.   | [What is the purpose of `res.sendFile()`?](#question-14-what-is-the-purpose-of-ressendfile)                                                                        |
| 15.   | [How do you access form data submitted from HTML forms?](#question-15-how-do-you-access-form-data-submitted-from-html-forms)                                       |
| 16.   | [What is URL encoding and why is it needed?](#question-16-what-is-url-encoding-and-why-is-it-needed)                                                               |
| 17.   | [How do you parse JSON request bodies manually without middleware?](#question-17-how-do-you-parse-json-request-bodies-manually-without-middleware)                 |
| 18.   | [What are route conflicts in Express.js?](#question-18-what-are-route-conflicts-in-expressjs)                                                                      |
| 19.   | [How do you handle unsupported HTTP methods in APIs?](#question-19-how-do-you-handle-unsupported-http-methods-in-apis)                                             |
| 20.   | [What is the purpose of the `NODE_ENV` environment variable?](#question-20-what-is-the-purpose-of-the-node_env-environment-variable)                               |

## Question 1. What is the purpose of the `public` directory in an Express.js application?

## Direct Answer

The **`public` directory** in an Express.js application is used to store **static assets** that are served directly to clients without any server-side processing. These assets typically include HTML, CSS, JavaScript files, images, fonts, videos, and other publicly accessible files.

Express serves these files using the built-in `express.static()` middleware.

---

# Detailed Explanation

In a typical Express.js project, the `public` directory contains files that should be accessible through HTTP requests.

Example project structure:

```text
my-app/
│
├── public/
│   ├── css/
│   │   └── style.css
│   ├── js/
│   │   └── app.js
│   ├── images/
│   │   └── logo.png
│   └── favicon.ico
│
├── routes/
├── controllers/
├── app.js
└── package.json
```

---

# Serving Static Files

Express provides the `express.static()` middleware.

```javascript
const express = require("express");

const app = express();

app.use(express.static("public"));

app.listen(3000);
```

Now the files become directly accessible:

```
public/css/style.css
→ http://localhost:3000/css/style.css

public/images/logo.png
→ http://localhost:3000/images/logo.png

public/js/app.js
→ http://localhost:3000/js/app.js
```

Notice that the `public` folder name does **not** appear in the URL.

---

# Why Use a Public Directory?

It separates **static content** from **application logic**.

Instead of writing routes like:

```javascript
app.get("/logo", (req, res) => {
  res.sendFile(__dirname + "/public/images/logo.png");
});
```

You simply configure:

```javascript
app.use(express.static("public"));
```

Express automatically handles serving the files.

---

# Common Files Stored in `public`

- CSS files
- Client-side JavaScript
- Images
- Icons/Favicon
- Fonts
- PDFs
- Audio
- Video
- Static HTML pages

Example:

```text
public/
├── css/
├── js/
├── images/
├── fonts/
├── videos/
├── favicon.ico
└── robots.txt
```

---

# Mounting the Public Directory

You can expose it from a custom URL path.

```javascript
app.use("/static", express.static("public"));
```

Now:

```
public/images/logo.png
```

becomes:

```
http://localhost:3000/static/images/logo.png
```

---

# Multiple Static Directories

Express allows serving multiple static folders.

```javascript
app.use(express.static("public"));
app.use(express.static("assets"));
```

Express searches them in the order they are registered.

---

# Absolute Path (Recommended)

Instead of using a relative path:

```javascript
app.use(express.static("public"));
```

Use an absolute path to avoid issues when starting the app from different working directories.

```javascript
const path = require("path");

app.use(express.static(path.join(__dirname, "public")));
```

---

# Middleware Flow

```text
Browser
    │
GET /css/style.css
    │
    ▼
express.static()
    │
Find file in public/css/style.css
    │
    ▼
Send file directly
```

If the file isn't found, Express passes control to the next middleware.

```javascript
app.use(express.static("public"));

app.get("*", (req, res) => {
  res.status(404).send("Not Found");
});
```

---

# Performance Benefits

`express.static()` provides several optimizations:

- Efficient file streaming
- Proper MIME type detection
- Support for HTTP caching headers
- Conditional requests (`ETag`, `Last-Modified`)
- Automatic handling of `HEAD` requests
- Support for byte-range requests (useful for media files)

---

# Security Considerations

Only place files in `public` that are safe for anyone to access.

**Safe:**

- CSS
- JavaScript bundles
- Images
- Fonts
- Public documents

**Never place in `public`:**

- Environment files (`.env`)
- Database files
- API keys
- Server-side code
- Configuration files
- Private uploads

Example (bad):

```text
public/
├── .env
├── config.json
├── users.db
```

These files could become publicly accessible.

---

# Best Practices

- Keep only publicly accessible assets in the `public` directory.
- Use `path.join(__dirname, 'public')` (or the appropriate equivalent in ES modules) when configuring `express.static()`.
- Organize assets into subfolders such as `css`, `js`, `images`, and `fonts`.
- Serve versioned or hashed asset filenames (e.g., `app.a1b2c3.js`) in production to improve cache management.
- Use a reverse proxy (such as Nginx) or a CDN to serve static assets in production for better scalability and performance.

---

# Common Interview Follow-up

### **Q: Does Express require the folder to be named `public`?**

**No.** The name is purely a convention. Any directory can be used.

```javascript
app.use(express.static("assets"));
```

or

```javascript
app.use("/files", express.static("static"));
```

---

## Interview Summary

The `public` directory is a convention for storing **static files** that should be directly accessible by clients. Express serves these files using `express.static()`, which efficiently handles static asset delivery with features like MIME type detection, caching support, and file streaming. Keeping only public assets in this directory and organizing them properly improves maintainability, security, and application performance.

## Question 2. How do you return plain text responses in Express.js?

## Question 3. What is the difference between `req.path` and `req.url`?

## Question 4. How do you access the hostname of an incoming request in Express.js?

## Question 5. What is the purpose of `req.ip`?

## Question 6. How can you detect whether a request is secure (HTTPS) in Express.js?

## Question 7. What is the role of `app.locals` in Express.js?

## Question 8. What is the role of `res.locals` in Express.js?

## Question 9. How do you define route prefixes in Express.js?

## Question 10. What is the difference between middleware and controllers?

## Question 11. What are common MIME types returned by Express.js APIs?

## Question 12. How do you send file downloads from Express.js?

## Question 13. What is the purpose of `res.download()`?

## Question 14. What is the purpose of `res.sendFile()`?

## Question 15. How do you access form data submitted from HTML forms?

## Question 16. What is URL encoding and why is it needed?

## Question 17. How do you parse JSON request bodies manually without middleware?

## Question 18. What are route conflicts in Express.js?

## Question 19. How do you handle unsupported HTTP methods in APIs?

## Question 20. What is the purpose of the `NODE_ENV` environment variable?
