---
title: "From Memory Rate Limits to Redis: Production-Ready Login Protection with Express"
description: "How to implement a real Redis-backed rate limit for admin login, ready for cloud and production"
tags: ["redis", "rate-limiting", "express", "nodejs", "security", "backend", "vercel", "cockroachdb"]
date: "2025-12-18"
author: "Maciel Alves"
image: "https://github.com/macielphp/blog/blob/main/assets/redisAndExpressjsRateLimit.webp?raw=true"
readTime: "10 min"
---
# Express Middleware: Product API vs CRUD API  
### Auth, Rate Limit and express.use — Explained with a Real Case

## Introduction

When building APIs with Express, many developers start by creating **CRUD APIs**.  
As the product grows, the same structure often becomes insufficient.

This article explains **how and why the usage of `express.use`, authentication, and rate limiting changes** when moving from a **CRUD API** to a **Product API**, using the **Self-Development-Room** project as a real-world case study.

---

## 1. CRUD API vs Product API — Conceptual Difference

### CRUD API
A CRUD API is:
- Table-oriented
- Endpoint-centric
- Operation-focused

Example:
```http
GET /languages
POST /languages
PUT /languages/:id
DELETE /languages/:id
```

### Product API
A Product API is:
- Feature-oriented
- Experience-driven
- Domain-focused

Example:
```http
GET /app/languages
GET /app/seasons?language=1
GET /app/lessons/10
POST /app/activities
````

This conceptual shift directly affects **how middleware is applied**.

---

## 2. How `express.use` Is Typically Used in CRUD APIs

In CRUD APIs, middleware is often attached **per route file** or **per endpoint**.

### Typical CRUD Pattern

```js
app.use('/languages', authMiddleware, languagesRoutes);
app.use('/seasons', authMiddleware, seasonsRoutes);
````

Characteristics:

* Repetitive
* Hard to scale
* Middleware logic scattered across files

This works for small APIs but becomes fragile as complexity grows.

---

## 3. `express.use` in Product APIs: Boundary-Based Design

Product APIs organize middleware by **product boundaries**, not tables.

### Self-Development-Room Boundaries

```
/admin/* → Admin product
/app/*   → User product
```

### Product-Oriented `express.use`

```js
app.use('/admin', adminAuthRoutes);

app.use('/admin/languages', authAdmin, adminLanguagesRoutes);
app.use('/admin/seasons', authAdmin, adminSeasonsRoutes);

app.use(
  '/app',
  authUser,
  userApiLimiter,
  userLanguageRoutes,
  userSeasonRoutes,
  userLessonRoutes,
  userActivityRoutes
);
```

Key difference:

> Middleware is applied **once per product**, not per resource.

---

## 4. Authentication: CRUD vs Product API

### CRUD API Authentication

In CRUD APIs:

* Auth is usually added to every route
* Often duplicated
* Hard to evolve

```js
router.get('/', auth, controller);
router.post('/', auth, controller);
```

---

### Product API Authentication

In Product APIs:

* Authentication is applied at the **product level**
* Routes inherit auth automatically

```js
app.use('/app', authUser, userRoutes);
```

### Benefits

* Cleaner route files
* Centralized security
* Easier auditing

---

## 5. Rate Limiting: Why CRUD Logic Fails in Products

### CRUD API Rate Limit (Naive)

```js
router.post('/login', limiter, loginController);
```

Problems:

* No product context
* No differentiation between public/private APIs
* Hard to tune limits

---

## 6. Product-Aware Rate Limiting (Self-Development-Room)

### Admin Login (Strict)

```js
router.post('/login', adminLoginLimiter, loginAdmin);
```

* Very low limit
* High security
* Sensitive endpoint

---

### User Product API (Broad & Flexible)

```js
app.use(
  '/app',
  authUser,
  userApiLimiter,
  userRoutes
);
```

Configuration example:

```js
windowMs: 1 * 60 * 1000,
max: 60
```

Why this works:

* Rate limit applies to the **whole product**
* Limits are per user/IP
* Easy to evolve

---

## 7. Why Not Put Rate Limit Inside Each Route?

In Product APIs:

* Users don’t think in endpoints
* They think in flows

Rate limiting should reflect:

* Usage patterns
* Business rules
* Product experience

Putting rate limit per route:

* Breaks consistency
* Adds noise
* Increases maintenance cost

---

## 8. CRUD Mentality vs Product Mentality

| Aspect      | CRUD API       | Product API      |
| ----------- | -------------- | ---------------- |
| Focus       | Table          | Feature          |
| Middleware  | Per route      | Per product      |
| Auth        | Repeated       | Centralized      |
| Rate Limit  | Endpoint-based | Experience-based |
| Scalability | Low            | High             |

---

## 9. Self-Development-Room: Why This Matters

In Self-Development-Room:

* Admins need strict security
* Users need smooth experience
* Learning flow must not break due to endpoint limits

By separating:

* `/admin/*`
* `/app/*`

The system gains:

* Clear security rules
* Clean middleware structure
* Production-ready scalability

---

## 10. Key Takeaways

✔ `express.use` should define **product boundaries**
✔ Authentication belongs to **product scope**, not tables
✔ Rate limiting should reflect **user behavior**, not endpoints
✔ CRUD patterns don’t scale into real products
✔ Product APIs are about **experience**, not database structure

---

## Conclusion

A CRUD API teaches you how to move data.

A Product API teaches you how to build systems that:

* Scale
* Stay secure
* Match real user behavior

The **Self-Development-Room** case shows that middleware architecture is not just technical — it is **product design**.

