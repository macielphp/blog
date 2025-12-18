---
title: "From Memory Rate Limits to Redis: Production-Ready Login Protection with Express"
description: "How to implement a real Redis-backed rate limit for admin login, ready for cloud and production"
tags: ["redis", "rate limiting", "express", "nodejs", "security", "backend", "vercel", "cockroachdb"]
date: "2025-12-18"
author: "Maciel Alves"
image: "https://github.com/macielphp/blog/blob/main/assets/redisAndExpressjsRateLimit.webp?raw=true"
readTime: "10 min"
---

## Introduction

Rate limiting is one of those backend topics that *looks simple* until you deploy your application.

Everything works fine locally:

* You add `express-rate-limit`


* You set `max: 3`
* You test it
* You see a `429 Too Many Requests`


Then you deploy.

Suddenly:

* Reloading the page resets the limit
* Scaling the server breaks protection
* Attacks bypass your defenses

This article documents a **real production case**: migrating from an in-memory rate limiter to a **Redis-backed store**, using:

* Express
* Redis Cloud (Free Plan)
* Vercel-compatible architecture
* Secure admin login protection

No theory. Only practical decisions.

---

## The Problem with Memory-Based Rate Limits

By default, `express-rate-limit` stores counters **in memory**.

That means:

* ❌ Server restart resets all limits
* ❌ Multiple instances don’t share state
* ❌ Serverless environments break protection

In other words:

> Memory-based rate limiting is **not security**. It’s a development convenience.

For an admin login endpoint, this is unacceptable.

---

## The Architecture Goal

We want:

* Persistent rate limiting
* Shared counters across instances
* Cloud compatibility
* Free-tier friendly

So the final stack looks like this:

```
Client → Vercel → Express API → Redis Cloud
                         ↓
                    CockroachDB
```

Redis becomes the **source of truth** for rate limits.

---

## Step 1 — Redis Cloud (Free Plan)

Redis Cloud offers a free tier:

* 30 MB
* Shared cloud deployment
* TLS enabled
* No credit card required

This is **more than enough** for:

* Rate limiting
* Auth throttling
* Small counters and TTLs

Once the database is created, Redis provides a connection URL like:

```
rediss://default:password@host:port
```

This URL is all we need.

---

## Step 2 — Redis Client Configuration

Create a Redis client using the official library.

`api/config/redis.js`

```js
import { createClient } from 'redis'
import dotenv from 'dotenv'

dotenv.config()

const redisClient = createClient({
  url: process.env.REDIS_URL
})

redisClient.on('error', (err) => {
  console.log('Redis client error:', err)
})

await redisClient.connect()

export default redisClient
```

This client:

* Uses TLS automatically
* Works locally and in production
* Fails loudly if Redis is unavailable

---

## Step 3 — Redis-Based Rate Limit Middleware

Now we replace memory storage with a real Redis store.

`api/middleware/rateLimit.js`

```js
import rateLimit from 'express-rate-limit'
import RedisStore from 'rate-limit-redis'
import redisClient from '../config/redis.js'

const adminLoginLimiter = rateLimit({
  store: new RedisStore({
    sendCommand: (...args) => redisClient.sendCommand(args)
  }),
  windowMs: 15 * 60 * 1000,
  max: 3,
  standardHeaders: true,
  legacyHeaders: false,
  message: {
    error: 'Too many login attempts. Try again later.'
  }
})

export default adminLoginLimiter
```

Now the rate limit:

* Persists across reloads
* Works across instances
* Survives deployments

This is **real protection**.

---

## Step 4 — Applying the Rate Limit Only Where It Matters

We apply the limiter **only to the login route**.

`api/routes/adminAuthRoutes.js`

```js
import express from 'express'
import { loginAdmin } from '../controllers/adminController.js'
import adminLoginLimiter from '../middleware/rateLimit.js'

const router = express.Router()

router.post('/login', adminLoginLimiter, loginAdmin)

export default router
```

This avoids:

* Penalizing authenticated users
* Limiting normal API usage

Security is **focused**, not global.

---

## Step 5 — Trusting the Proxy (Critical)

When running behind Vercel or any proxy, Express must trust it.

`api/index.js`

```js
app.set('trust proxy', 1)
```

Without this:

* All requests appear from the same IP
* Rate limits become useless

This single line is **mandatory** in production.

---

## Final Result

After this implementation:

* Brute-force attacks are blocked
* Reloading the page doesn’t reset limits
* Redis handles all counters
* The system scales naturally

This setup is:

* ✅ Free
* ✅ Production-ready
* ✅ Cloud-safe
* ✅ Secure

---

## Key Takeaways

* Memory rate limits are not security
* Redis turns rate limiting into infrastructure
* Free tiers are enough when used correctly
* Good security is about **where** you apply protection

If you’re building an admin panel, this is the **minimum acceptable baseline**.

---

## What’s Next?

Possible evolutions:

* Rate limit by IP + email
* Temporary account lockout
* Login attempt audit logs
* Frontend UX for HTTP 429

Security is not a feature — it’s a system.
