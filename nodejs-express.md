# Node.js and Express: Interview Cheat Sheet

Only the most important questions. Say each answer out loud in your own voice.

---

## 1. What is Node.js? Why is it good for I/O?

Node.js is a runtime that lets me run **JavaScript outside the browser**, on a server. It uses Google's V8 engine.

It is good for I/O work (APIs, databases, file reading) because it is **non-blocking**. Node runs my JavaScript on **one thread**, but when it needs to do slow work like a database query or a file read, it hands the work to the system (libuv) and **keeps handling other requests**. When the work is done, the callback goes to the event loop. So one Node process can handle thousands of connections at the same time.

It is **not** a good fit for heavy CPU work (like image processing), because that blocks the single thread.

---

## 2. How does the Node.js event loop work?

It is similar to the browser event loop, but with **phases**. The main ones, in order:

1. **Timers:** `setTimeout` and `setInterval` callbacks
2. **Poll:** I/O callbacks (network, files). Node waits here for new events.
3. **Check:** `setImmediate` callbacks
4. **Close:** cleanup callbacks like `socket.on('close')`

Between every step, Node runs `process.nextTick` callbacks first, and then **Promise microtasks**.

```js
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('promise'));
console.log('sync');

// sync, nextTick, promise, then timeout and immediate
```

Order to remember: **sync code, then `nextTick`, then promises, then timers and immediate.** The order of `timeout` and `immediate` in the main file is not guaranteed. Inside an I/O callback, `setImmediate` always runs first.

---

## 3. What is blocking vs non-blocking code? How do I handle CPU-heavy tasks?

- **Blocking:** the code stops everything until it finishes. Example: `fs.readFileSync`, or a huge `for` loop. While it runs, no other request is served.
- **Non-blocking:** the code starts the work and continues. Example: `fs.readFile` with a callback, or `await fs.promises.readFile`.

For CPU-heavy work, I do not run it on the main thread. I use:
- **`worker_threads`** for heavy calculations,
- **`child_process`** or a separate service,
- a **job queue** (like BullMQ) that processes the work in the background.

---

## 4. What is the difference between CommonJS and ES Modules?

| | CommonJS | ES Modules |
|---|---|---|
| Syntax | `require()` and `module.exports` | `import` and `export` |
| Loading | synchronous | asynchronous, supports top-level `await` |
| Default in Node | `.js` files (older style) | when `"type": "module"` is set in `package.json`, or `.mjs` files |

```js
// CommonJS
const fs = require('fs');
module.exports = { hello };

// ES Modules
import fs from 'fs';
export const hello = () => {};
```

New projects use ES Modules, because it is the same as the browser standard.

---

## 5. What is Express? What is middleware?

Express is a small web framework for Node. It makes routing and handling requests simple.

**Middleware** is a function that runs **between the request and the response**. It gets `(req, res, next)`. It can read or change the request, end the response, or call `next()` to pass control to the next middleware. **The order matters**, because they run from top to bottom.

```js
const express = require('express');
const app = express();

app.use(express.json());                       // parse JSON body

app.use((req, res, next) => {                  // my own logger middleware
  console.log(req.method, req.url);
  next();
});

app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await getUser(req.params.id);
    if (!user) return res.status(404).json({ error: 'Not found' });
    res.json(user);
  } catch (err) {
    next(err);                                 // send error to the error handler
  }
});

// Error-handling middleware: 4 parameters, placed LAST
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: 'Something went wrong' });
});

app.listen(3000);
```

Common uses of middleware: logging, JSON parsing, authentication, CORS, rate limiting, validation.

---

## 6. How do I handle errors in Express?

- Use **one central error-handling middleware** (with 4 parameters) at the end.
- In **Express 4**, errors inside an `async` route are **not** caught by themselves. I use `try...catch` and call `next(err)`, or a small wrapper. **Express 5** forwards rejected promises to the error handler automatically.
- Send the right **status code** (400 for bad input, 401 for not logged in, 403 for not allowed, 404 not found, 500 for server error).
- Never send the stack trace to the client in production. Log it on the server.

```js
const asyncHandler = fn => (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);

app.get('/users', asyncHandler(async (req, res) => {
  res.json(await getUsers());
}));
```

---

## 7. How do I design a good REST API?

- Use **nouns** in URLs, not verbs: `GET /users`, not `/getUsers`.
- Use the right **HTTP method**: `GET` read, `POST` create, `PUT` or `PATCH` update, `DELETE` remove.
- Return the right **status codes** (200, 201, 204, 400, 401, 403, 404, 500).
- Support **pagination, filtering and sorting**: `GET /users?page=2&limit=20`.
- **Validate input** on the server (with Zod or Joi). Never trust the client.
- **Version** the API: `/api/v1/users`.
- Keep a **consistent error format**: `{ "error": "message" }`.
- Know **idempotency**: `GET`, `PUT` and `DELETE` give the same result if called many times. `POST` does not.

---

## 8. How does login and authentication work in a Node app?

1. **Register:** I never store a plain password. I **hash** it with `bcrypt`.
2. **Login:** compare the password with the hash. If it is correct, create a **JWT** (or a session).
3. **Send the token** in an **HttpOnly cookie** (safer) or the `Authorization` header.
4. **Protect routes** with an auth middleware that verifies the token.

```js
// Register
const hash = await bcrypt.hash(password, 10);

// Login
const ok = await bcrypt.compare(password, user.passwordHash);
if (!ok) return res.status(401).json({ error: 'Invalid credentials' });

const token = jwt.sign({ id: user.id }, process.env.JWT_SECRET, { expiresIn: '15m' });
res.cookie('token', token, { httpOnly: true, secure: true, sameSite: 'lax' });

// Auth middleware
function auth(req, res, next) {
  try {
    req.user = jwt.verify(req.cookies.token, process.env.JWT_SECRET);
    next();
  } catch {
    res.status(401).json({ error: 'Unauthorized' });
  }
}
```

I use a **short expiry** for the access token plus a **refresh token** to get a new one.

---

## 9. What are Streams and Buffers?

- A **Buffer** is a chunk of raw binary data in memory.
- A **Stream** processes data **piece by piece** instead of loading it all at once.

Example: sending a 2 GB file. With `readFile`, all 2 GB goes into memory. With a stream, only small chunks are in memory at a time.

```js
fs.createReadStream('big-file.mp4').pipe(res);
```

Types: Readable, Writable, Duplex and Transform. I use streams for large files, uploads and downloads.

---

## 10. How do I secure a Node and Express app?

- **Validate and sanitize** all input.
- Use **parameterized queries** to stop SQL injection.
- Hash passwords with **bcrypt**.
- Use **`helmet`** to set safe HTTP headers.
- Add **rate limiting** (`express-rate-limit`) to stop abuse and brute force.
- Configure **CORS** with only the origins I trust.
- Keep secrets in **environment variables** (`process.env`, `.env` file), never in the code or Git.
- Use **HTTPS**, keep dependencies updated (`npm audit`).

---

## 11. How do I scale a Node.js app?

- Node uses one CPU core per process. To use all cores I run **many processes**: the `cluster` module, or **PM2** in cluster mode.
- Put a **load balancer** (like Nginx) in front of many instances.
- Keep the server **stateless**: store sessions in Redis or use JWT, not in the memory of one process.
- Add **caching** (Redis), a **job queue** for slow tasks, and **database indexes**.

(More in `system-design-scaling.md`.)

---

## Last-Minute Revision

- Node: single thread + non-blocking I/O. Bad for heavy CPU work.
- Event loop order: sync, `nextTick`, promises, timers, immediate.
- Never use `*Sync` methods in a server. Use `worker_threads` for CPU work.
- CommonJS: `require`. ES Modules: `import`.
- Middleware: `(req, res, next)`. Order matters. Error handler has 4 params and goes last.
- REST: nouns, HTTP methods, status codes, pagination, validation, versioning.
- Auth: bcrypt for the password, JWT in an HttpOnly cookie, short expiry plus refresh token.
- Streams process data in chunks.
- Security: validation, bcrypt, helmet, rate limit, CORS, env variables.
- Scale: cluster or PM2, load balancer, stateless, Redis.