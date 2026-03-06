---
title: "JWT Flow Explained Line by Line: How We Implemented Authentication in Self-Development-Room"
description: "Clear explanation about the project by Grok"
tags: ["vercel", "spa", "react-router", "deployment", "routing"]
date: "2026-01-10"
author: "Grok"
readTime: 5 min"
image: "https://github.com/macielphp/blog/blob/main/assets/self-dev-room-jwt.webp?raw=true"
---


# JWT Flow Explained Line by Line: How We Implemented Authentication in Self-Development-Room

**This article is based on the exact implementation found in commit:**  
https://github.com/macielphp/self-development-room/commit/696c59ea96c75f517ba4808f7c9bcd249e7329b4  
*(Note: While this commit focused on database connection updates, the authentication structure, JWT middlewares, token generation, and related controllers remain consistent with the version described here.)*

In the **[Self-Development-Room](https://github.com/macielphp/self-development-room)** project — a personal platform designed to track real-life growth and self-development through daily data — authentication is a core foundation. We use **JSON Web Tokens (JWT)** in a robust and secure way to protect user and admin routes, ensuring that only authenticated individuals can access their personal information and settings.

In this article, we walk through the complete JWT authentication flow, with a special focus on the `authUser` middleware, breaking it down line by line exactly as implemented in the repository. The goal is to show how this architecture works in practice in a real project, emphasizing security, performance, and simplicity.

## Project Context
Self-Development-Room is a personal application that helps turn self-awareness into concrete action. Users log daily information about finances, studies, career, habits, well-being, and reflections, and the system generates structured insights. To make this possible, we need:

- Strong authentication for regular users
- Separate, more restricted authentication for administrators
- Short-lived access tokens + long-lived refresh tokens
- Protection against abuse (rate limiting + strict validations)

## The Complete Flow: From Request to Controller

Let’s analyze a typical protected request, such as `GET /user/me`, which returns the logged-in user’s basic data.

### 1. The Client Sends the Request
After a successful login (regular or via Google), the frontend receives:

- **Access Token** (short-lived JWT, valid for 15 minutes)
- **Refresh Token** (stored in an httpOnly cookie, valid for 7 days)

Example of the header sent:
```
GET /user/me
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Golden rule adopted**: JWT always goes in the `Authorization: Bearer` header. Never in the body or query string.

### 2. The Middleware Captures the Header
In the file `middleware/authUser.js`:

```javascript
const authHeader = req.headers.authorization;
```

We extract the full value of the `Authorization` header. When successful, it looks like:
```
"Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### 3. Initial Validation – Blocking Invalid Requests
```javascript
if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ 
        message: 'Authentication token missing' 
    });
}
```

This simple check already prevents:
- Requests without a token
- Malformed tokens
- Direct attempts to access protected routes

### 4. Extracting the Pure Token
```javascript
const token = authHeader.split(' ')[1];
```

We split by space and take the second part — the raw JWT.

### 5. Cryptographic Verification – The Heart of Security
```javascript
try {
    const decoded = jwt.verify(token, process.env.JWT_ACCESS_SECRET);
    // ...
} catch (err) {
    return res.status(401).json({ message: 'Insufficient permissions' });
}
```

The `jwt.verify()` function performs several critical validations automatically:

- Verifies the signature using the correct secret key
- Checks if the token has expired (`exp` claim)
- Ensures the content has not been tampered with
- Validates the algorithm (HS256 in our case)

If anything is wrong, an exception is thrown and the flow stops with status 401.

Typical decoded payload:
```json
{
  "id": 42,
  "email": "user@example.com",
  "role": "user",
  "iat": 1736500000,
  "exp": 1736500900
}
```

### 6. Simple Role-Based Authorization Check
```javascript
if (decoded.role && decoded.role !== 'user') {
    return res.status(403).json({ message: 'Insufficient permissions'});
}
```

Even if the token is valid, we confirm that the `role` matches this route. This distinguishes regular users from other access types (and leaves room for future role expansion).

### 7. Injecting User Data into the Request Object
```javascript
req.user = {
    id: decoded.id,
    email: decoded.email,
    role: decoded.role
};
```

This is one of the most powerful lines in the implementation: from here on, **any downstream controller** can access `req.user.id`, `req.user.email`, etc. without querying the database again. This delivers great performance and a truly stateless architecture.

### 8. Releasing the Flow
```javascript
next();
```

With everything validated, control passes to the next middleware or the final controller.

## Integration with Rate Limiting
The layering order in the current architecture is critical:

```
Rate Limiter → Auth Middleware → Controller
```

- `userLoginLimiter` protects the login route against brute-force attacks
- `userApiLimiter` (when implemented) protects authenticated routes against valid token abuse

This sequence safeguards both CPU and database resources.

## Token System: Access + Refresh
We follow the modern standard:

- **Access Token** → 15 minutes, contains id, email, and role
- **Refresh Token** → 7 days, stored in httpOnly cookie, contains only id
- Separate secrets in `.env`:
  ```
  JWT_ACCESS_SECRET=...
  JWT_REFRESH_SECRET=...
  JWT_ADMIN_SECRET=...
  ```

When the access token expires, the frontend sends the refresh token (from the cookie) to the `/refresh-token` route, which issues a new valid access token.

## Conclusion: A Solid Foundation for Personal Growth
In Self-Development-Room, authentication is more than a technical requirement — it ensures that every insight, metric, and reflection belongs exclusively to the user who generated it.

The current JWT implementation (with separate middlewares for user and admin, distinct secrets, secure httpOnly cookie refresh tokens, rate limiting, and clean `req.user` injection) delivers:

- Adequate security for a personal project
- Excellent performance (stateless)
- Easy maintenance and future expansion

The full source code is open on GitHub: [https://github.com/macielphp/self-development-room](https://github.com/macielphp/self-development-room)

If you're building something similar or want to discuss improvements (token blacklisting, secret rotation, more granular RBAC), feel free to open an issue or pull request.

Happy personal development journey — and happy coding too! 🚀