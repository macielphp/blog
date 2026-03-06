---
title: "sameSite Cookies in Practice: Why `'strict'` Breaks Cross-Origin Applications"
description: ""
tags: [""]
date: "2026-01-12"
author: "Revised"
readTime: "5 min"
image: "https://github.com/macielphp/blog/blob/main/assets/self-dev-room-samesite.webp?raw=true"
---

## sameSite Cookies in Practice: Why `'strict'` Breaks Cross-Origin Applications

### (and How Self-Development Room Solved It)

When we talk about modern authentication using **JWT + refresh tokens stored in httpOnly cookies**, the `sameSite` attribute stops being a minor detail and becomes a **real architectural decision**.

In the **Self-Development Room** project, this became very clear when integrating:

* Traditional login (email + password)
* Google Sign-In (OAuth)
* Frontend hosted on a separate domain (Vercel)
* Node.js + Express backend
* Refresh tokens stored in `httpOnly` cookies

In this article, we'll explain **why `sameSite: 'strict'` fails in cross-origin scenarios**, when it *does* make sense, and how the correct configuration differs between **development and production**.

---

## What is `sameSite`?

The `sameSite` attribute defines **when a cookie is allowed to be sent along with a request**.

Available values:

| Value    | Behavior                                                                |
| -------- | ----------------------------------------------------------------------- |
| `strict` | Cookie is sent **only** if the navigation originates from the same site |
| `lax`    | Cookie is sent in common navigations (top-level GET requests)           |
| `none`   | Cookie is sent **in all contexts** (requires `secure: true`)            |

👉 Its primary purpose is to **mitigate CSRF attacks**.

📚 Reference:
[https://www.geeksforgeeks.org/computer-networks/what-is-samesite-cookies-and-csrf-protection/](https://www.geeksforgeeks.org/computer-networks/what-is-samesite-cookies-and-csrf-protection/)

---

## The common mistake: always using `sameSite: 'strict'`

At first glance, `strict` looks like the safest option:

```js
res.cookie('refreshToken', token, {
  httpOnly: true,
  sameSite: 'strict'
});
```

❌ **The problem:** this **breaks any cross-origin authentication flow**.

In Self-Development Room, the frontend runs on:

```
http://localhost:5173   (development)
https://*.vercel.app    (production)
```

And the backend runs on:

```
http://localhost:5000
https://api.yourdomain.com
```

➡️ These are **different origins** (different domains in production, different ports in development).

With `sameSite: 'strict'`:

* The cookie is **not sent**
* The refresh token never reaches the backend
* Sessions silently break
* Login appears to work, but the user is logged out shortly after

---

## The special case: Google Sign-In

Below is a **real excerpt** from the project:

```js
export const googleAuth = async (req, res) => {
  const { credential } = req.body;

  const { data } = await axios.get(
    `https://oauth2.googleapis.com/tokeninfo?id_token=${credential}`
  );

  const { email, name } = data;

  // find or create user...

  const refreshToken = generateRefreshToken({ id: user.id });

  const isProd = process.env.NODE_ENV === 'production';

  res.cookie('refreshToken', refreshToken, {
    httpOnly: true,
    secure: isProd,
    sameSite: isProd ? 'none' : 'lax',
  });

  res.json({ accessToken, user });
};
```

⚠️ If `sameSite` were set to `strict`:

* The cookie wouldn't be sent on subsequent API calls from the frontend
* The browser blocks the cookie in cross-origin requests
* Authentication fails even with a valid OAuth token

---

## The solution adopted in Self-Development Room

The project solved this using **environment-aware cookie configuration**:

```js
const isProd = process.env.NODE_ENV === 'production';

res.cookie('refreshToken', refreshToken, {
  httpOnly: true,
  secure: isProd,
  sameSite: isProd ? 'none' : 'lax',
  maxAge: 7 * 24 * 60 * 60 * 1000
});
```

### Why does this work?

#### 🧪 Development

```js
sameSite: 'lax'
secure: false
```

* Allows requests between `localhost:5173` → `localhost:5000`
* Does not require HTTPS
* Prevents unnecessary browser restrictions during development

#### 🚀 Production

```js
sameSite: 'none'
secure: true
```

* Allows frontend and backend on different domains
* Fully compatible with Vercel, Google OAuth, and mobile apps
* Browsers **require HTTPS**, maintaining security

---

## When you DON'T need `sameSite: 'none'`

`sameSite: 'none'` is **not required** for all SPAs, OAuth flows, or decoupled APIs. You can use `sameSite: 'lax'` or `'strict'` when:

* **Same-domain architecture**: Frontend and backend share the same domain (e.g., `example.com` serving both, or using a proxy)
* **Subdomain setup with proper configuration**: `app.example.com` and `api.example.com` can share cookies with the `domain` attribute set to `.example.com`
* **Token-based auth without cookies**: Using `Authorization` headers with tokens stored in memory
* **Standard OAuth flows**: Authorization Code Flow with PKCE works fine with `sameSite: 'lax'` because the redirect is a top-level navigation

### Alternative architectures to consider:

1. **API proxy**: Configure your frontend server to proxy `/api/*` requests to your backend
2. **Subdomain strategy**: Deploy both on subdomains of the same parent domain
3. **BFF (Backend-for-Frontend)**: Use a thin backend layer on the same domain as your frontend
4. **Token rotation pattern**: Store refresh tokens in `httpOnly` cookies on the same domain, access tokens in memory

---

## When `sameSite: 'none'` IS required

You need `sameSite: 'none'` specifically when:

* Frontend and backend are on **completely different domains** (e.g., `app.com` and `api.different.com`)
* You're using **cookie-based authentication** across those domains
* Your API is consumed by **third-party sites** or **embedded contexts**
* You have **mobile apps** that need to authenticate via web views

👉 This is a **deployment architecture choice**, not an inherent requirement of SPAs or OAuth.

---

## Security is more than cookies

In Self-Development Room, `sameSite` is just **one layer** of security:

* Refresh tokens stored in `httpOnly` cookies
* Short-lived access tokens in headers
* Rate limiting with Redis
* CSRF mitigation by architecture (not form tokens)
* Token rotation readiness
* Clear domain separation between frontend and backend

🔐 Security comes from the **system design**, not a single flag.

---

## Conclusion

> `sameSite: 'strict'` is secure **in theory**, but breaks cross-origin flows **in practice**.

Self-Development Room demonstrates that:

* Security must respect **real-world deployment choices**
* **Cross-origin SPAs with cookie-based auth require `sameSite: 'none'`**
* Different environments require **different configurations**
* **Same-domain architectures can use stricter `sameSite` values**

If your application:

* Uses Google Sign-In
* Is deployed on Vercel (separate domain from backend)
* Has a backend on a different domain
* Stores refresh tokens in cookies

👉 **`sameSite: 'none'` is the appropriate choice for this specific architecture.**

However, if you can restructure to use the same domain (via proxy or subdomains), you can achieve better security with `sameSite: 'lax'` or `'strict'`.

---

### Project reference

All explanations and code examples in this article are based on the project state up to the following commit:

🔗 [https://github.com/macielphp/self-development-room/commit/696c59ea96c75f517ba4808f7c9bcd249e7329b4](https://github.com/macielphp/self-development-room/commit/696c59ea96c75f517ba4808f7c9bcd249e7329b4)

Check the repository for newer updates and improvements.