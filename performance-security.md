# Performance, Security and Accessibility: Interview Cheat Sheet

These are the "bar raiser" questions. They show how deep my frontend knowledge is. Only the most important ones are here.

---

## Performance

### 1. How do I reduce the initial load time?

I attack it from 3 sides.

- **Send less:** minify and compress files (Gzip or Brotli), compress images and use modern formats (WebP, AVIF), remove unused code with **tree shaking**.
- **Load later:** **code splitting** and lazy loading, so each page loads only the code it needs. Lazy load images below the fold. Use `defer` for scripts.
- **Send faster:** use a **CDN**, set good **caching**, and use `preload` and `preconnect` for important resources.

And I **measure first** with Lighthouse or the Network tab, so I fix the real problem and not a guess.

### 2. What is a CDN? What are its benefits?

A CDN (Content Delivery Network) is a group of servers around the world that keep **copies of my static files**. A user gets the files from the **nearest server**.

Benefits: **faster load** (less distance), **less load** on my main server, better handling of **traffic spikes**, and some **protection** against attacks like DDoS.

### 3. What caching options does the browser give me?

- **HTTP cache:** controlled by headers like `Cache-Control: max-age=31536000`, plus `ETag` for checking if a file changed (the server replies `304 Not Modified`). Static files with a hash in the name (`app.a1b2.js`) can be cached for a very long time.
- **Service Worker and Cache API:** my own code controls what is cached. It is used for offline support.
- **Memory cache:** in the current page session.
- **Storage:** `localStorage`, `sessionStorage` and IndexedDB for app data.

### 4. What are `preload`, `prefetch` and `preconnect`?

- **`preload`:** "Load this resource **now**, I need it soon on this page." (a font or hero image)
- **`prefetch`:** "Load this in the **background** with low priority, I may need it on the **next page**."
- **`preconnect`:** "Set up the connection (DNS + TCP + TLS) to this domain **early**." (a CDN or API domain)

```html
<link rel="preload" href="font.woff2" as="font" crossorigin />
<link rel="prefetch" href="next-page.js" />
<link rel="preconnect" href="https://cdn.example.com" />
```

### 5. What are CSR, SSR, SSG and ISR?

- **CSR (Client-Side Rendering):** the browser gets an empty page and JavaScript builds the content. Simple, but slow first paint and weak SEO.
- **SSR (Server-Side Rendering):** the server builds the HTML **on every request**. Fast first paint and good SEO, but the server does more work.
- **SSG (Static Site Generation):** the HTML is built **once at build time**. Very fast and cheap, good for blogs and docs. But the content is fixed until the next build.
- **ISR (Incremental Static Regeneration):** it works like SSG, but pages are **rebuilt in the background** after a set time. I get the speed of static pages with fresh content.

### 6. What are the Core Web Vitals?

- **LCP (Largest Contentful Paint):** how fast the **main content** loads. Good is under 2.5 seconds.
- **INP (Interaction to Next Paint):** how fast the page **responds** to clicks and taps. Good is under 200 ms. (It replaced FID.)
- **CLS (Cumulative Layout Shift):** how much the layout **jumps** while loading. Good is under 0.1. I fix it by setting `width` and `height` on images.

### 7. What is the Critical Rendering Path?

It is the steps the browser takes to show the first pixels: HTML becomes the **DOM**, CSS becomes the **CSSOM**, they combine into the **render tree**, then **layout**, **paint** and **composite**.

**CSS blocks rendering** and **JS blocks parsing**. So I keep the critical CSS small, use `defer` for scripts, and avoid big blocking files.

### 8. How do I handle 10,000 DOM elements?

I never render all of them. I use:

- **Virtualization (windowing):** render only the items visible on screen, plus a few extra. Libraries: `react-window`, `react-virtualized`.
- **Pagination** or **infinite scroll** to load data in parts.
- **Event delegation** instead of 10,000 listeners.
- **Debounce** for search and filter, and a `DocumentFragment` or batch updates to avoid many reflows.

### 9. What is Tree Shaking? What is a Source Map?

- **Tree shaking:** the bundler **removes code that is never used** (dead code) from the final bundle. It works with ES modules (`import` and `export`).
- **Source map:** a file that connects the **minified production code** back to my **original code**, so I can debug real errors in the browser.

### 10. Development build vs production build

- **Development:** big files, readable code, source maps, hot reload, extra warnings. Made for **debugging**.
- **Production:** **minified**, compressed, tree-shaken, split into chunks, with hashed file names. Made for **speed**.

### 11. `package.json` vs `package-lock.json`

- **`package.json`** lists my dependencies and scripts. Versions can be ranges (`^1.2.0`).
- **`package-lock.json`** saves the **exact version** of every package that was installed, so everyone on the team and the build server get the same result. I commit both files.

### 12. WebSocket vs SSE

- **WebSocket:** a **two-way** connection. Both client and server can send messages at any time. Good for chat and games.
- **SSE (Server-Sent Events):** a **one-way** connection from the server to the client. Simple, and it reconnects automatically. Good for live feeds, notifications and stock prices.

---

## Security

### 13. What is CORS?

By default, the browser blocks a page on one **origin** (protocol + domain + port) from reading data from another origin. This is the **same-origin policy**. **CORS** is the way for the **server** to say "I allow this other origin," using headers like `Access-Control-Allow-Origin`.

Important: CORS is enforced by the **browser**, and the fix is on the **server**. For some requests, the browser first sends a **preflight** `OPTIONS` request to check if it is allowed.

### 14. What is an HttpOnly cookie?

A cookie with the `HttpOnly` flag **cannot be read by JavaScript** (`document.cookie`). So if an attacker injects a script (XSS), they cannot steal it. I use it for session and auth tokens. I also add `Secure` (HTTPS only) and `SameSite` (helps against CSRF).

### 15. How do I prevent XSS?

**XSS (Cross-Site Scripting)** is when an attacker gets their **script to run on my page**, and it can steal data or tokens.

Prevention:
- **Never put user input in the page as HTML.** Use `textContent`, not `innerHTML`.
- **Escape or sanitize** user content (for example with DOMPurify). React escapes by default, so I avoid `dangerouslySetInnerHTML`.
- Add a **Content Security Policy (CSP)** header.
- Use **HttpOnly** cookies for tokens.

### 16. How do I prevent CSRF?

**CSRF (Cross-Site Request Forgery)** is when a bad website tricks the user's browser into sending a **request to my site** using the user's logged-in cookies.

Prevention:
- Use **`SameSite`** cookies (`Lax` or `Strict`).
- Use **CSRF tokens**, a random value that the attacker cannot know.
- Check the `Origin` header, and use re-authentication for sensitive actions.

---

## Accessibility and SEO

### 17. How do I make a website accessible? What are ARIA attributes?

- Use **semantic HTML** (`button`, `nav`, `main`), not `div` for everything.
- Give images an **`alt` text** and inputs a **`label`**.
- Make everything work with the **keyboard** and show a visible focus outline.
- Keep enough **color contrast** and do not use color alone to show meaning.
- Test with a **screen reader** (a tool that reads the page aloud for blind users).

**ARIA** attributes add extra meaning when HTML alone is not enough: `role="dialog"`, `aria-label="Close"`, `aria-expanded="true"`, `aria-hidden="true"`. The first rule of ARIA is: use a native HTML element if there is one.

### 18. What is SEO? How do I handle it in a SPA?

**SEO** means making my site easy for search engines to find and understand, so it ranks higher.

Basics: good `title` and `meta description`, semantic HTML, fast speed, mobile-friendly design, and clean URLs.

**In a SPA**, the first HTML is empty, so crawlers may not see the content. I fix it with **SSR or SSG** (Next.js), or pre-rendering, plus a correct title and meta tags for each page.

Two helper files:
- **`robots.txt`** tells crawlers **which pages they may or may not visit**.
- **`sitemap.xml`** gives crawlers **a list of all my important URLs**.

---

## Last-Minute Revision

- Initial load: send less, load later, send faster. Measure first.
- CDN: copies of files near the user.
- `preload` is for now. `prefetch` is for later. `preconnect` is for early connections.
- CSR is client. SSR is per request. SSG is at build time. ISR is static with background refresh.
- Core Web Vitals: LCP (loading), INP (response), CLS (layout shift).
- 10,000 elements: virtualization.
- CORS is enforced by the browser but fixed on the server.
- HttpOnly cookies cannot be read by JavaScript.
- XSS: do not inject user input as HTML. CSRF: `SameSite` and tokens.
- Accessibility: semantic HTML, alt, labels, keyboard, contrast.
- SPA SEO: SSR or SSG, `robots.txt`, `sitemap.xml`.