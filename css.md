# CSS: Interview Cheat Sheet

Only the most important questions. Say each answer out loud in your own voice.

---

## 1. What is specificity in CSS?

Specificity decides **which rule wins** when many rules target the same element. From strongest to weakest:

1. **Inline style** (`style="..."`)
2. **ID** (`#box`)
3. **Class, attribute, pseudo-class** (`.box`, `[type="text"]`, `:hover`)
4. **Element, pseudo-element** (`div`, `::before`)

If two rules have the same specificity, the **last one** in the file wins. `!important` beats everything, but I avoid it, because it makes the code hard to maintain. I use it only to override third-party CSS.

```css
p { color: blue; }          /* 1 */
.text { color: green; }     /* 10 */
#intro { color: red; }      /* 100 - wins */
```

---

## 2. What is inheritance in CSS?

Some properties pass from a parent to its children automatically. Examples that inherit: `color`, `font-family`, `font-size`, `line-height`, `text-align`, `cursor`. Examples that do **not** inherit: `margin`, `padding`, `border`, `background`, `width`.

I can force it with `inherit`: `border: inherit;`.

---

## 3. What is the box model?

Every element is a box with 4 layers, from inside to outside: **content**, **padding**, **border**, **margin**.

By default (`content-box`), `width` is only the content, so padding and border are **added** on top. With `border-box`, `width` **includes** padding and border, which is easier to calculate. I always start with:

```css
* { box-sizing: border-box; }
```

---

## 4. How can I center a div horizontally and vertically? (3 ways)

```css
/* 1. Flexbox (my first choice) */
.parent { display: flex; justify-content: center; align-items: center; height: 100vh; }

/* 2. Grid (shortest) */
.parent { display: grid; place-items: center; height: 100vh; }

/* 3. Absolute + transform */
.parent { position: relative; height: 100vh; }
.child  { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); }
```

For only horizontal centering of a block with a fixed width: `margin: 0 auto;`.

---

## 5. What are the position values? What is the difference between `sticky` and `fixed`?

- **`static`** is the default. Normal flow.
- **`relative`** is moved from its normal place, but it still keeps its space. It is also the reference for absolute children.
- **`absolute`** is removed from the flow and placed relative to the nearest positioned parent.
- **`fixed`** is placed relative to the **screen**. It stays there when I scroll.
- **`sticky`** is a mix: it acts like `relative` until I scroll to a set point (like `top: 0`), then it sticks like `fixed`. But it stays **inside its parent**.

**`sticky` vs `fixed`:** `fixed` is always stuck to the screen and is removed from the flow. `sticky` stays in the flow and sticks only after reaching its point, and only inside its parent.

---

## 6. What is the difference between `opacity: 0`, `visibility: hidden` and `display: none`?

| | Visible? | Takes space? | Click / events work? |
|---|---|---|---|
| `opacity: 0` | no | **yes** | **yes** |
| `visibility: hidden` | no | **yes** | no |
| `display: none` | no | **no** | no |

Only `opacity` can be animated smoothly, so I use it for fade effects.

---

## 7. What is the difference between `rem` and `em`?

- **`rem`** is relative to the **root** (`html`) font size. The default is **16px**, so `1rem = 16px`.
- **`em`** is relative to the **parent's** font size (for `font-size`) or the element's own font size (for other properties like padding). Nested `em` values multiply, which can be confusing.

**How do I make `1rem = 20px`?**

```css
html { font-size: 20px; }     /* or 125% */
```

I prefer a percentage, because it respects the user's browser font size setting.

**Why use `rem`?** The sizes stay consistent, it is easy to scale the whole page by changing one value, and it is better for accessibility.

---

## 8. What do `calc()` and `clamp()` do?

- **`calc()`** does math with different units: `width: calc(100% - 40px);`
- **`clamp(min, preferred, max)`** keeps a value between a minimum and a maximum. It is great for fluid font sizes:

```css
h1 { font-size: clamp(1.5rem, 4vw, 3rem); }
```

---

## 9. Explain flexbox and its main properties.

Flexbox is a one-dimensional layout system (a row or a column). It makes alignment and spacing easy.

**On the container:**

- `display: flex`
- `flex-direction`: `row` or `column`
- `justify-content`: alignment on the **main** axis
- `align-items`: alignment on the **cross** axis
- `flex-wrap`: let items go to the next line
- `gap`: space between items

**On the items:**

- `flex-grow`: how much extra space the item takes
- `flex-shrink`: how much it shrinks when there is no space
- `flex-basis`: the starting size
- `flex: 1`: shortcut for equal sharing of space
- `align-self`, `order`

---

## 10. How does `z-index` work?

`z-index` decides which element is on top. It works on **positioned** elements (anything except `static`) and also on flex and grid items.

The important part is the **stacking context**. Some properties create a new one: `position` with `z-index`, `opacity` less than 1, `transform`, `filter`. A child **cannot go above** an element outside its parent's stacking context, even with a huge `z-index`. This is the most common reason why "z-index is not working".

The maximum value is **2147483647**.

---

## 11. What is margin collapsing?

When the **vertical margins** of two block elements touch, they **merge into one**, and the bigger one wins. They do not add up.

```css
.a { margin-bottom: 30px; }
.b { margin-top: 20px; }
/* gap between them is 30px, not 50px */
```

It also happens between a parent and its first or last child, if the parent has no padding, border or content in between.

It does **not** happen in flex or grid items, for floated elements, or for horizontal margins. I can stop it with `padding`, `border`, `overflow: hidden` or `display: flow-root` on the parent.

---

## 12. What are pseudo-classes and pseudo-elements?

- **Pseudo-class** (one colon) selects an element in a certain **state**: `:hover`, `:focus`, `:first-child`, `:nth-child(2)`, `:disabled`.
- **Pseudo-element** (two colons) selects a **part** of an element, or creates a fake one: `::before`, `::after`, `::first-letter`, `::placeholder`, `::selection`.

```css
.btn::after { content: '→'; margin-left: 4px; }
```

`::before` and `::after` need the `content` property to show up.

**First word of a paragraph?** CSS has `::first-letter` and `::first-line`, but nothing for the first word. For that I wrap the word in a `<span>`.

---

## 13. How do I show `...` for overflowing text? How do I limit the number of lines?

**One line with `...`:**

```css
.text {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

**Limit to N lines (for example 3):**

```css
.text {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
```

---

## 14. What does `:has()` do?

It is the **parent selector**. It selects an element **if it contains** something.

```css
.card:has(img) { padding: 0; }                 /* cards that have an image */
form:has(input:invalid) { border: 1px solid red; }
```

---

## 15. How do I check if a device supports hover?

I use a media query:

```css
@media (hover: hover) {
  .btn:hover { background: #333; }
}
```

On touch phones, the hover style is skipped, so the button does not stay "stuck" in the hover look after a tap.

---

## 16. What is the difference between `outline` and `border`?

- **`border`** is part of the box model. It takes space and changes the layout. I can set it per side.
- **`outline`** is drawn **outside** the element, takes **no space**, and cannot be set per side. I can move it with `outline-offset`.

I never remove the focus outline without giving a replacement, because keyboard users need it.

For a border only on the left and right: `border-left: 2px solid; border-right: 2px solid;` (or `border-inline`).

---

## 17. Quick ones

- **Ways to add CSS:** external file with `<link>` (best), `<style>` tag, inline `style=""`, and `@import`.
- **`<style>` in the body:** it works in browsers, but it is not valid HTML. I put it in the head.
- **Sibling selectors:** `A + B` is the **next** sibling. `A ~ B` is **all** following siblings.
- **Horizontal scroll only:** `overflow-x: auto; overflow-y: hidden;`
- **Hide the scrollbar:** `scrollbar-width: none;` and `::-webkit-scrollbar { display: none; }`
- **Rotate a div by 45 degrees:** `transform: rotate(45deg);`
- **Change the cursor:** `cursor: pointer;`
- **Disabled button:** `button:disabled { opacity: 0.5; cursor: not-allowed; }`
- **Vendor prefixes:** `-webkit-`, `-moz-`. They add browser-specific support for new properties. Tools like Autoprefixer add them for me.
- **Clearfix:** fixes a parent that collapses because its children are floated. Modern fix: `display: flow-root` on the parent.

---

## Last-Minute Revision

- Specificity: inline > id > class > element. Same score: last wins.
- Always use `box-sizing: border-box`.
- Center: flex with `justify-content` and `align-items`, or grid `place-items: center`.
- `sticky` stays inside its parent. `fixed` sticks to the screen.
- `opacity: 0` keeps space and clicks. `visibility: hidden` keeps space only. `display: none` keeps nothing.
- `rem` uses the root size (16px by default). `em` uses the parent.
- Stacking context is why `z-index` sometimes fails.
- Margin collapse: only vertical margins, bigger one wins.
- `:has()` is the parent selector.