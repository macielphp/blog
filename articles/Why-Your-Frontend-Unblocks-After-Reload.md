---
title: "Why Your Frontend Unblocks After Reload — and Why That’s Actually Correct"
description: "Understand why frontend state resets on reload while backend rate limiting remains enforced, and why this behavior is a sign of a secure architecture."
tags: [
  "backend",
  "frontend",
  "security",
  "rate limiting",
  "express",
  "nodejs",
  "react",
  "authentication",
  "web security",
  "owasp"
]
date: "2025-12-16"
author: "Maciel Alves"
image: "https://github.com/macielphp/blog/blob/main/assets/279ee680.webp?raw=true"
readTime: "8 min"
---
# Why Your Frontend “Unblocks” After Reload — and Why That’s Actually Correct

*Note: I’m learning, and this is a practical conclusion I reached with the help of AI. #StayLearning*

---

When building secure web applications, developers often implement rate limiting to prevent brute-force attacks on login endpoints. A common question that comes up during development is:

> *“Why does the frontend unblock after a page reload even though the backend rate limit hasn’t expired? Is this a bug?”*

No — and in this article, I’ll explain **why this happens**, why it’s **expected behavior**, and why it’s actually a sign your architecture is correct.

---

## Frontend vs Backend: Two Different Types of State

### Frontend (React)

In many React applications, we track UI state using React hooks such as:

```js
const [isBlocked, setIsBlocked] = useState(false);
```

This state lives **only in memory** — specifically, in the component instance. When you refresh the page:

* React unmounts the component.
* All state is reset.
* `isBlocked` becomes `false` again.

This means that, from the frontend’s perspective, the login button appears enabled again after reload.

📌 This is **normal and expected** behavior because the browser completely resets the JavaScript runtime on page reload.

---

### Backend (Express + rate limiting)

In contrast, backend rate limiting (e.g., using `express-rate-limit`) lives entirely on the server:

```js
const adminLoginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 3,
  standardHeaders: true,
  legacyHeaders: false,
  message: {
    error: 'Too many login attempts. Try again later.'
  }
});
```

Key characteristics:

* It’s based on the **client IP address**.
* It lives in the server’s memory (or external store).
* It is **independent of the frontend state**.

Even after a page reload, the backend still enforces the rate limit. If you try again before the window resets, the server returns:

```
HTTP 429 Too Many Requests
```

---

## Why It *Looks* Like the Block Disappeared

From your point of view in the UI:

1. You hit the rate limit and see an error.
2. You refresh the page.
3. The button looks enabled again.

➡️ But when you click that button:

* The frontend sends a new request.
* The backend checks the rate limit.
* The server returns **429** again.

So the block didn’t disappear — it just wasn’t stored on the frontend.

👉 **The backend is the source of truth. The frontend only reacts to the server response.**

---

## Why Not Store the Block on the Frontend?

It may seem appealing to store a “block until” timestamp in localStorage, sessionStorage, or memory. For example:

```js
localStorage.setItem('blockedUntil', Date.now() + 900000);
```

But this approach is insecure and unreliable:

🔓 **Leaks timing information**
📌 **Can be manipulated by the client**
🌫 **Creates a false sense of protection**

Security decisions should **never** be based solely on client-side logic. The frontend should only display what the server tells it — never decide access control.

---

## The Correct Flow

Here’s the right way to handle rate limiting in a login flow:

1. **Backend controls** the rate limit.
2. **Frontend responds** to the server status.
3. **Frontend UI updates**, but does not enforce security logic.

This ensures:

* The client doesn’t know how much time is left.
* The server maintains control over access.
* You avoid security pitfalls from trusting client logic.

---

## How to Confirm It Yourself

You can verify this behavior using your browser’s Developer Tools:

1. Trigger the rate limit (e.g., too many login attempts).
2. Reload the login page.
3. Attempt to log in again.
4. In the **Network tab**, inspect the request and response.

You’ll see:

```
Status: 429 Too Many Requests
```

Even though the UI looked enabled, the backend is still blocking.

---

## A Note About Production and Scaling

By default, `express-rate-limit` uses a **memory store**, which:

* Works well for development.
* Resets when the server restarts.
* Is not shared between multiple instances.

In larger deployments or clustered environments, you should use an external store (like **Redis**), which allows:

* Shared rate limit data across instances.
* Better horizontal scaling.
* Persistent state (until expiry).

---

## Conclusion

✔️ The rate limit is still in effect after a reload.
✔️ The frontend state resets because it lives in memory.
✔️ This is expected and desired behavior.
✔️ **Frontend should not be trusted for security logic.**

Security is more than just UX — but thoughtful UX can help communicate server-side decisions without revealing internal timing or logic.

**Stay curious. Keep learning. And build secure systems that behave predictably.**