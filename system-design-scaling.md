# System Design and Scaling: Interview Cheat Sheet

This is the **basic level** that a full-stack or web developer interview asks. I do not need deep distributed systems knowledge, but I should be able to explain how an app grows from 100 users to 1 million users.

Say each answer out loud in your own voice.

---

## 1. What is the difference between vertical and horizontal scaling?

- **Vertical scaling (scale up):** make **one server stronger**: more CPU, more RAM. It is simple, but there is a limit, it costs a lot, and it is a single point of failure.
- **Horizontal scaling (scale out):** add **more servers** and share the work between them. It has no hard limit and gives better reliability, but the app must be designed for it (stateless servers) and it needs a load balancer.

Most big systems scale horizontally.

---

## 2. What is a load balancer?

A load balancer sits **in front of my servers** and spreads the incoming requests across them. If one server dies, it sends traffic to the healthy ones (**health checks**).

```
Users --> Load Balancer --> Server 1
                       --> Server 2
                       --> Server 3
```

Common algorithms:
- **Round robin:** one by one, in turn.
- **Least connections:** send to the server with the fewest active connections.
- **IP hash:** the same user always goes to the same server.

Examples: Nginx, HAProxy, AWS ALB.

---

## 3. Why should servers be stateless?

A **stateless** server does **not keep user data in its own memory** between requests. Every request has everything the server needs (for example a JWT), or the data is read from a shared place (database or Redis).

Why it matters: if the session lives in the memory of Server 1, and the next request goes to Server 2, the user is logged out. With stateless servers, **any server can handle any request**, so I can add or remove servers freely.

---

## 4. What is caching? Where can I cache?

Caching means saving the result of something slow, so the next time I can return it **quickly**.

Layers, from the user to the database:
1. **Browser cache:** static files and API responses (`Cache-Control`).
2. **CDN:** copies of static files near the user.
3. **Application cache (Redis, Memcached):** results of expensive queries, sessions.
4. **Database cache:** query cache and indexes.

**Cache-aside** (the most common pattern):

```js
async function getUser(id) {
  const cached = await redis.get(`user:${id}`);
  if (cached) return JSON.parse(cached);              // cache hit

  const user = await db.query('SELECT * FROM users WHERE id = $1', [id]);
  await redis.set(`user:${id}`, JSON.stringify(user), 'EX', 300);   // keep for 5 minutes
  return user;
}
```

**The hard part is invalidation:** when the data changes, the cache can be old. I solve it with a **TTL** (expiry time), or by **deleting the cache key** when I update the data.

---

## 5. How do I scale a database?

I go step by step, from the cheapest option:

1. **Optimize first:** add **indexes**, fix slow queries, and avoid the N+1 problem.
2. **Connection pooling:** reuse database connections.
3. **Caching:** reduce the number of reads that reach the database.
4. **Read replicas:** one **primary** database handles writes. Many **replicas** copy the data and handle reads. It works well for read-heavy apps. The replicas can be a little behind (**replication lag**).
5. **Partitioning:** split one huge table into smaller parts.
6. **Sharding:** split the data across **many databases**, for example by `user_id`. It is powerful, but complex: cross-shard queries and joins become hard. I use it last.

---

## 6. What is a message queue? When do I use it?

A queue (RabbitMQ, Kafka, AWS SQS, BullMQ with Redis) lets one part of the system **send a task** and another part **process it later**. The two parts do not wait for each other.

Example: a user signs up. I do not make them wait while I send the welcome email and create a report. I put "send email" in a queue and reply to the user immediately. A **worker** picks the job from the queue and does it in the background.

Benefits: the app feels **faster**, it handles **traffic spikes** (the queue absorbs them), and **failed jobs can be retried**.

---

## 7. What is rate limiting? Why do I need it?

Rate limiting **limits how many requests** a user or IP can make in a period, for example 100 requests per minute. It protects the server from abuse, brute-force login attempts and accidental overload.

When the limit is reached, the server returns **HTTP 429 Too Many Requests**.

Common algorithms: **fixed window**, **sliding window**, and **token bucket** (a bucket that fills with tokens at a steady rate, and each request uses one token). I usually keep the counters in **Redis**, so that all servers share them.

---

## 8. What is the CAP theorem?

In a distributed system, when there is a **network partition** (a break in communication between servers), I can guarantee only **two out of three**:

- **C**onsistency: every read sees the latest write.
- **A**vailability: every request gets a response.
- **P**artition tolerance: the system keeps working even with network failure.

Network failures always happen, so in reality I choose between **consistency** (a bank) and **availability** (a social media feed). Many systems pick **eventual consistency**: data may be slightly old for a short time, but it becomes the same everywhere soon.

---

## 9. Monolith vs microservices

- **Monolith:** one codebase and one deployment. It is **simple** to build, test and deploy. It is the best start for most projects. It becomes hard when the team and the code get very big.
- **Microservices:** many small services, each with its own job and often its own database. Teams can work and deploy **independently**, and I can scale each service on its own. But it adds **complexity**: network calls, data consistency, monitoring and debugging.

My advice: **start with a well-structured monolith**, and split into services only when there is a real need.

---

## 10. How do I approach a system design question?

I follow these steps and think out loud:

1. **Clarify requirements:** what features? How many users? Read-heavy or write-heavy? What matters more, speed or consistency?
2. **Estimate the scale:** requests per second, data size.
3. **Define the API:** the main endpoints.
4. **Design the data model:** tables or collections and their relations.
5. **Draw the high-level design:** client, load balancer, servers, cache, database, queue, CDN.
6. **Find the bottlenecks:** and fix them with caching, replicas, queues, sharding.
7. **Talk about trade-offs:** there is no perfect design. I explain why I chose something.

---

## 11. Example: design a URL shortener

**Requirements:** make a short link from a long URL. When someone opens the short link, redirect to the long URL. It is **read-heavy** (many more redirects than creations).

**API:**
- `POST /shorten` with `{ longUrl }` returns `{ shortUrl }`
- `GET /:code` redirects to the long URL

**How to create the code:** save the row, get its auto-increment `id`, and convert it to **base62** (`a-z`, `A-Z`, `0-9`). With 7 characters I get about 3.5 trillion possible links.

**Data model:** `links (id, code, long_url, user_id, created_at, expires_at)` with an **index on `code`**.

**Design:**

```
User --> Load Balancer --> App Servers --> Redis cache --> Database (+ read replicas)
```

- Cache **popular codes in Redis**, because the same links get a lot of traffic.
- Use **read replicas** for the heavy read load.

**301 vs 302 redirect:** `301` (permanent) is cached by the browser, so it gives fewer hits to my server, but I cannot count clicks. `302` (temporary) always comes to my server, so I can track analytics.

---

## 12. What do I monitor in production?

- **Logs:** structured logs with request ids.
- **Metrics:** response time, error rate, CPU and memory, requests per second.
- **Alerts:** notify the team when something goes wrong.
- **Error tracking:** tools like Sentry.
- **Health check endpoint:** `GET /health`.

---

## Last-Minute Revision

- Vertical: bigger server. Horizontal: more servers (needs stateless design and a load balancer).
- Load balancer: round robin, least connections, health checks.
- Stateless: no user data in server memory. Use JWT or Redis.
- Caching layers: browser, CDN, Redis, database. Hard part is invalidation (TTL or delete on update).
- Database scaling order: indexes, caching, read replicas, partitioning, sharding.
- Queue: do slow work in the background, retry failures.
- Rate limiting: return 429. Use Redis counters.
- CAP: during a network failure, pick consistency or availability.
- Start with a monolith. Split when needed.
- Design steps: requirements, scale, API, data model, high-level design, bottlenecks, trade-offs.