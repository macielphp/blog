---
title: "Do not use dotenv in Railway"
description: "How I discovered the error"
tags: ["railway", ".env", "dotenv", "deploy", "API", "Claude"]
date: "2026-03-06"
author: "Maciel Alves"
image: "https://github.com/macielphp/blog/blob/main/assets/railway_dotenv-config.webp?raw=true"
readTime: "2 min"
---

# Do not use dotenv in Railway

## The crash
I tried several times to deploy the project's API in Railway. But, I got errors saying that RESEND_API_KEY was undefined.

<img src="./../assets/railway-deploy-crash.png"/>

I asked Claude and Grok to help me solve the problem. They also told me I had to verify if the Railway Service API variable value was correct. 

- "Yes", I said. 

<img src="./../assets/self-dev-room-railway-variables.png" />

- Would it be wrong? 
- "No". 
- Why?
- Because the local API and Supabase DB worked before with the same code. 

I went to Resend to check the activity of the API key and it looked good. No errors.

<img src="./../assets/resend-api-key-activity.png" />

Claude even helped me with some PORT hard code. 

## Questioning code
The first thing I questioned was whether the path configured in dotenv.config was correct.

```js
dotenv.config({
    path: '../../.env'
});
```

What if Railway ran the code and dotenv.config pointed to a different path than expected? 

Even though it worked locally, it might not work in Railway because of internal environment differences that I wasn't aware of.

So I commented out the dotenv.config, committed the change, pushed the code, and redeployed.

<img src="./../assets/commit-comment-out-unused-imports-and-dotenv-configuration-in-userController.png" />

It worked.

<img src="./../assets/self-dev-room-success-deploy.png" />

## Lesson
When deploying on Railway, you usually should not manually load .env files with custom paths.

Railway already injects environment variables into process.env, so overriding or redirecting .env paths can break the configuration.

In my case, removing dotenv.config fixed the issue.

Check the repository for newer updates and improvements: https://github.com/macielphp/self-development-room

