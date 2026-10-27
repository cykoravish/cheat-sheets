# Web Fundamentals: Interview Cheat Sheet

The most important "frontend awareness" questions. Say each answer out loud in your own voice.

---

## 1. How does client-server work? What is an API?

The **client** (the browser) sends a **request**. The **server** processes it and sends back a **response**. The client is the one who asks, the server is the one who answers.

An **API** is a set of rules that lets two programs talk to each other. In web development, the frontend calls a backend API (like `GET /users`), and the backend sends back data, usually as JSON. The frontend does not need to know how the backend works inside. It only needs to know how to ask.

---

## 2. What is DNS? How does it work?

DNS is like the **phone book of the internet**. People remember names (`google.com`), but computers need IP addresses (`142.250.x.x`). DNS converts the name into the IP address.

How a lookup works, step by step:

1. The browser checks its **own cache**, then the **OS cache**.
2. If not found, it asks the **DNS resolver** (usually from the internet provider).
3. The resolver asks a **root server**, which points to the **TLD server** (`.com`).
4. The TLD server points to the **authoritative server** for the domain.
5. That server gives the **IP address**. The resolver saves it in cache and returns it to the browser.

---

## 3. What happens when I type a URL and press Enter?

This is the most asked question here. I explain it in steps:

1. **URL parsing:** the browser reads the protocol, domain and path. It checks its cache first.
2. **DNS lookup:** the domain is converted to an IP address.
3. **TCP connection:** the browser connects to the server with a three-way handshake.
4. **TLS handshake:** if it is HTTPS, they agree on encryption.
5. **HTTP request:** the browser sends `GET /` to the server.
6. **Server response:** the server sends back the HTML.
7. **Rendering:** the browser parses the HTML to build the **DOM** and the CSS to build the **CSSOM**. They combine into the **render tree**. Then the browser does **layout**, **paint** and **composite**.
8. **More requests:** while parsing, the browser finds CSS, JS and images, and requests them too. JavaScript runs, and the page becomes interactive.

---

## 4. What is TCP? What is the three-way handshake?

**TCP** is a protocol that sends data **reliably**. It makes sure the data arrives, arrives in the right order, and is re-sent if lost. (It is slower than UDP, but safe.)

**Three-way handshake** is how a TCP connection starts:

1. **SYN:** the client says "I want to connect."
2. **SYN-ACK:** the server says "OK, I am ready."
3. **ACK:** the client says "Great, let's start."

After this, both sides are sure the connection works, and data can flow.

---

## 5. What are HTTP, HTTPS and SSL/TLS? What is new in HTTP/2?

- **HTTP** is the protocol the browser uses to talk to the server. It sends data as **plain text**, so anyone in the middle can read it.
- **HTTPS** is HTTP **plus encryption** using SSL/TLS. The data is private and cannot be changed on the way. The server also proves its identity with a **certificate**. (SSL is the old name. Today we actually use TLS.)

**HTTP/2 improvements over HTTP/1.1:**

- **Multiplexing:** many requests share **one connection** at the same time. In HTTP/1.1, requests wait in line.
- **Header compression:** headers are smaller.
- **Binary format:** faster for computers to read.

HTTP/3 goes one step further and uses QUIC (over UDP) to avoid delays when packets are lost.

---

## 6. What is REST?

REST is a style for designing APIs. The server exposes **resources** (like users or orders) using URLs, and I use **HTTP methods** to work with them.

| Method | Use | Example |
|---|---|---|
| `GET` | read data | `GET /users` |
| `POST` | create data | `POST /users` |
| `PUT` / `PATCH` | update (full / partial) | `PATCH /users/5` |
| `DELETE` | delete data | `DELETE /users/5` |

Main rules: it is **stateless** (each request has all the information the server needs), and the response uses standard **status codes**.

**Status codes I remember:** `200` OK, `201` Created, `204` No Content, `301` Moved, `304` Not Modified, `400` Bad Request, `401` Unauthorized, `403` Forbidden, `404` Not Found, `500` Server Error.

---

## 7. Authentication vs authorization. Session vs JWT.

- **Authentication** = **who are you?** (login with a password.)
- **Authorization** = **what are you allowed to do?** (an admin can delete, a normal user cannot.)

Authentication always comes first.

**Session token:** After login, the server creates a **session**, stores it, and gives the browser a **session ID** (usually in a cookie). On every request, the browser sends the ID and the server looks it up. The server keeps the state.

**JWT (JSON Web Token):** After login, the server gives a **signed token** that has the user's information inside. The server does not need to store it. It only checks the signature on every request. A JWT has 3 parts: `header.payload.signature`.

Important points about JWT:
- It is **signed, not encrypted**. Anyone can read the payload, so I never put passwords or secrets in it.
- It is hard to cancel before it expires. So I use a short expiry time plus a refresh token.
- I store it in an **HttpOnly cookie** where possible, because `localStorage` can be stolen with XSS.

---

## 8. SPA vs traditional web apps (MPA)

- **Traditional (MPA):** every click asks the server for a **new full HTML page**. The page reloads. It is good for SEO and the first load.
- **SPA (Single Page App):** the browser loads **one HTML page** and JavaScript updates the content without reloading. It feels fast and smooth, like a native app (React, Vue, Angular). But the first load is heavier, and SEO needs extra work (SSR or pre-rendering).

---

## 9. What is CI/CD? How do I deploy an application?

**CI (Continuous Integration):** every time a developer pushes code, an automatic pipeline **builds the project and runs the tests**. Bugs are found early.

**CD (Continuous Delivery / Deployment):** after the tests pass, the code is **automatically released** to staging or production.

**How I deploy a frontend app:**
1. Run the build (`npm run build`). It creates optimized static files.
2. Upload the files to a host: Vercel, Netlify, S3 with CloudFront, or my own server with Nginx.
3. Point the domain to it with DNS, and enable HTTPS.
4. Set up CI/CD (like GitHub Actions) so every push to `main` deploys automatically.

---

## 10. Quick ones

- **Race condition:** two operations run at the same time and the result depends on who finishes first. Example: two API calls return out of order and the old response overwrites the new one.
- **Deadlock:** two tasks each wait for the other to release something, so both wait forever.
- **Concurrency vs parallelism:** concurrency is **handling many tasks** by switching between them (JavaScript does this with the event loop). Parallelism is **running many tasks at the exact same time** on multiple cores (Web Workers).
- **LAN / MAN / WAN:** a LAN is a small network like a home or office. A MAN covers a city. A WAN covers countries, and the internet is the biggest WAN.
- **Firewall:** a security guard for the network. It filters traffic by rules and blocks anything unwanted.
- **Pixel:** the smallest dot on a screen. A CSS pixel is a logical unit and may use more than one physical pixel on high-density screens.
- **CMS:** a Content Management System lets people manage website content without coding (WordPress, Strapi, Contentful).

---

## Last-Minute Revision

- Client sends a request, server sends a response. An API is the contract between them.
- DNS turns a name into an IP: cache, resolver, root, TLD, authoritative.
- Press Enter: URL, DNS, TCP, TLS, HTTP request, response, DOM + CSSOM, render.
- Three-way handshake: SYN, SYN-ACK, ACK.
- HTTPS is HTTP plus TLS. HTTP/2 has multiplexing and header compression.
- REST: resources + HTTP methods + stateless + status codes.
- Authentication is who you are. Authorization is what you can do.
- JWT is signed, not encrypted.
- SPA: one page, JS updates it. MPA: a new page per click.
- CI runs build and tests. CD releases the code.