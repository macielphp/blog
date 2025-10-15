---
title: "Understanding Callbacks and Error Handling: A Practical Case"
description: "Learn how to make data 'travel' between functions using callbacks and implement efficient error handling with practical examples"
tags: ["javascript", "callbacks", "async", "error-handling", "tutorial"]
date: "2025-09-08"
author: "Maciel Alves"
image: "https://images.unsplash.com/photo-1555066931-4365d14bab8c?w=800&h=400&fit=crop"
readTime: "7 min"
---

# Understanding Callbacks and Error Handling: A Practical Case

## The Problem: How to Make Data "Travel" Between Functions

### Initial Scenario
Imagine you have an application that fetches books from an API and displays them on the screen. You have two main functions:

```javascript
// main.js - Entry point
fetchBooks(displayBooksOnScreen)

// fetchBooks.js - Fetches data from API
function fetchBooks(functionParam) {
    // ... code ...
    functionParam(booksObj)  // Calls the function passing booksObj
}

// displayBooksOnScreen.js - Displays data on screen
function displayBooksOnScreen(booksObj) {
    // Expects to receive booksObj as parameter
}
```

### The Initial Confusion
**Common question:** "How will `displayBooksOnScreen` receive `booksObj` if I don't pass any parameter to `fetchBooks`?"

**Answer:** You're thinking **synchronously** when you should be thinking **asynchronously**.

## Lesson 1: Understanding the Callback Pattern

### The Data Flow
1. **`fetchBooks`** is responsible for **fetching** data from the API
2. **`displayBooksOnScreen`** is responsible for **displaying** data on the screen
3. **`booksObj`** is the **data** that needs to "travel" from one function to another

### The Chef Analogy
Imagine you have:
- A **chef** (`fetchBooks`) who will fetch ingredients
- A **recipe** (`displayBooksOnScreen`) that needs the ingredients
- You give the recipe to the chef and say: "when you have the ingredients, execute this recipe"

### How It Works in Practice
```javascript
// ❌ WRONG - Executes immediately
fetchBooks(displayBooksOnScreen())

// ✅ CORRECT - Passes the function to be executed later
fetchBooks(displayBooksOnScreen)
```

### The Real Flow
1. `main.js` calls `fetchBooks(displayBooksOnScreen)`
2. `fetchBooks` receives the `displayBooksOnScreen` function as parameter
3. `fetchBooks` fetches data from the API
4. `fetchBooks` calls `displayBooksOnScreen(booksObj)` passing the data
5. `displayBooksOnScreen` receives the data and displays it

## Lesson 2: Error Handling with Callbacks

### The Error Problem
```javascript
// Current code - only logs the error
async function fetchBooks(functionParam) {
    try {
        let booksObj = await getFetchBooksFromAPI(apiEndpoint)  
        if (booksObj) {
            functionParam(booksObj)  // ✅ Success
        }
    } catch (error) {
        console.log(error)  // ❌ Only logs, doesn't inform the user
    }
}
```

### Error Handling Strategies

#### Strategy 1: Two Callbacks (Success + Error)
```javascript
// How to use:
fetchBooks(
    displayBooksOnScreen,        // Function for success
    displayErrorMessage          // Function for error
)

// Implementation:
async function fetchBooks(onSuccess, onError) {
    try {
        let booksObj = await getFetchBooksFromAPI(apiEndpoint)  
        if (booksObj) {
            onSuccess(booksObj)     // ✅ Success
        }
    } catch (error) {
        onError(error)              // ❌ Error
    }
}
```

**Pros:** Clear, explicit
**Cons:** More parameters, can get confusing

#### Strategy 2: One Callback that Receives Status
```javascript
// How to use:
fetchBooks(displayResult)

// Implementation:
async function fetchBooks(callback) {
    try {
        let booksObj = await getFetchBooksFromAPI(apiEndpoint)  
        if (booksObj) {
            callback({ success: true, data: booksObj })
        }
    } catch (error) {
        callback({ success: false, error: error })
    }
}

// Function that processes the result:
function displayResult(result) {
    if (result.success) {
        displayBooksOnScreen(result.data)
    } else {
        displayErrorMessage(result.error)
    }
}
```

**Pros:** Single callback, flexible
**Cons:** Need to check status

#### Strategy 3: Promise (More Modern)
```javascript
// How to use:
fetchBooks()
    .then(books => displayBooksOnScreen(books))
    .catch(error => displayErrorMessage(error))

// Implementation:
async function fetchBooks() {
    try {
        let booksObj = await getFetchBooksFromAPI(apiEndpoint)  
        return booksObj
    } catch (error) {
        throw error  // Re-throws the error to be caught by .catch()
    }
}
```

**Pros:** Modern standard, clean
**Cons:** Changes the way you call the function

## Lesson 3: Thinking Like a User

### Important Questions
1. **What should the user see when there's an error?**
   - Generic message: "Error loading books"
   - Specific message: "Connection error" or "Server unavailable"

2. **Where to display the error message?**
   - In the same place as the books
   - In a specific location for errors
   - In a modal/popup

3. **What to do after the error?**
   - Retry automatically
   - Show "Try again" button
   - Just inform the error

### Practical Example
```javascript
function displayErrorMessage(error) {
    const booksElement = document.getElementById('books')
    booksElement.innerHTML = `
        <div class="error">
            <p>Oops! Could not load the books.</p>
            <p>Please try again in a few moments.</p>
            <button onclick="location.reload()">Try Again</button>
        </div>
    `
}
```

## Conclusion: Mental Pattern for Callbacks

### Questions You Should Ask
1. **Who has the data?** → `fetchBooks`
2. **Who needs the data?** → `displayBooksOnScreen`
3. **How to make the data travel?** → Pass a function that will be called with the data

### Mental Pattern
```
Data → Function that Processes → Function that Uses the Result
```

### Thought Summary
Error handling is about **communication**:
- **Communicating** with the user when something goes wrong
- **Communicating** clearly and usefully
- **Communicating** available options (try again, etc.)

**The key is to think: "If I were the user, what would I like to see when something went wrong?"**

---

*This article demonstrates how to think about callbacks and error handling through a real practical case, showing different strategies and their implications.*