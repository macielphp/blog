---
title: "How to Fix 404 Error in SPAs on Vercel: Client-Side Routing"
description: "Practical guide to fix NOT_FOUND error when accessing direct URLs in Single Page Applications deployed on Vercel"
tags: ["vercel", "spa", "react-router", "deployment", "routing"]
date: "2025-09-18"
author: "Maciel Alves"
readTime: "4 min"
image: "https://github.com/macielphp/blog/blob/main/assets/camera-capture-snap-shot-graphic.webp?raw=true"
---

# How to Fix 404 Error in SPAs on Vercel: Client-Side Routing

## The Problem

You've developed a Single Page Application (SPA) with React Router, deployed it on Vercel, and everything works perfectly when navigating through the application. However, when someone tries to directly access a URL like `/blog/read/my-article`, they get a **404: NOT_FOUND** error.

```bash
404: NOT_FOUND
Code: NOT_FOUND
ID: gru1::cl82c-1758241176185-4d5d01af6760
```

## Why does this happen?

The problem occurs because:

1. **SPAs use client-side routing** - React Router manages routes in the browser
2. **Vercel's server doesn't know about these routes** - It tries to find a physical file at the path `/blog/read/my-article`
3. **Since it doesn't exist**, it returns a 404 error

### Problem flow:

```
User accesses: /blog/read/my-article
     ↓
Vercel looks for: physical file at this path
     ↓
Doesn't find it → 404 NOT_FOUND
```

## The Solution

The solution is to configure Vercel to redirect all requests to `index.html`, allowing React Router to take control.

### 1. Configure vercel.json

Add the `rewrites` configuration to your `vercel.json`:

```json
{
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/static-build"
    }
  ],
  "outputDirectory": "dist",
  "rewrites": [
    {
      "source": "/(.*)",
      "destination": "/index.html"
    }
  ]
}
```

### 2. How it works

- **`rewrites`**: Redirects requests before serving files
- **`"source": "/(.*)"`**: Captures any route (regex that matches everything)
- **`"destination": "/index.html"`**: Always serves the SPA's main file

### 3. Fixed flow:

```
User accesses: /blog/read/my-article
     ↓
Vercel redirects to: /index.html
     ↓
React loads → React Router takes over → Correct route rendered
```

## Practical Example

### Before (problematic vercel.json):
```json
{
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/static-build"
    }
  ],
  "outputDirectory": "dist"
}
```

### After (fixed vercel.json):
```json
{
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/static-build"
    }
  ],
  "outputDirectory": "dist",
  "rewrites": [
    {
      "source": "/(.*)",
      "destination": "/index.html"
    }
  ]
}
```

## Testing the Solution

1. **Commit** the changes
2. **Push** to the repository
3. **Wait for automatic deployment** on Vercel
4. **Test** by directly accessing any route of your SPA

## Important Considerations

### ✅ Advantages:
- Direct URLs work perfectly
- SEO-friendly (important for crawlers)
- Link sharing works
- Improved user experience

### ⚠️ Precautions:
- The configuration affects **all** routes
- Make sure there are no conflicts with static files
- Test thoroughly after deployment

## Alternatives

If you need more control, you can use:

```json
{
  "rewrites": [
    {
      "source": "/api/(.*)",
      "destination": "/api/$1"
    },
    {
      "source": "/(.*)",
      "destination": "/index.html"
    }
  ]
}
```

This preserves API routes and redirects everything else to the SPA.

## Conclusion

The 404 error in SPAs on Vercel is a common problem that's easy to solve. The key is understanding that the server needs to "hand over" control to the client-side JavaScript.

With this simple configuration, your direct URLs will work perfectly, significantly improving user experience and link sharing capabilities.

---

**💡 Tip**: This configuration is standard for any SPA (React, Vue, Angular) deployed on Vercel. Always keep this configuration in your projects!