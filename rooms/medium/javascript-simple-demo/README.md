# JavaScript: Simple Demo

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Programming Fundamentals

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included. Code examples below are generic
> reference examples, not captures from a completed session.

## Overview

JavaScript is the scripting language that runs natively in every web browser, and it is also the
language behind server-side platforms like Node.js. A "simple demo" room like this one is normally
aimed at getting a beginner comfortable with the core building blocks — variables, functions,
conditionals, loops, and how JavaScript interacts with the Document Object Model (DOM) — before moving
into more security-relevant territory such as client-side validation bypass and cross-site scripting
(XSS). This guide covers those fundamentals and explains why understanding JavaScript execution
context matters for anyone doing web application security work.

## Core Concepts

### Variables and types

Modern JavaScript uses `let` and `const` for variable declarations (the older `var` keyword has
function scope rather than block scope and is generally discouraged today):

```javascript
const siteName = "example";      // block-scoped, cannot be reassigned
let attempts = 0;                // block-scoped, mutable
attempts += 1;
console.log(attempts);           // 1
```

JavaScript is dynamically typed — a variable's type is determined at runtime, and the primitive types
are `string`, `number`, `bigint`, `boolean`, `undefined`, `null`, and `symbol`, with `object` covering
everything else (including arrays and functions).

### Functions

Functions can be declared conventionally or as arrow functions, which do not bind their own `this`:

```javascript
function add(a, b) {
  return a + b;
}

const multiply = (a, b) => a * b;

console.log(add(2, 3));       // 5
console.log(multiply(2, 3));  // 6
```

### Conditionals and comparison

A common beginner pitfall is confusing `==` (loose equality, which performs type coercion) with `===`
(strict equality, which does not):

```javascript
console.log(0 == "0");   // true  — coerced comparison
console.log(0 === "0");  // false — different types, no coercion
```

Security-conscious JavaScript style guides, including MDN's own guidance, recommend `===`/`!==` by
default specifically because loose equality's coercion rules are a frequent source of logic bugs.

### Loops and arrays

```javascript
const users = ["alice", "bob", "carol"];

for (const user of users) {
  console.log(`Hello, ${user}`);
}

// Functional style
users.forEach((user) => console.log(user.toUpperCase()));
```

### The DOM and event handling

What sets JavaScript apart from a general-purpose scripting language is its access to the DOM — the
browser's live, in-memory representation of the page. This is where "simple demo" rooms typically
show off interactivity:

```html
<button id="check-btn">Check</button>
<script>
  document.getElementById("check-btn").addEventListener("click", () => {
    const input = document.getElementById("password").value;
    if (input.length < 8) {
      alert("Password too short");
    }
  });
</script>
```

This snippet is deliberately illustrative of a security anti-pattern: the length check happens only in
the browser. Nothing stops a user from disabling JavaScript, editing the DOM with browser developer
tools, or sending a request directly to the server with curl or Burp Suite, bypassing the check
entirely. This single idea — that client-side JavaScript is a UX layer, not a trust boundary — is the
concept most "simple demo" rooms are building toward.

### Node.js and `console.log`

Outside the browser, Node.js runs JavaScript with access to the filesystem, network sockets, and
process environment through its standard library (`fs`, `http`, `process`, etc.), which is why
JavaScript (via Node.js) shows up constantly in modern tooling, including security tooling, CLI
utilities, and browser automation frameworks like Puppeteer and Playwright.

## Why It Matters for Security

The single most important lesson from any JavaScript fundamentals room, from a security perspective,
is the **client-side trust boundary**. Because JavaScript executes entirely under the control of
whoever holds the browser, any logic implemented purely in client-side JavaScript — input length
checks, "disabled" buttons, hidden form fields, feature flags — can be inspected, modified, or skipped
by an attacker. Server-side validation is mandatory; client-side JavaScript checks are, at best, a
usability convenience.

The other major implication is **cross-site scripting (XSS)**. Because JavaScript can read and rewrite
the DOM of the page it runs on, an attacker who can get arbitrary JavaScript to execute in a victim's
browser (for example, by injecting a `<script>` tag or event handler into unsanitized user input that
gets reflected back into the page) can steal session cookies, perform actions as the victim, or
deface the page. OWASP catalogs this as one of the most common and impactful web vulnerability
classes. Understanding how JavaScript actually manipulates the DOM — `innerHTML`, `document.write`,
event handler attributes — is a prerequisite for understanding *why* those specific APIs are the ones
XSS payloads target, and why frameworks that auto-escape output (React, Angular's default binding)
reduce this risk.

## Common Pitfalls / Misconfigurations

- **Trusting client-side validation** — any check written in browser JavaScript must be duplicated on
  the server, because the client cannot be trusted.
- **Using `innerHTML` with untrusted data** — assigning unsanitized user input to `innerHTML` (rather
  than `textContent`) is one of the most direct routes to a DOM-based XSS vulnerability.
- **Loose equality (`==`) bugs** — type coercion can produce unexpected `true` results, which has
  historically caused authentication and authorization logic errors.
- **Exposing secrets in client-side code** — API keys or credentials embedded in JavaScript shipped to
  the browser are visible to anyone who views page source or the network tab.
- **Ignoring Content Security Policy (CSP)** — CSP headers are one of the primary browser-level
  defenses against XSS, and are frequently misconfigured or omitted entirely.

## Related TryHackMe Rooms in This Series

- [Database SQL Basics](../../easy/database-sql-basics/README.md) — the server-side analogue: input
  that isn't trusted or validated properly at the server/database boundary leads to SQL injection,
  the same class of "trust boundary" mistake discussed above for the client side.
- [Python: Simple Demo](../../easy/python-simple-demo/README.md) — a parallel fundamentals room for a
  server-side/general-purpose scripting language.

## References

- [MDN Web Docs — JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
- [MDN — Equality comparisons and sameness](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Equality_comparisons_and_sameness)
- [MDN — Element.innerHTML security considerations](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML#security_considerations)
- [OWASP — Cross Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/)
- [OWASP — Content Security Policy Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)
- [Node.js Documentation](https://nodejs.org/en/docs/)
