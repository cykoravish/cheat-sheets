# UI Tasks: Interview Cheat Sheet

These are the most asked "build this small UI" tasks. The interviewer checks how I structure the code, how I name things, and if I think about edge cases.

**How I approach every task:**
1. Write the **HTML structure** first.
2. Add **CSS** for the look and states (open, active, hidden).
3. Add **JavaScript** for the behavior. Keep the state in one place.
4. Mention edge cases at the end (keyboard, accessibility, many items).

Short versions are given here. First do them in plain JS, and then in React if asked.

---

## 1. Modal (dialog) with close on outside click and Escape key

**Idea:** A full-screen overlay holds the modal box. I toggle a CSS class to show or hide it.

```html
<button id="open">Open</button>
<div id="overlay" class="overlay">
  <div class="modal" role="dialog" aria-modal="true">
    <p>Hello!</p>
    <button id="close">Close</button>
  </div>
</div>
```

```css
.overlay { position: fixed; inset: 0; background: rgba(0,0,0,.5);
           display: none; align-items: center; justify-content: center; }
.overlay.open { display: flex; }
.modal { background: #fff; padding: 20px; border-radius: 8px; }
```

```js
const overlay = document.getElementById('overlay');
const openModal = () => overlay.classList.add('open');
const closeModal = () => overlay.classList.remove('open');

document.getElementById('open').addEventListener('click', openModal);
document.getElementById('close').addEventListener('click', closeModal);

overlay.addEventListener('click', (e) => {
  if (e.target === overlay) closeModal();          // click on the dark area only
});
document.addEventListener('keydown', (e) => {
  if (e.key === 'Escape') closeModal();
});
```

**Extra points:** The native `<dialog>` element with `showModal()` handles the Escape key by itself. I would also mention focus trapping and returning focus to the button.

---

## 2. Accordion (expand and collapse)

**Idea:** Toggle an `open` class. For a smooth animation, I animate `max-height` using the real content height (`scrollHeight`).

```html
<div id="accordion">
  <div class="item">
    <button class="header">Question 1</button>
    <div class="body"><p>Answer 1</p></div>
  </div>
</div>
```

```css
.body { max-height: 0; overflow: hidden; transition: max-height .3s ease; }
```

```js
document.getElementById('accordion').addEventListener('click', (e) => {
  const header = e.target.closest('.header');
  if (!header) return;

  const item = header.parentElement;
  const body = item.querySelector('.body');
  const isOpen = item.classList.toggle('open');
  body.style.maxHeight = isOpen ? body.scrollHeight + 'px' : null;
});
```

I use **event delegation**, so one listener handles all items. For "only one open at a time", I close the other items first.

---

## 3. Star rating

**Idea:** Show 5 stars. On hover, color up to the hovered star. On click, save the rating. On mouse out, go back to the saved rating.

```js
const stars = document.querySelectorAll('.star');
let rating = 0;

function paint(n) {
  stars.forEach((star, i) => star.classList.toggle('active', i < n));
}

stars.forEach((star, i) => {
  star.addEventListener('mouseover', () => paint(i + 1));
  star.addEventListener('mouseout', () => paint(rating));
  star.addEventListener('click', () => { rating = i + 1; paint(rating); });
});
```

---

## 4. Autocomplete (suggestions while typing)

**Idea:** Listen to the `input` event, filter my array, and show the matches. I use **debounce**, so I do not filter on every key press. (The `debounce` function is in `javascript.md`.)

```js
const items = ['apple', 'apricot', 'banana', 'mango', 'orange'];

input.addEventListener('input', debounce((e) => {
  const query = e.target.value.trim().toLowerCase();
  list.innerHTML = '';
  if (!query) return;

  items
    .filter(item => item.toLowerCase().startsWith(query))
    .forEach(item => {
      const li = document.createElement('li');
      li.textContent = item;                         // textContent, not innerHTML (safe from XSS)
      li.addEventListener('click', () => {
        input.value = item;
        list.innerHTML = '';
      });
      list.appendChild(li);
    });
}, 300));
```

If the data comes from an API, I also **cancel the old request** (`AbortController`) so an old slow response does not replace a new one.

---

## 5. Toast notifications (success, error, info, warning)

**Idea:** A fixed container holds all toasts. Each call creates a new toast element and removes it after 3 seconds. Many toasts can show at the same time.

```css
#toast-container { position: fixed; top: 16px; right: 16px;
                   display: flex; flex-direction: column; gap: 8px; }
.toast { padding: 12px 16px; border-radius: 6px; color: #fff; }
.toast.success { background: #2e7d32; }
.toast.error   { background: #c62828; }
.toast.info    { background: #1565c0; }
.toast.warning { background: #ef6c00; }
```

```js
function showToast(message, type = 'info') {
  const toast = document.createElement('div');
  toast.className = `toast ${type}`;
  toast.textContent = message;
  document.getElementById('toast-container').appendChild(toast);

  setTimeout(() => toast.remove(), 3000);
}

showToast('Saved!', 'success');
```

---

## 6. Infinite scroll

**Idea:** Put an empty "sentinel" element at the bottom of the list. When it comes into view, load the next page. `IntersectionObserver` is better than a scroll listener, because it is fast and does not run on every scroll.

```js
let page = 1;
let loading = false;

const observer = new IntersectionObserver(async ([entry]) => {
  if (!entry.isIntersecting || loading) return;
  loading = true;

  const items = await loadItems(page++);   // fetch the next page
  items.forEach(addItemToList);

  loading = false;
}, { rootMargin: '200px' });                // start loading a bit early

observer.observe(document.getElementById('sentinel'));
```

**Edge cases to mention:** stop when there is no more data, show a loading spinner, handle errors, and remove old items for very long lists (virtualization).

---

## 7. Stopwatch

**Idea:** Use `Date.now()` to calculate the time, not a counter, so there is no drift. Keep the elapsed time so I can resume.

```js
let start = 0, elapsed = 0, id = null;

function startWatch() {
  if (id) return;                                 // already running
  start = Date.now() - elapsed;
  id = setInterval(() => {
    elapsed = Date.now() - start;
    display.textContent = (elapsed / 1000).toFixed(2);
  }, 10);
}

function stopWatch() {
  clearInterval(id);
  id = null;
}

function resetWatch() {
  stopWatch();
  elapsed = 0;
  display.textContent = '0.00';
}
```

A timer is the same idea in reverse: count down from a number, and when it reaches 0, clear the interval and show an alert.

---

## 8. Lazy load an image

**Easy way:** `<img src="photo.jpg" loading="lazy" alt="...">`

**Manual way** (works everywhere and gives me control): keep the real URL in `data-src`, and set `src` when the image comes into view.

```js
const observer = new IntersectionObserver((entries, obs) => {
  entries.forEach(entry => {
    if (!entry.isIntersecting) return;
    const img = entry.target;
    img.src = img.dataset.src;
    obs.unobserve(img);
  });
});

document.querySelectorAll('img[data-src]').forEach(img => observer.observe(img));
```

---

## 9. Small JS tasks that are often asked

**Print 1 to 100 with a 2 second delay and show it in the UI:**

```js
for (let i = 1; i <= 100; i++) {
  setTimeout(() => { output.textContent = i; }, i * 2000);
}
```

I use `let` so each loop gets its own `i`. With `var`, all timers would print 101.

**Find the time taken by a function:**

```js
const t0 = performance.now();
myFunction();
console.log(`${performance.now() - t0} ms`);
```

**Clear all timeouts:** I keep the IDs in an array when I create them, and clear each one later.

```js
const timers = [];
timers.push(setTimeout(fn, 1000));
timers.forEach(clearTimeout);
```

---

## Last-Minute Revision

- Modal: overlay + class toggle. Close on `e.target === overlay` and on Escape.
- Accordion: toggle class, animate `max-height` with `scrollHeight`, use delegation.
- Star rating: paint on hover, save on click, repaint the saved value on mouse out.
- Autocomplete: `input` event + debounce + filter. Use `textContent`.
- Toast: fixed container, create and remove after 3 seconds.
- Infinite scroll: `IntersectionObserver` on a sentinel element, guard with a `loading` flag.
- Stopwatch: `Date.now()` and `setInterval`, keep elapsed time.
- Lazy load: `loading="lazy"` or `data-src` with an observer.