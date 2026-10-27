# Polyfills: Interview Cheat Sheet

**Already in `javascript.md`:** map, reduce, bind, Promise.all, debounce, throttle. They are not repeated here.

**What is a polyfill?** It is code that adds a modern feature to an older browser that does not have it. I check if the feature is missing, and then I add my own version on the `prototype`.

**How I answer in the interview:** I say the idea in one line, write the code, and mention one edge case (like skipping empty slots in sparse arrays).

---

## 1. forEach

**Idea:** loop over the array and call the callback for each item. It returns nothing.

```js
Array.prototype.myForEach = function (cb, thisArg) {
  for (let i = 0; i < this.length; i++) {
    if (i in this) cb.call(thisArg, this[i], i, this);
  }
};
```

`i in this` skips empty slots, like the real `forEach`.

---

## 2. filter

**Idea:** call the callback on each item, and keep the item only if the callback returns a truthy value.

```js
Array.prototype.myFilter = function (cb, thisArg) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    if (i in this && cb.call(thisArg, this[i], i, this)) {
      result.push(this[i]);
    }
  }
  return result;
};
```

---

## 3. some and every

**Idea:** `some` returns `true` as soon as **one** item passes. `every` returns `false` as soon as **one** item fails. Both stop early.

```js
Array.prototype.mySome = function (cb, thisArg) {
  for (let i = 0; i < this.length; i++) {
    if (i in this && cb.call(thisArg, this[i], i, this)) return true;
  }
  return false;
};

Array.prototype.myEvery = function (cb, thisArg) {
  for (let i = 0; i < this.length; i++) {
    if (i in this && !cb.call(thisArg, this[i], i, this)) return false;
  }
  return true;
};
```

Edge case: `[].some(...)` is `false` and `[].every(...)` is `true` for an empty array.

---

## 4. call

**Idea:** I temporarily put the function on the object as a property, call it (so `this` becomes that object), and then remove it. I use a `Symbol` key so I do not overwrite an existing property.

```js
Function.prototype.myCall = function (ctx, ...args) {
  ctx = ctx == null ? globalThis : Object(ctx);
  const key = Symbol();
  ctx[key] = this;
  const result = ctx[key](...args);
  delete ctx[key];
  return result;
};
```

---

## 5. apply

**Idea:** same as `call`, but the arguments come as an array.

```js
Function.prototype.myApply = function (ctx, args = []) {
  ctx = ctx == null ? globalThis : Object(ctx);
  const key = Symbol();
  ctx[key] = this;
  const result = ctx[key](...args);
  delete ctx[key];
  return result;
};
```

**Remember the 3 together:** `call` runs now with arguments one by one. `apply` runs now with an array. `bind` returns a new function to run later.

---

## 6. JSON.stringify (simple version)

**Idea:** check the type of the value. Handle primitives directly, and call the function again (recursion) for arrays and objects.

Rules to mention: `undefined`, functions and symbols are **skipped in objects** and become `null` **inside arrays**. `NaN` and `Infinity` become `null`.

```js
function myStringify(value) {
  if (value === null) return 'null';
  const type = typeof value;

  if (type === 'string') return `"${value}"`;          // escapes ignored for simplicity
  if (type === 'number') return isFinite(value) ? String(value) : 'null';
  if (type === 'boolean') return String(value);
  if (type === 'undefined' || type === 'function' || type === 'symbol') return undefined;

  if (Array.isArray(value)) {
    const items = value.map(v => {
      const s = myStringify(v);
      return s === undefined ? 'null' : s;
    });
    return `[${items.join(',')}]`;
  }

  const parts = [];
  for (const key of Object.keys(value)) {
    const s = myStringify(value[key]);
    if (s !== undefined) parts.push(`"${key}":${s}`);
  }
  return `{${parts.join(',')}}`;
}

myStringify({ a: 1, b: [2, undefined], c: undefined });   // '{"a":1,"b":[2,null]}'
```

I would say: "This is a simple version. The real one also handles `toJSON`, dates, circular references and the replacer argument."

---

## Last-Minute Revision

- Always use `i in this` to skip empty slots in array polyfills.
- Use `cb.call(thisArg, item, index, array)` so the callback gets `this` and all 3 arguments.
- `some` stops at the first `true`. `every` stops at the first `false`.
- `call` and `apply`: attach the function to the object with a `Symbol` key, run it, then delete the key.
- `JSON.stringify`: skip `undefined`, functions and symbols in objects, and turn them into `null` in arrays.