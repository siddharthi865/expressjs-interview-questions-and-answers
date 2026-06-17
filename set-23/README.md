# Set 23

| S.No. | Question                                                                                                                                            |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How would you implement multilingual error messages in APIs?](#question-1-how-would-you-implement-multilingual-error-messages-in-apis)             |
| 2.    | [How do locale and timezone handling affect backend API design?](#question-2-how-do-locale-and-timezone-handling-affect-backend-api-design)         |
| 3.    | [What are common serialization formats besides JSON?](#question-3-what-are-common-serialization-formats-besides-json)                               |
| 4.    | [How do Protocol Buffers compare with JSON APIs?](#question-4-how-do-protocol-buffers-compare-with-json-apis)                                       |
| 5.    | [What are the trade-offs of binary protocols in backend systems?](#question-5-what-are-the-trade-offs-of-binary-protocols-in-backend-systems)       |
| 6.    | [How do you secure public-facing APIs from automated scraping?](#question-6-how-do-you-secure-public-facing-apis-from-automated-scraping)           |
| 7.    | [What is device fingerprinting and how can it support API security?](#question-7-what-is-device-fingerprinting-and-how-can-it-support-api-security) |
| 8.    | [How do you implement IP allowlists and denylists in Express.js?](#question-8-how-do-you-implement-ip-allowlists-and-denylists-in-expressjs)        |
| 9.    | [What are bot mitigation strategies for APIs?](#question-9-what-are-bot-mitigation-strategies-for-apis)                                             |
| 10.   | [How do you detect abnormal API usage patterns?](#question-10-how-do-you-detect-abnormal-api-usage-patterns)                                        |
| 11.   | [How do you implement rolling API keys without downtime?](#question-11-how-do-you-implement-rolling-api-keys-without-downtime)                      |
| 12.   | [What are the benefits and risks of shared caches?](#question-12-what-are-the-benefits-and-risks-of-shared-caches)                                  |
| 13.   | [How do cache stampedes occur in backend systems?](#question-13-how-do-cache-stampedes-occur-in-backend-systems)                                    |
| 14.   | [What are stale-while-revalidate caching strategies?](#question-14-what-are-stale-while-revalidate-caching-strategies)                              |
| 15.   | [How do you implement distributed session invalidation?](#question-15-how-do-you-implement-distributed-session-invalidation)                        |
| 16.   | [What are sticky caches and when can they become problematic?](#question-16-what-are-sticky-caches-and-when-can-they-become-problematic)            |
| 17.   | [How do you design APIs for offline-first applications?](#question-17-how-do-you-design-apis-for-offline-first-applications)                        |
| 18.   | [What are synchronization conflicts in offline-capable systems?](#question-18-what-are-synchronization-conflicts-in-offline-capable-systems)        |
| 19.   | [How do you handle duplicate form submissions in APIs?](#question-19-how-do-you-handle-duplicate-form-submissions-in-apis)                          |
| 20.   | [What are common causes of inconsistent API state across services?](#question-20-what-are-common-causes-of-inconsistent-api-state-across-services)  |

## Question 1. How would you implement multilingual error messages in APIs?

## Direct Answer

To implement multilingual error messages in APIs, I would separate **error codes** from **human-readable messages**, detect the user's preferred language (typically using the `Accept-Language` header or user profile), and use an internationalization (i18n) library or translation files to return localized error messages while keeping the error structure consistent.

A common response format is:

```json
{
  "success": false,
  "error": {
    "code": "INVALID_EMAIL",
    "message": "Invalid email address."
  }
}
```

The `code` remains constant across all languages, while the `message` is translated based on the user's locale.

---

# Detailed Explanation

A well-designed multilingual API should satisfy three goals:

- Keep error codes language-independent.
- Translate only user-facing messages.
- Allow clients to reliably handle errors regardless of language.

Never rely on translated text for application logic.

---

# 1. Use Stable Error Codes

Instead of returning only text:

```json
{
  "message": "Email is invalid"
}
```

Return:

```json
{
  "error": {
    "code": "INVALID_EMAIL",
    "message": "Email is invalid"
  }
}
```

The frontend can:

- identify the error using `INVALID_EMAIL`
- display the translated message
- implement language-independent logic

---

# 2. Detect User Language

Common approaches:

### Accept-Language Header

```
Accept-Language: fr
```

or

```
Accept-Language: en-US,en;q=0.9
```

Express middleware:

```javascript
app.use((req, res, next) => {
  req.locale = req.acceptsLanguages("en", "fr", "es") || "en";
  next();
});
```

---

### User Profile

Some applications store the preferred language:

```javascript
{
    id: 12,
    language: "es"
}
```

After authentication:

```javascript
req.locale = req.user.language;
```

---

### Query Parameter (Less Common)

```
GET /products?lang=fr
```

Useful for testing but generally less preferred than headers or stored user preferences.

---

# 3. Store Translation Files

Example directory:

```
locales/
    en.json
    fr.json
    es.json
```

**en.json**

```json
{
  "INVALID_EMAIL": "Invalid email address.",
  "USER_NOT_FOUND": "User not found.",
  "PASSWORD_REQUIRED": "Password is required."
}
```

**fr.json**

```json
{
  "INVALID_EMAIL": "Adresse e-mail invalide.",
  "USER_NOT_FOUND": "Utilisateur introuvable.",
  "PASSWORD_REQUIRED": "Le mot de passe est obligatoire."
}
```

---

# 4. Use an i18n Library

Popular libraries include:

- i18next
- i18n
- Polyglot.js (client-side)
- Custom translation service

Example:

```javascript
const i18next = require("i18next");

const message = i18next.t("INVALID_EMAIL", {
  lng: req.locale,
});
```

---

# 5. Centralized Error Handler

```javascript
app.use((err, req, res, next) => {
  const message = req.t(err.code);

  res.status(err.status || 500).json({
    success: false,
    error: {
      code: err.code,
      message,
    },
  });
});
```

Throw errors like:

```javascript
next({
  status: 400,
  code: "INVALID_EMAIL",
});
```

---

# 6. Validation Errors

Validation libraries usually return machine-readable information.

Example:

```javascript
next({
  status: 400,
  code: "VALIDATION_ERROR",
  fields: [
    {
      field: "email",
      code: "INVALID_EMAIL",
    },
  ],
});
```

Response:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed.",
    "fields": [
      {
        "field": "email",
        "code": "INVALID_EMAIL",
        "message": "Adresse e-mail invalide."
      }
    ]
  }
}
```

Each field message can be translated independently.

---

# 7. Keep Internal Errors Untranslated

Do not expose internal exception messages:

```javascript
throw new Error("MongoServerError: duplicate key...");
```

Instead:

```json
{
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "An unexpected error occurred."
  }
}
```

Log the actual error internally.

---

# Example Express Implementation

```javascript
const translations = {
  en: {
    INVALID_EMAIL: "Invalid email address.",
  },
  fr: {
    INVALID_EMAIL: "Adresse e-mail invalide.",
  },
};

app.use((req, res, next) => {
  req.locale = req.acceptsLanguages("en", "fr") || "en";

  req.t = (key) => translations[req.locale][key] || translations.en[key] || key;

  next();
});

app.get("/users", (req, res) => {
  res.status(400).json({
    error: {
      code: "INVALID_EMAIL",
      message: req.t("INVALID_EMAIL"),
    },
  });
});
```

---

# Best Practices

- Use stable, language-independent error codes.
- Detect locale using the `Accept-Language` header or authenticated user preferences.
- Store translations in external resource files rather than hardcoding strings.
- Centralize translation logic in middleware or a global error handler.
- Provide a fallback language (typically English) when a translation is unavailable.
- Return localized messages only for user-facing text; keep logs and internal diagnostics in a single language (commonly English).
- Localize validation messages consistently across all API endpoints.

---

# Common Pitfalls

- **Using translated messages for program logic:** Clients should rely on error codes, not message text.
- **Hardcoding messages in controllers:** This makes maintenance and localization difficult.
- **Inconsistent error formats:** Keep the JSON structure the same regardless of language.
- **Missing fallback translations:** Always default to a known language if a translation key is missing.
- **Exposing raw exception messages:** Internal errors should be logged, not sent directly to API consumers.

---

# Interview Tip

In an interview, emphasize that **error codes are the API contract**, while **localized messages are a presentation concern**. A robust multilingual API uses stable error codes, detects the user's locale via the `Accept-Language` header or profile settings, translates messages centrally using an i18n library, and falls back gracefully when translations are unavailable. This approach keeps the API reliable for both machines and humans.

## Question 2. How do locale and timezone handling affect backend API design?

## Question 3. What are common serialization formats besides JSON?

## Question 4. How do Protocol Buffers compare with JSON APIs?

## Question 5. What are the trade-offs of binary protocols in backend systems?

## Question 6. How do you secure public-facing APIs from automated scraping?

## Question 7. What is device fingerprinting and how can it support API security?

## Question 8. How do you implement IP allowlists and denylists in Express.js?

## Question 9. What are bot mitigation strategies for APIs?

## Question 10. How do you detect abnormal API usage patterns?

## Question 11. How do you implement rolling API keys without downtime?

## Question 12. What are the benefits and risks of shared caches?

## Question 13. How do cache stampedes occur in backend systems?

## Question 14. What are stale-while-revalidate caching strategies?

## Question 15. How do you implement distributed session invalidation?

## Question 16. What are sticky caches and when can they become problematic?

## Question 17. How do you design APIs for offline-first applications?

## Question 18. What are synchronization conflicts in offline-capable systems?

## Question 19. How do you handle duplicate form submissions in APIs?

## Question 20. What are common causes of inconsistent API state across services?
