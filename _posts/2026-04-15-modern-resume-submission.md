---
layout: single
title: "Reverse Engineering an MCP-Based Job Application (Without Instructions)"
excerpt: "How I cracked an unconventional job application that gave me nothing but a URL"
categories: [dl-llm-systems]
tags: [engineering, oauth, debugging, systems, mcp, backend, ai]
header:
  teaser: /assets/img/mcp.jpg
---

Most job applications are simple - upload your CV, fill out a form, and click submit. But this one was completely different.

I came across a role for a **Forward Deployed Engineer** that had a very unusual ask:

<iframe src="https://www.linkedin.com/embed/feed/update/urn:li:share:7450164975286906880" height="789" width="504" frameborder="0" allowfullscreen="" title="Embedded post"></iframe>

> "Submit your application through this MCP endpoint. No instructions."

Just a URL. No documentation. No user interface. No hints.

Here's how I figured out the entire system from scratch and successfully submitted my application.

---

## What Is MCP?

Before diving in, let me quickly explain. **MCP (Model Context Protocol)** is a way for AI tools and services to communicate with each other. Think of it like a language that AI systems use to talk. In this case, the job application was hidden behind an MCP server - meaning I had to speak its language to apply.

---

## Step 1 - Hitting the Endpoint

I started with the simplest thing possible - just visiting the URL:

```bash
curl -i https://submit-cv-mcp.webrix.workers.dev/mcp
```

The response:

```json
{
  "error": "invalid_token",
  "error_description": "Missing or invalid access token"
}
```

In plain English: the door is locked, and I need a key (an access token) to get in.

---

## Step 2 - Reading the Response Headers

When a server sends back an error, it often includes **headers** (extra metadata) in the response. These headers had a critical clue:

```
www-authenticate: Bearer realm="OAuth",
resource_metadata=".../.well-known/oauth-protected-resource/mcp"
```

This is like the locked door having a sign that says: *"The key shop is at this address."* The server was pointing me toward an **OAuth** authentication system - a standard way websites let you prove who you are.

---

## Step 3 - OAuth Discovery

Following the clue from the headers, I visited the metadata URL:

```bash
curl https://submit-cv-mcp.webrix.workers.dev/.well-known/oauth-protected-resource/mcp
```

This told me where the authentication server lives. Then I fetched the full configuration:

```bash
curl https://submit-cv-mcp.webrix.workers.dev/.well-known/oauth-authorization-server
```

This gave me three important URLs:
- `/authorize` - where I go to prove who I am
- `/token` - where I exchange a temporary code for a real access token
- `/register` - where I register myself as a new "client" (application)

Think of it like discovering the full map of a building: *"Registration is on floor 1, authorization on floor 2, and key pickup on floor 3."*

---

## Step 4 - Registering a Client

Before I could authenticate, I needed to register. This is like signing up before you can log in:

```bash
curl -X POST https://submit-cv-mcp.webrix.workers.dev/register \
  -H "Content-Type: application/json" \
  -d '{"redirect_uris":["http://localhost"]}'
```

The server gave me a `client_id` and `client_secret` - think of these as my username and password for the authentication process.

---

## Step 5 - PKCE (Proof Key for Code Exchange)

OAuth uses an extra security step called **PKCE** (pronounced "pixie"). It ensures nobody can intercept and steal your login mid-process. Here's how it works in simple terms:

1. I generate a random secret string (the **verifier**)
2. I create a scrambled version of it (the **challenge**)
3. I send the challenge first, and prove I know the original verifier later

```bash
VERIFIER=$(openssl rand -base64 32 | tr -d '=+/')
CHALLENGE=$(echo -n $VERIFIER | openssl dgst -sha256 -binary | openssl base64 | tr '+/' '-_' | tr -d '=')
```

It's like telling someone *"I'll prove my identity later by completing this puzzle that only I can solve."*

---

## Step 6 - Authorization (Browser Flow)

Next, I had to open a URL in my browser with all the pieces put together:

```
/authorize?response_type=code
  &client_id=...
  &redirect_uri=http://localhost
  &code_challenge=...
  &code_challenge_method=S256
```

This is the part where you actually log in and give permission.

---

## Step 7 - The Unexpected Twist

Here's where things got interesting. Instead of a simple login form:

- **CSRF protection** appeared (a security measure to prevent fake requests)
- The system required a **browser session** with cookies
- It then **redirected me to GitHub** for login

This revealed that the system uses **GitHub as the identity provider**. In other words: *"Prove who you are by logging into GitHub."*

---

## Step 8 - Completing the OAuth Flow

The full sequence:
1. Visit the authorize URL
2. Get redirected to GitHub
3. Log in and approve access
4. Get redirected back with a temporary `code`

I captured this code from the redirect URL. This code is like a receipt - proof that I successfully authenticated.

---

## Step 9 - Getting the Access Token

Now I traded my temporary code for a real access token:

```bash
curl -X POST https://submit-cv-mcp.webrix.workers.dev/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=authorization_code&client_id=...&client_secret=...&code=...&redirect_uri=http://localhost&code_verifier=..."
```

The server verified everything matched (the code, the PKCE verifier, the client credentials) and gave me the golden key: an **access token**.

---

## Step 10 - Calling MCP (A New Problem)

With my access token in hand, I tried calling the MCP endpoint again. But I got another error:

```
Client must accept application/json and text/event-stream
```

This told me something important: **this is not a regular REST API.** It uses a streaming protocol, meaning the server can send data continuously rather than all at once - like a live conversation instead of sending letters.

---

## Step 11 - JSON-RPC and MCP Protocol

MCP uses a format called **JSON-RPC** - a simple way to call functions remotely. Instead of visiting different URLs for different actions (like REST), you send messages to one endpoint with a "method" field:

```json
{
  "jsonrpc": "2.0",
  "method": "...",
  "id": 1
}
```

Think of it like calling a receptionist and saying *"I'd like to speak to the submit-cv department"* instead of walking to different offices.

---

## Step 12 - Starting a Session

First, I needed to initialize a session:

```json
{
  "method": "initialize"
}
```

The response included a **session ID** - meaning the server remembers who I am across multiple requests. This is important because MCP is **stateful** (it keeps track of the conversation).

---

## Step 13 - Discovering What's Available

Next, I asked the server what tools (actions) are available:

```json
{
  "method": "tools/list"
}
```

Response:

```json
{
  "tools": [
    "get-job-description",
    "submit-cv"
  ]
}
```

Two tools: one to read the job description, and one to submit my CV.

---

## Step 14 - Hidden Requirements

When I tried to use `submit-cv`, it told me I needed two special codes:
- `resourceCode`
- `promptCode`

These weren't given to me directly - I had to fetch them from the MCP server using its **resources** and **prompts** features. It's like being told *"bring form A and form B to the submission window"* -- but first you have to find where forms A and B are stored.

---

## Step 15 - Fetching the Resource Code

```json
{
  "method": "resources/read",
  "params": {
    "uri": "submit-cv-mcp://cv/submission-code"
  }
}
```

This gave me the first required code.

---

## Step 16 - Fetching the Prompt Code

```json
{
  "method": "prompts/get",
  "params": {
    "name": "submit-cv"
  }
}
```

This gave me the second required code.

---

## Step 17 - Submitting the CV

With both codes in hand, I made the final submission:

```json
{
  "method": "tools/call",
  "params": {
    "name": "submit-cv",
    "arguments": {
      "resourceCode": "...",
      "promptCode": "..."
    }
  }
}
```

---

## Step 18 - One More Thing (Elicitation)

Just when I thought I was done, the server sent back a new kind of request:

```json
{
  "method": "elicitation/create"
}
```

It was asking me for:
- **Email** (required)
- **Phone** (optional)
- **Note** (optional)

This is called **elicitation** -- the server is asking the client for information mid-conversation, like a form popping up during a chat.

---

## Step 19 - Responding Correctly

This was the trickiest part. Instead of calling a new method, I had to **respond using the same request ID** that the server sent. This is how two-way (bidirectional) JSON-RPC works:

```json
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": {
    "email": "your@email.com",
    "phone": "",
    "note": "Looking forward to discussing this opportunity."
  }
}
```

It's like the server said *"Hey, before I finish, I need one more thing"* -- and I had to reply in exactly the right format.

---

## The Result

- CV submitted
- Contact details shared
- Challenge completed

---

## What This Challenge Actually Tested

This wasn't a coding test. It was a **thinking test**. Here's what it evaluated:

**System Thinking** - Can you follow signals and breadcrumbs instead of waiting for instructions?

**Debugging Skills** - Can you read error messages and headers to figure out what's going on?

**Protocol Understanding** - Do you know how OAuth 2.0, PKCE, JSON-RPC, and MCP work (or can you figure them out)?

**Adaptability** - Can you switch between tools (curl, browser, APIs, streaming) as needed?

**Real Engineering Behavior** - Can you experiment, fail, iterate, and solve?

---

## Key Takeaways

1. **Error messages are guidance, not noise** - they tell you exactly what's wrong and often hint at the solution
2. **HTTP headers contain critical clues** - always read them, especially on auth failures
3. **Not everything is REST** - protocols like JSON-RPC and MCP exist and are increasingly important in the AI era
4. **Authentication flows are stateful** - you need to track sessions, codes, and tokens carefully
5. **Understanding protocols matters more than knowing tools** - if you understand the "why," you can figure out the "how"

---

## Final Thoughts

This was one of the most interesting application processes I've encountered. Instead of asking *"What do you know?"*, it asked *"How do you think?"*

And that makes all the difference.
