# HTML: Interview Cheat Sheet

Only the most important questions. Say each answer out loud in your own voice.

---

## 1. What are semantic and non-semantic elements?

**Semantic** elements tell the **meaning** of the content. Examples: `<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<aside>`, `<footer>`, `<button>`.

**Non-semantic** elements tell nothing about the content. Examples: `<div>` and `<span>`.

Why do I use semantic elements?

- Better **accessibility**: screen readers understand the page structure.
- Better **SEO**: search engines understand the content.
- The code is easier to read and maintain.

---

## 2. What is the difference between inline and block elements?

- **Block** elements start on a **new line** and take the **full width**. I can set width, height and margins. Examples: `div`, `p`, `h1`, `ul`, `section`.
- **Inline** elements stay in the line and take **only the space they need**. Width and height do not work, and top/bottom margins have no effect. Examples: `span`, `a`, `strong`.
- **Inline-block** is a mix: it sits in the line like inline, but width and height work.

---

## 3. What is the `<!DOCTYPE html>` declaration? Is HTML case-sensitive?

`<!DOCTYPE html>` tells the browser to read the page in **standards mode**. Without it, the browser goes into **quirks mode**, where it follows old rules and the layout can look different in different browsers. It is not an HTML tag, it is an instruction to the browser.

HTML is **not case-sensitive** for tags and attributes. `<DIV>` works like `<div>`. But I always write lowercase, because that is the standard.

---

## 4. What is the `<head>` tag?

It holds the **information about the page**, not the content that users see. It has:

- `<title>`: the text on the browser tab
- `<meta charset="UTF-8">` and `<meta name="viewport" ...>`: encoding and mobile layout
- `<meta name="description">`: used by search engines
- `<link>`: CSS files and the favicon
- `<script>`: JavaScript files (with `defer`)

---

## 5. How can I take input from the user? How do I make an upload button?

I use a `<form>` with these elements:

- `<input>` with different types: `text`, `email`, `password`, `number`, `date`, `checkbox`, `radio`, `file`, `range`
- `<textarea>` for long text
- `<select>` with `<option>` for a dropdown
- A `<label>` for each field, which is good for accessibility

```html
<form>
  <label for="name">Name</label>
  <input id="name" type="text" required />
  <button type="submit">Send</button>
</form>
```

**File upload:**

```html
<input type="file" />
```

**Allow only `.jpg` files:**

```html
<input type="file" accept=".jpg,.jpeg" />
<!-- or -->
<input type="file" accept="image/jpeg" />
```

**Gotcha:** `accept` is only a **hint**. The user can still choose another file type. So I always validate the file in JavaScript and also on the server.

---

## 6. How can I refresh the page every 60 seconds using HTML?

I use a meta tag in the `<head>`:

```html
<meta http-equiv="refresh" content="60" />
```

This reloads the full page. For a better user experience, I would use JavaScript (`setInterval` with `fetch`) or WebSockets to update only the data.

---

## 7. What is the `<picture>` tag and `srcset`? What is `loading="lazy"`?

**`srcset`** lets the browser pick the best image size for the screen.

```html
<img
  src="small.jpg"
  srcset="small.jpg 480w, medium.jpg 800w, large.jpg 1200w"
  sizes="(max-width: 600px) 480px, 800px"
  alt="Product"
/>
```

**`<picture>`** lets me give **different image formats** or different images for different screens (art direction). The browser uses the first source it supports.

```html
<picture>
  <source srcset="photo.avif" type="image/avif" />
  <source srcset="photo.webp" type="image/webp" />
  <img src="photo.jpg" alt="Photo" />
</picture>
```

**`loading="lazy"`** loads the image only when it is near the screen. It saves data and makes the first load faster. **`loading="eager"`** (the default) loads it right away. I use `eager` for the main image at the top of the page and `lazy` for the rest. I also set `width` and `height` to avoid the layout jumping.

---

## 8. How do I change the image in the browser tab title?

That image is called the **favicon**. I add it in the `<head>`:

```html
<link rel="icon" href="favicon.ico" type="image/x-icon" />
```

---

## 9. Quick ones

- **`<noscript>`:** its content is shown only when JavaScript is turned off or not supported.
- **`<table>`:** only for **data in rows and columns**, never for page layout. `rowspan` merges cells vertically and `colspan` merges cells horizontally.
- **`<marquee>`:** an old scrolling text tag. It is **deprecated**, so I do not use it.

---

## Last-Minute Revision

- Semantic tags help accessibility and SEO. `div` and `span` have no meaning.
- Block takes full width. Inline takes only its content. Inline-block is a mix.
- No DOCTYPE means quirks mode. HTML tags are case-insensitive.
- `<head>` holds title, meta, links and scripts.
- `accept=".jpg"` is a hint only, so I validate again.
- Refresh page: `<meta http-equiv="refresh" content="60">`.
- `srcset` for sizes, `<picture>` for formats, `loading="lazy"` for off-screen images.
- Favicon: `<link rel="icon">`.