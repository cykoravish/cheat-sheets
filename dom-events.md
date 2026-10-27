# DOM and Events: Interview Cheat Sheet

Only the important questions. Bubbling, capturing, delegation, `preventDefault` and `stopPropagation` are already in `javascript.md`.

---

## 1. What is the difference between `DOMContentLoaded` and `load`?

- **`DOMContentLoaded`** fires when the HTML is fully read and the DOM is ready. It does **not** wait for images, CSS or iframes.
- **`load`** fires on `window` when **everything** is loaded: HTML, images, CSS, iframes.

So `DOMContentLoaded` comes first, `load` comes later.

```js
document.addEventListener('DOMContentLoaded', () => console.log('DOM ready'));
window.addEventListener('load', () => console.log('everything loaded'));
```

I use `DOMContentLoaded` for most things, because I do not want to wait for big images. If I use `defer` on my script, I do not even need it.

---

## 2. What is the difference between `onclick` and `addEventListener`?

| | `onclick` | `addEventListener` |
|---|---|---|
| Handlers on one element | only **one** (new one replaces old) | **many** |
| Options | none | `once`, `capture`, `passive`, `signal` |
| Remove | `el.onclick = null` | `removeEventListener` |

```js
btn.onclick = () => console.log('first');
btn.onclick = () => console.log('second');   // only this one runs

btn.addEventListener('click', () => console.log('first'));
btn.addEventListener('click', () => console.log('second'));   // both run
```

Useful options: `once: true` runs the handler one time only. `passive: true` promises I will not call `preventDefault`, which makes scrolling smoother.

To remove a listener I must pass the **same function reference**, so I give my handlers names.

---

## 3. What is the event object? What is `target` vs `currentTarget`?

When an event happens, the browser creates an **event object** and passes it to my handler. It has the details: `type`, `target`, `key`, `clientX`, and methods like `preventDefault()`.

- **`event.target`** is the element where the event **started** (the exact thing clicked).
- **`event.currentTarget`** is the element the listener is **attached to**.

```js
list.addEventListener('click', (e) => {
  console.log(e.target);          // the <li> I clicked
  console.log(e.currentTarget);   // the <ul> with the listener
});
```

This is the base of event delegation. I use `e.target` to find which child was clicked.

---

## 4. What is the difference between `keydown`, `keyup` and `keypress`?

- **`keydown`** fires when I press a key. It keeps firing if I hold the key, and it works for all keys (Shift, arrows, etc.).
- **`keyup`** fires once, when I release the key.
- **`keypress`** is **deprecated**. I do not use it.

```js
input.addEventListener('keydown', (e) => {
  if (e.key === 'Enter') console.log('Enter pressed');
});
```

`event.key` gives the key value (`"Enter"`). `event.code` gives the physical key (`"KeyA"`). To know when text in an input changes, I use the `input` event instead, because it also works for paste.

---

## 5. What is the difference between `querySelector` and `getElementsByClassName`?

| | `querySelector(All)` | `getElementsByClassName` |
|---|---|---|
| Input | any CSS selector | only a class name |
| Returns | first element, or a **NodeList** (`All`) | an **HTMLCollection** |
| Live or static | **static** snapshot | **live**, updates with the DOM |
| `forEach` | yes | no (use `Array.from`) |

```js
const all = document.querySelectorAll('.box');        // static
const live = document.getElementsByClassName('box');  // live
// after I add a new .box: all.length is the same, live.length is +1
```

In daily work I use `querySelector`, because it is flexible. The live collection can be a trap inside loops, because its length can change while I loop.

---

## Last-Minute Revision

- `DOMContentLoaded` is DOM ready. `load` is images and everything ready.
- `onclick` allows one handler. `addEventListener` allows many and has options.
- To remove a listener, use the same function reference.
- `target` is where the event started. `currentTarget` is where the listener sits.
- `keypress` is deprecated. Use `keydown` and `keyup`.
- `querySelectorAll` is static. `getElementsByClassName` is live.