# JavaScript Extra: Interview Cheat Sheet (Part 2)

Only the most important questions that are **not** already in `javascript.md`. Say each answer out loud in your own voice.

---

## 1. What are `async` and `defer`? Does a script in the head cause a problem?

A normal script in the head **stops the HTML reading**. The browser downloads the script, runs it, and only then continues. So the user sees a blank page for longer.

`async` and `defer` both download the script **in the background**. The difference is *when it runs*.

- **`async`:** runs as soon as it is downloaded. The order is **not guaranteed**. I use it for independent scripts like analytics.
- **`defer`:** runs **after the HTML is fully read**, just before `DOMContentLoaded`. The **order is kept**. I use it for my main app scripts.

```html
<script src="analytics.js" async></script>
<script src="app.js" defer></script>
```

Scripts with `type="module"` are deferred by default.

---

## 2. What are the data types? How do I check the type?

- **Primitive (7):** `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`
- **Non-primitive:** `object` (arrays, functions, dates are all objects)

How I check the type:

- `typeof x` works for most things. But `typeof null` is `"object"` (an old bug).
- `Array.isArray(x)` for arrays, because `typeof []` is also `"object"`.
- `x instanceof Class` for class instances.
- `Object.prototype.toString.call(x)` gives the exact type, like `"[object Date]"`.

**Falsy values (8):** `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`. Everything else is truthy, even `[]` and `{}`.

---

## 3. How does `==` work?

`==` is called abstract equality. If the types are different, JavaScript **converts them first**, then compares.

1. Same type: works like `===`.
2. `null == undefined` is `true`, and they are not equal to anything else.
3. Number and string: the string becomes a number.
4. Boolean: it becomes a number first (`true` is 1, `false` is 0).
5. Object and primitive: the object is converted to a primitive.

```js
'5' == 5           // true
null == undefined  // true
null == 0          // false
'' == 0            // true
[] == false        // true   ([] -> '' -> 0, false -> 0)
NaN == NaN         // false
```

In real projects I always use `===`. The only time I use `==` is `x == null`, which checks `null` and `undefined` together.

---

## 4. What is an execution context?

It is the environment where JavaScript code runs. There are two types: the **global** context (created once) and a **function** context (created on every function call).

Each context works in two phases:

1. **Creation phase:** memory is set up. `var` becomes `undefined`, function declarations are stored fully, `let` and `const` stay in the TDZ, and `this` is decided.
2. **Execution phase:** code runs line by line and real values are assigned.

All contexts are managed by the **call stack**. A context is pushed when a function is called and popped when it ends. Hoisting is just the result of the creation phase.

---

## 5. What is a stale closure?

A closure remembers the variables from where it was created. A **stale closure** happens when the function keeps using an **old value**, because it was created when the value was old and never got the new one.

The common place is React:

```js
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => console.log(count), 1000); // always 0
    return () => clearInterval(id);
  }, []);   // empty array: effect runs once, so count stays 0
}
```

**Fix:** add `count` to the dependency array, or use the functional update `setCount(c => c + 1)`, or keep the latest value in a `useRef`.

---

## 6. What is an IIFE? Give a few ways to write it.

An IIFE is a function that runs the moment it is created. I use it to make a private scope.

```js
(function () { console.log('one'); })();      // parentheses around function
(function () { console.log('two'); }());      // parentheses around whole call
(() => { console.log('three'); })();          // arrow IIFE
!function () { console.log('four'); }();      // operator (! or void)
(async () => { await fetch('/api'); })();     // async IIFE
```

The parentheses (or an operator) turn the function into an **expression**. Without them, JavaScript reads it as a declaration, which needs a name.

---

## 7. What does the `new` keyword do?

When I write `new Person('Aman')`, JavaScript does 4 things:

1. Creates a new empty object.
2. Links its prototype to `Person.prototype`.
3. Runs the constructor with `this` set to the new object.
4. Returns the object (unless the constructor returns another object).

If I forget `new`, `this` is `undefined` in strict mode (the code crashes), or it is the global object in normal mode (I create accidental globals).

---

## 8. What is prototypal inheritance? How is it different from classical inheritance?

- **Classical** (Java, C++): a class is a blueprint. Objects are made from it. Inheritance happens between classes.
- **Prototypal** (JavaScript): an object **links directly to another object**. If a property is missing, JavaScript looks in the linked object. The link is **live**, not a copy, so a change in the prototype shows in all objects.

```js
const animal = { eats: true };
const dog = Object.create(animal);
dog.eats;   // true, found through the prototype chain
```

The ES6 `class` keyword looks classical, but under the hood it still uses prototypes.

---

## 9. What are callbacks? What is inversion of control?

A callback is a function I pass to another function so it can be called later.

**Inversion of control** means that when I pass my callback to someone else's code, I **give up control**. That code decides when my callback runs, how many times, or if it runs at all. For example, a payment library might call my callback twice, and the user is charged twice.

**Promises fix this.** I get a Promise back and I decide what to do with `.then`. A Promise settles only **once**, so my code cannot be called twice.

---

## 10. What are the differences between an arrow function and a normal function?

1. **`this`:** arrow has no own `this`. It uses the `this` of the place where it was written.
2. **`arguments`:** arrow has no `arguments` object. I use rest parameters.
3. **`new`:** an arrow function cannot be used as a constructor.
4. **`prototype`:** arrow functions have no `prototype`.
5. **`call`, `apply`, `bind`:** they cannot change `this` of an arrow function.
6. **Hoisting:** a function declaration is fully hoisted. An arrow function is stored in a variable, so it cannot be used before its line.
7. **Syntax:** arrow is shorter and can return without the `return` keyword: `x => x * 2`.

```js
const obj = {
  name: 'Aman',
  start() {
    setTimeout(() => console.log(this.name), 0);   // "Aman"
  }
};
```

I do not use arrow functions as object methods, because `this` will not point to the object.

---

## 11. What is `"use strict"`?

It turns on **strict mode**, which throws errors for bad code that was silently allowed before. ES modules and classes are strict by default.

- Using an undeclared variable throws a `ReferenceError`.
- `this` in a normal function call is `undefined`, not the global object.
- Duplicate parameter names are not allowed.
- Writing to a read-only property throws an error.

---

## 12. What is the difference between `setTimeout` and `requestAnimationFrame`?

- **`setTimeout`** runs after **at least** the given time. It does not know about the screen, the timing can drift, and it keeps running in background tabs.
- **`requestAnimationFrame`** runs **right before the next paint**, about 60 times a second. It pauses in background tabs, saves battery, and gives smooth animations.

So for animations I always use `requestAnimationFrame`.

```js
let x = 0;
function move() {
  x += 2;
  box.style.transform = `translateX(${x}px)`;
  if (x < 300) requestAnimationFrame(move);
}
requestAnimationFrame(move);
```

---

## Last-Minute Revision

- `async`: runs when downloaded, no order. `defer`: runs after HTML, order kept.
- `typeof null` is `"object"`. Use `Array.isArray` for arrays.
- `null == undefined` is true, `null == 0` is false.
- Hoisting comes from the creation phase of the execution context.
- Stale closure: an old value stuck in a function that never re-ran.
- `new`: create object, link prototype, run constructor, return object.
- Inversion of control: giving your callback to someone else. Promises fix it.
- Arrow function: no `this`, no `arguments`, no `new`, no `prototype`.
- Animations: `requestAnimationFrame`, not `setTimeout`.