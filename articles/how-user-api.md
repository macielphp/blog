# How to Design URLs for a Product API  
### A Practical Guide with the Self-Development-Room Case Study

## Introduction

One of the most common mistakes when building APIs is designing URLs based on **database tables** instead of **product behavior**.

A well-designed **Product API URL** should reflect:
- What the user wants to do
- The flow of the application
- The domain language of the product

This article explains how to design URLs for a **Product API**, using the **Self-Development-Room** learning platform as a real-world example.

---

## 1. What Is a Product API URL?

A **Product API URL** represents a **feature**, not a table.

Bad question:
> “What table am I accessing?”

Good question:
> “What experience am I enabling for the user?”

### Example

❌ Table-oriented (CRUD-style):
```http
GET /languages
GET /seasons
GET /lessons
```

✅ Product-oriented:
```http
GET /app/languages
GET /app/seasons?language=1
GET /app/lessons/:lessonId
```

---

## 2. Start with a Product Boundary

The first decision is defining **who the API is for**.

In **Self-Development-Room**, there are two clear products:
- **Admin Panel**
- **Learning App (User)**

### URL Boundaries

```http
/admin/*   → Admin product
/app/*     → User product
```

This separation:
- Improves security
- Simplifies rate limiting
- Clarifies responsibility

---

## 3. Use Nouns, Not Verbs

Product APIs should use **nouns that represent concepts**, not actions.

### Example

❌ Verb-based:

```http
/getLanguages
/loadSeasons
/doLesson
```

✅ Noun-based:
```http
/languages
/seasons
/lessons
```

The **HTTP method** already expresses the action.

---

## 4. Represent Relationships in the URL

Product APIs must reflect **natural relationships** between entities.

### Database Relationship

```

Language → Season → Lesson → Question → Alternative

````

### Product API Representation

```http
GET /app/seasons?language=1
GET /app/lessons?season=3
GET /app/lessons/10
````

Why query parameters?

* Cleaner URLs
* Easier filtering
* More flexible than deep nesting

---

## 5. When to Use Route Params vs Query Params

### Route Parameters (`:id`)

Use when:

* Accessing a **single resource**
* The ID is mandatory

```http
GET /app/lessons/10
```

### Query Parameters (`?`)

Use when:

* Filtering
* Optional relationships

```http
GET /app/seasons?language=1
GET /app/lessons?season=3
```

---

## 6. Avoid Leaking Database Structure

Users should never see:

* Table names
* Foreign keys
* Join logic

❌ Database-driven:

```
/app/questions?lesson_id=3
```

✅ Product-driven:

```
/app/lessons/3/questions
```

Or even better:

```
/app/lessons/3
```

(Questions are part of the lesson experience.)

---

## 7. Group URLs by Feature, Not by Table

In **Self-Development-Room**, routes are grouped by **feature**:

```
/app/languages
/app/seasons
/app/lessons
/app/activities
```

Not by database structure.

### Why `/activities`?

Because:

* Users are not “answering alternatives”
* Users are **performing learning activities**

This is domain language.

---

## 8. Designing URLs for User Actions

Product APIs often need **action-based endpoints**, but still without verbs.

### Example: Submitting Answers

❌ Verb-heavy:

```
POST /submitAnswers
```

✅ Product-oriented:

```
POST /app/activities
```

The payload defines the action:

```json
{
  "lessonId": 10,
  "answers": [
    { "questionId": 1, "alternativeId": 3 }
  ]
}
```

---

## 9. Authentication and Rate Limiting in URLs

URLs should not contain:

* User IDs
* Tokens
* Hashes

Authentication belongs to **headers**, not URLs.

### Correct

```http
Authorization: Bearer <token>
GET /app/lessons/10
```

Rate limiting is applied by:

* Product boundary (`/app`)
* User identity
* IP fallback

---

## 10. Final URL Design for Self-Development-Room

### User Product API

```http
GET  /app/languages
GET  /app/seasons?language=1
GET  /app/lessons?season=2
GET  /app/lessons/:lessonId
POST /app/activities
GET  /app/progress
```

### Admin Product API

```http
POST   /admin/languages
POST   /admin/seasons
POST   /admin/lessons
POST   /admin/questions
POST   /admin/alternatives
```

---

## 11. Key Principles to Remember

✔ Design URLs around **user intent**
✔ Separate **admin** and **user** products
✔ Prefer **nouns over verbs**
✔ Hide database complexity
✔ Let HTTP methods express actions
✔ Keep URLs predictable and readable

---

## Conclusion

A well-designed Product API URL is part of the **user experience**.

In **Self-Development-Room**, URLs are not just endpoints — they represent:

* Learning flow
* Product logic
* Business intent

Designing URLs this way makes your API:

* Easier to use
* Easier to evolve
* Easier to secure

---

**Project:** Self-Development-Room
**Topics:** API Design, REST, Product API, Backend Architecture, Node.js

