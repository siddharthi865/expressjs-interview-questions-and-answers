# Set 18

| S.No. | Question                                                                                                                                                 |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you detect suspicious login activity in backend systems?](#question-1-how-do-you-detect-suspicious-login-activity-in-backend-systems)            |
| 2.    | [How would you implement account lockout mechanisms?](#question-2-how-would-you-implement-account-lockout-mechanisms)                                    |
| 3.    | [What are the challenges of implementing MFA in APIs?](#question-3-what-are-the-challenges-of-implementing-mfa-in-apis)                                  |
| 4.    | [How do you validate nested request payloads efficiently?](#question-4-how-do-you-validate-nested-request-payloads-efficiently)                          |
| 5.    | [What is schema versioning in APIs?](#question-5-what-is-schema-versioning-in-apis)                                                                      |
| 6.    | [How do you deprecate old API endpoints safely?](#question-6-how-do-you-deprecate-old-api-endpoints-safely)                                              |
| 7.    | [What are breaking vs non-breaking API changes?](#question-7-what-are-breaking-vs-non-breaking-api-changes)                                              |
| 8.    | [How do you design APIs for backward compatibility?](#question-8-how-do-you-design-apis-for-backward-compatibility)                                      |
| 9.    | [What are common pagination abuse problems in APIs?](#question-9-what-are-common-pagination-abuse-problems-in-apis)                                      |
| 10.   | [How would you implement search endpoints efficiently?](#question-10-how-would-you-implement-search-endpoints-efficiently)                               |
| 11.   | [What are N+1 query problems and how do they affect APIs?](#question-11-what-are-n1-query-problems-and-how-do-they-affect-apis)                          |
| 12.   | [How do you optimize serialization and deserialization performance?](#question-12-how-do-you-optimize-serialization-and-deserialization-performance)     |
| 13.   | [What are common bottlenecks in JSON-heavy APIs?](#question-13-what-are-common-bottlenecks-in-json-heavy-apis)                                           |
| 14.   | [How would you compress large API payloads effectively?](#question-14-how-would-you-compress-large-api-payloads-effectively)                             |
| 15.   | [What is lazy loading in backend systems?](#question-15-what-is-lazy-loading-in-backend-systems)                                                         |
| 16.   | [How do you implement feature toggles for API behavior changes?](#question-16-how-do-you-implement-feature-toggles-for-api-behavior-changes)             |
| 17.   | [What are the risks of shared mutable state in Node.js applications?](#question-17-what-are-the-risks-of-shared-mutable-state-in-nodejs-applications)    |
| 18.   | [How do you handle partial failures in distributed requests?](#question-18-how-do-you-handle-partial-failures-in-distributed-requests)                   |
| 19.   | [What are fallback mechanisms in backend services?](#question-19-what-are-fallback-mechanisms-in-backend-services)                                       |
| 20.   | [How do you implement service degradation strategies during overload?](#question-20-how-do-you-implement-service-degradation-strategies-during-overload) |

## Question 1. How do you detect suspicious login activity in backend systems?

## Direct Answer

Suspicious login activity is detected by combining **behavioral analysis, risk scoring, device and network fingerprinting, anomaly detection, and security controls**. Instead of relying on a single rule, modern backend systems assign a **risk score** to each login attempt based on factors such as IP address, geolocation, device, login time, failed attempts, and user behavior. High-risk logins can then trigger additional verification like MFA, CAPTCHA, or temporary account lockout.

---

# Detailed Explanation

A secure backend should evaluate every login request before granting access.

Typical flow:

```
User Login Request
        │
        ▼
Validate Credentials
        │
        ▼
Collect Login Metadata
(IP, Device, Browser, Location, Time)
        │
        ▼
Risk Analysis Engine
        │
 ┌──────┼─────────┐
 │      │         │
Low   Medium    High Risk
 │      │         │
Allow  Require   Block /
       MFA       Review
```

---

# Common Indicators of Suspicious Activity

## 1. Multiple Failed Login Attempts

One of the simplest indicators.

Example:

```
10 failed logins
within 2 minutes
from same IP
```

Possible actions:

- Rate limiting
- CAPTCHA
- Temporary IP block
- Temporary account lock

Example:

```javascript
const loginAttempts = new Map();

function recordFailure(ip) {
  const count = loginAttempts.get(ip) || 0;
  loginAttempts.set(ip, count + 1);

  if (count >= 5) {
    console.log("Block IP temporarily");
  }
}
```

In production, this is typically stored in Redis rather than process memory.

---

## 2. Impossible Travel Detection

Example:

```
10:00 AM
Login from India

↓

10:20 AM

Login from Germany
```

The user cannot realistically travel that distance in 20 minutes.

Backend actions:

- Challenge with MFA
- Notify user
- Require password reset if extremely suspicious

---

## 3. New Device Detection

Track:

- Browser
- OS
- Device ID
- Cookies
- User Agent

If a login comes from an unknown device:

```
Known devices:
✓ Laptop
✓ Mobile

Unknown:
Windows PC
```

Trigger:

- Email verification
- OTP
- MFA

---

## 4. Unusual Login Time

Example:

```
User normally logs in

9 AM - 6 PM

Today:

3:12 AM
```

This alone isn't proof of compromise, but it increases the overall risk score.

---

## 5. Login from Anonymous Networks

Detect:

- VPN
- Tor
- Proxy
- Hosting providers
- Data center IPs

These IPs often receive higher risk scores.

Many companies use threat intelligence databases to classify IP reputation.

---

## 6. Geographic Anomalies

Example:

```
Previous:

Delhi

Current:

Russia
```

If unexpected:

- Require MFA
- Send notification email

---

## 7. Multiple Accounts from Same IP

Example:

```
IP:

192.168.x.x

Attempts:

alice@example.com
bob@example.com
charlie@example.com
...
```

This may indicate:

- Credential stuffing
- Brute-force attack

---

## 8. Credential Stuffing Detection

Attackers use leaked username/password combinations.

Indicators:

- Thousands of usernames
- Same password pattern
- High failure rate
- Multiple IPs

Mitigation:

- Rate limiting
- IP reputation
- MFA
- CAPTCHA
- Password breach checks

---

## 9. Password Spray Attack

Instead of trying many passwords for one account:

```
Password:

Summer2026!

↓

Applied to

1000 accounts
```

Detection:

- Same password
- Many usernames
- Short time window

---

## 10. Excessive Login Velocity

Example:

```
User logs in

20 times

within 1 minute
```

Possible causes:

- Session theft
- Automation
- Bots

---

# Risk Scoring

Rather than using isolated rules, assign points to suspicious signals.

Example:

| Event                 | Risk Score |
| --------------------- | ---------: |
| New device            |        +20 |
| New country           |        +30 |
| VPN detected          |        +15 |
| Failed attempts       |        +25 |
| Impossible travel     |        +40 |
| Login at unusual time |        +10 |

Total:

```
20 + 30 + 25

= 75
```

Policy:

```
0–30
Allow

31–60
Require MFA

61+
Block or manual verification
```

This layered approach reduces false positives while responding proportionally to risk.

---

# Express.js Example

```javascript
app.post("/login", async (req, res) => {
  const { email, password } = req.body;

  const ip = req.ip;
  const userAgent = req.headers["user-agent"];

  let riskScore = 0;

  if (isNewDevice(email, userAgent)) {
    riskScore += 20;
  }

  if (isHighRiskIP(ip)) {
    riskScore += 30;
  }

  if (riskScore >= 50) {
    return res.status(403).json({
      message: "Additional verification required",
    });
  }

  // Continue authentication
});
```

In production, helper functions like `isNewDevice` and `isHighRiskIP` would query persistent stores and threat intelligence services rather than relying on in-memory data.

---

# Logging for Security Auditing

Record every login attempt with useful metadata:

```json
{
  "userId": "12345",
  "timestamp": "2026-06-19T10:15:00Z",
  "ip": "203.0.113.25",
  "country": "India",
  "device": "Chrome on Windows",
  "success": true,
  "riskScore": 35
}
```

Avoid logging sensitive information such as plaintext passwords or authentication tokens. Security logs should be centralized, retained according to policy, and monitored for anomalies.

---

# Additional Protection Techniques

- **Multi-Factor Authentication (MFA):** Require a second factor for medium- or high-risk logins.
- **Rate Limiting:** Limit login attempts per IP, user, or device to reduce brute-force attacks.
- **CAPTCHA:** Challenge suspected automated traffic after repeated failures.
- **Account Lockout:** Temporarily lock or progressively delay authentication after repeated failed attempts, avoiding overly aggressive permanent lockouts that can enable denial-of-service attacks.
- **User Notifications:** Alert users about logins from new devices or unusual locations.
- **Session Management:** Invalidate or revoke existing sessions after confirmed account compromise or password changes.
- **Threat Intelligence:** Use IP reputation feeds to identify malicious networks and known bot activity.
- **Continuous Monitoring:** Feed authentication events into a SIEM or monitoring platform to detect patterns across users and services.

---

# Common Pitfalls

- Blocking users solely because they changed location or are traveling.
- Relying only on IP addresses, which can change frequently or be shared.
- Trusting the `User-Agent` header as a unique device identifier, since it can be spoofed.
- Applying permanent account lockouts after a few failures, which attackers can abuse to lock out legitimate users.
- Not logging authentication events, making incident investigation difficult.
- Storing login attempt counters only in memory, which doesn't work well in distributed deployments.

---

# Best Practices

- Use **risk-based authentication** instead of single-rule decisions.
- Combine multiple signals (IP, device, geolocation, time, historical behavior, and failed attempts) for more accurate detection.
- Store counters and session metadata in shared stores like Redis for horizontally scaled systems.
- Implement progressive responses: allow low-risk logins, require MFA for medium risk, and block or escalate only high-risk attempts.
- Monitor authentication metrics continuously and update detection rules as attack patterns evolve.
- Protect user privacy by collecting only necessary metadata and securing authentication logs.

---

## Interview Summary

In backend systems, suspicious login activity is best detected using a **risk-based authentication model**. Rather than depending on a single indicator, the backend evaluates multiple signals—such as failed login attempts, unfamiliar devices, unusual geolocations, impossible travel, IP reputation, and behavioral anomalies—to calculate a risk score. Based on that score, the system can transparently allow the login, require additional verification like MFA, or block the attempt. This layered approach provides stronger security while minimizing unnecessary friction for legitimate users.

## Question 2. How would you implement account lockout mechanisms?

## Question 3. What are the challenges of implementing MFA in APIs?

## Question 4. How do you validate nested request payloads efficiently?

## Question 5. What is schema versioning in APIs?

## Question 6. How do you deprecate old API endpoints safely?

## Question 7. What are breaking vs non-breaking API changes?

## Question 8. How do you design APIs for backward compatibility?

## Question 9. What are common pagination abuse problems in APIs?

## Question 10. How would you implement search endpoints efficiently?

## Question 11. What are N+1 query problems and how do they affect APIs?

## Question 12. How do you optimize serialization and deserialization performance?

## Question 13. What are common bottlenecks in JSON-heavy APIs?

## Question 14. How would you compress large API payloads effectively?

## Question 15. What is lazy loading in backend systems?

## Question 16. How do you implement feature toggles for API behavior changes?

## Question 17. What are the risks of shared mutable state in Node.js applications?

## Question 18. How do you handle partial failures in distributed requests?

## Question 19. What are fallback mechanisms in backend services?

## Question 20. How do you implement service degradation strategies during overload?
