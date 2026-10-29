# JavaScript Interview Cheat Sheet (Simple English)

These answers are written the way you can say them out loud in an interview. Short sentences, simple words. Practice saying each one in your own voice.

## 1. Core Concepts

**What is hoisting?** Before JavaScript runs the code, it moves declarations to the top of their scope. `var` is hoisted and set to `undefined`. `let` and `const` are also hoisted, but they cannot be used before the line where they are declared. If you try, you get a `ReferenceError`. Function declarations are fully hoisted, so you can call them before they appear in the code.

**What is the difference between var, let and const?**

- `var` is function-scoped. It can be redeclared and it is hoisted as `undefined`.
- `let` is block-scoped. It cannot be redeclared in the same scope.
- `const` is block-scoped and cannot be reassigned. But if it holds an object or an array, the inside of it can still change.

**What is the Temporal Dead Zone?** when we try to access let and const before its declaration that time is known as temporal dead zone. If you use the variable in this time, you get a `ReferenceError`.

**What is scope and lexical scope?** Scope decides where a variable can be used. Lexical scope means a function can use the variables from the place where it was written, not from the place where it was called. JavaScript looks for a variable in the current scope first, then in the outer scopes, one by one.

**What is a closure?** A closure is when an inner function remembers the variables of its outer function, even after the outer function has finished.

```js
function counter() {
  let count = 0;
  return () => ++count;
}
const c = counter();
c(); // 1
c(); // 2
```

We use closures for data privacy, memoization, currying, and event handlers.

**How does the `this` keyword work?** The value of `this` depends on how the function is called:

- Normal function call: `undefined` in strict mode, or the global object.
- Method call like `obj.fn()`: `this` is `obj`.
- With `new`: `this` is the new object.
- With `call`, `apply` or `bind`: `this` is what you pass.
- Arrow function: it has no `this` of its own. It uses the `this` from the outer scope.

**What is the difference between call, apply and bind?**

- `call` runs the function right now. You pass arguments one by one.
- `apply` runs the function right now. You pass arguments in an array.
- `bind` does not run the function. It returns a new function with `this` fixed, which you can call later.

**What is the difference between == and ===?** `==` converts the types before comparing, so `'5' == 5` is true. `===` checks both type and value, so `'5' === 5` is false. I always use `===` to avoid surprises.

**What is the difference between null and undefined?** `undefined` means a variable is declared but has no value yet. `null` means we set it to "no value" on purpose. One famous bug: `typeof null` returns `"object"`.

**What is the difference between primitive and reference types?** Primitives (string, number, boolean, null, undefined, symbol, bigint) are copied by value. Objects, arrays and functions are copied by reference. So if I write `b = a` for an object, and change `b`, then `a` also changes.

**What is the difference between shallow copy and deep copy?** A shallow copy (`{...obj}`, `Object.assign`, `slice`) copies only the first level. Nested objects are still shared. A deep copy copies everything. For a deep copy I use `structuredClone(obj)`. `JSON.parse(JSON.stringify(obj))` also works, but it loses functions, `undefined` and dates.

## 2. Prototype and OOP

**What is the prototype chain?** Every object has a hidden link to another object, called its prototype. If a property is not found on the object, JavaScript looks for it in the prototype, then in the prototype's prototype, until it reaches `null`. This is how inheritance works in JavaScript.

**What is the difference between `__proto__` and `prototype`?** `prototype` is a property of a constructor function. Objects created with `new` inherit from it. `__proto__` is the link on an object that points to that prototype. In modern code, I prefer `Object.getPrototypeOf(obj)`.

**What are the ways to create an object?** Object literal `{}`, `new Object()`, a constructor function with `new`, `Object.create(proto)`, and ES6 `class`.

**What is an ES6 class?** It is a cleaner syntax on top of prototype-based inheritance. We use `extends` to inherit and `super()` to call the parent constructor. In a child class, `super()` must be called before using `this`.

**What is the difference between Object.freeze, Object.seal and const?**

- `freeze`: you cannot add, delete or change anything (only the first level).
- `seal`: you cannot add or delete properties, but you can change existing values.
- `const`: it only stops reassigning the variable.

## 3. Functions

**What is the difference between an arrow function and a normal function?** An arrow function has no `this` and no `arguments` of its own, and it cannot be used with `new`. A normal function has all of these.

**What is a higher-order function?** A function that takes another function as an argument, or returns a function. `map`, `filter` and `reduce` are examples.

**What is a pure function?** A function that always gives the same output for the same input, and has no side effects. It does not change anything outside itself. Pure functions are easy to test.

**What is currying?** Turning a function with many arguments into a chain of functions that take one argument each, like `add(1)(2)(3)`.

**What is memoization?** Saving the result of an expensive function call, so that if we call it again with the same input, we return the saved result.

**What is an IIFE?** It is a function that runs as soon as it is defined: `(function(){ ... })();`. It is used to create a private scope.

**What is the difference between map and forEach?** `map` returns a new array, so you can chain it. `forEach` returns `undefined` and is used only for side effects.

## 4. Async JavaScript (asked the most)

**JavaScript is single-threaded. How does it do async work?** JavaScript has one call stack. Slow tasks like timers and network calls are handled by the browser (or Node.js) in the background. When they finish, their callbacks wait in a queue. The **event loop** keeps checking: if the call stack is empty, it moves the next callback from the queue to the stack.

**What is the order of the event loop?**

1. First, all the normal synchronous code runs.
2. Then all **microtasks** run (Promise `.then`, `queueMicrotask`).
3. Then one **macrotask** runs (`setTimeout`, `setInterval`, I/O). After that, microtasks run again, and so on.

```js
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');
// Output: 1, 4, 3, 2
```

**What is callback hell?** When callbacks are nested inside callbacks many times, the code becomes hard to read and maintain. We fix it with Promises or async/await.

**What are the states of a Promise?** A Promise starts as `pending`. Then it becomes `fulfilled` (success) or `rejected` (failure). Once it is settled, the state never changes.

**What is the difference between Promise.all, allSettled, race and any?**

- `all`: waits for all promises. If one fails, it rejects right away.
- `allSettled`: waits for all, and gives the result of each one, success or failure. It never rejects.
- `race`: returns the first promise that finishes, whether it succeeds or fails.
- `any`: returns the first promise that succeeds. If all fail, it gives an `AggregateError`.

**What is async/await?** It is a cleaner way to work with Promises. An `async` function always returns a Promise. `await` pauses that function until the Promise is done, but it does not block the rest of the program. We handle errors with `try/catch`.

**How do you avoid unhandled Promise errors?** Always add `.catch()` or use `try/catch`. And do not forget to `return` inside a `.then` chain.

**Why does setTimeout with 0 delay not run immediately?** Because it is a macrotask. It runs only after the current code and all microtasks are finished.

**How do you cancel a fetch request?** Use `AbortController`. Pass `signal: controller.signal` to `fetch`, and call `controller.abort()` when you want to cancel.

## 5. ES6+ Features

**What is destructuring?** It is a short way to take values out of arrays or objects.

```js
const { name, age = 18 } = user;   // default value
const [a, b] = [1, 2];
[a, b] = [b, a];                    // swap
```

**What is the difference between spread and rest?** Spread (`...`) opens a collection into separate items: `[...arr]`, `{...obj}`. Rest (`...args`) gathers many items into one array. Rest is used in function parameters and must be last.

**What are template literals?** Strings written with backticks. They support `${expression}` inside the string and can span many lines.

**What are optional chaining (`?.`) and nullish coalescing (`??`)?** `user?.address?.city` does not crash if something in the middle is `null` or `undefined`. `a ?? b` gives `b` only when `a` is `null` or `undefined`. The `||` operator also treats `0` and `''` as false, but `??` does not.

**What is the difference between for...of and for...in?** `for...of` loops over the **values** of an iterable (array, string, Map, Set). `for...in` loops over the **keys** of an object. Do not use `for...in` on arrays.

**What is the difference between Map and Object?** A Map can use any type as a key, keeps the insertion order, has a `size` property, and works better when we add and remove items often. An Object only uses strings or symbols as keys.

**What is a Set?** A collection of unique values. To remove duplicates from an array: `[...new Set(arr)]`.

**What are WeakMap and WeakSet?** Their keys must be objects, and they are held weakly. If nothing else refers to the object, it can be garbage collected. You cannot loop over them. They help avoid memory leaks.

**What are modules?** Modules let us split code into files using `import` and `export`. ES modules are the modern standard. CommonJS (`require`) is the older style used in Node.js. Dynamic `import('./file.js')` loads code only when needed.

**What are generators and iterators?** A generator (`function*` with `yield`) can pause and resume. An iterator is an object with a `next()` method that returns `{ value, done }`. To make an object iterable, we add `[Symbol.iterator]`.

**What is Symbol?** A unique and immutable primitive value, often used for unique object keys.

**What is BigInt?** A type for very large integers, like `123n`. Normal numbers are safe only up to `2^53 - 1`.

## 6. Arrays and Objects

**What is the difference between slice and splice?**

- `slice(start, end)` returns a new array and does **not** change the original. The end index is not included.
- `splice(start, deleteCount, ...items)` **changes** the original array and returns the removed items.

**Which array methods change the original array?** These change it: `push, pop, shift, unshift, splice, sort, reverse, fill`. These return a new array: `map, filter, slice, concat, flat`.

**What is the gotcha with sort()?** Without a compare function, `sort()` sorts as strings. So `[10, 9, 1].sort()` gives `[1, 10, 9]`. For numbers, use `arr.sort((a, b) => a - b)`.

**Useful array tricks**

```js
arr.flat(Infinity);          // flatten
[...new Set(arr)];           // unique values
arr.length = 0;              // empty the array
```

**How do you check if an object has a key?** Use `'key' in obj`, `obj.hasOwnProperty('key')`, or `Object.hasOwn(obj, 'key')`.

**What are Object.keys, values and entries?** They return an array of keys, an array of values, and an array of `[key, value]` pairs.

## 7. DOM and Browser

**What are event bubbling and capturing?** An event travels in two phases. In **capturing**, it goes from the window down to the target. In **bubbling**, it goes from the target back up to the window. By default, listeners run in the bubbling phase. To use capturing, pass `true` as the third argument.

**What is event delegation?** Instead of adding a listener to every child, we add one listener to the parent and check `event.target`. It saves memory and also works for elements added later.

**What is the difference between preventDefault and stopPropagation?** `preventDefault()` stops the default browser action, like a form submit or a link click. `stopPropagation()` stops the event from moving to other elements.

**What is the difference between cookie, localStorage and sessionStorage?**

|  | Cookie | localStorage | sessionStorage |
| --- | --- | --- | --- |
| Size | about 4 KB | about 5-10 MB | about 5 MB |
| Expiry | can be set | never | when the tab closes |
| Sent to server | yes, with every request | no | no |

**What are the same-origin policy and CORS?** The browser stops a script from one origin (protocol + domain + port) from reading data of another origin. CORS is a way for the server to allow this, using headers like `Access-Control-Allow-Origin`.

**What is the difference between debounce and throttle?**

- Debounce: run the function only after the user stops the action for some time. Example: search box typing.
- Throttle: run the function at most once in a fixed time. Example: scroll or resize.

**What is the difference between reflow and repaint?** Reflow means the browser calculates the layout again (size or position changed). It is expensive. Repaint means only the look changed, like color. It is cheaper.

**What causes memory leaks?** Accidental global variables, event listeners and timers that are never removed, closures that hold big objects, and removed DOM nodes that are still referenced.

**What are Web Workers?** They run heavy work in a separate thread, so the main thread (UI) does not freeze. They cannot access the DOM.

## 8. Coding Questions (you may need to write these)

**Debounce**

```js
function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}
```

**Throttle**

```js
function throttle(fn, limit) {
  let inThrottle = false;
  return function (...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}
```

**Curry**

```js
function curry(fn) {
  return function curried(...args) {
    return args.length >= fn.length
      ? fn.apply(this, args)
      : (...next) => curried.apply(this, [...args, ...next]);
  };
}
```

**Memoize**

```js
function memoize(fn) {
  const cache = {};
  return (...args) => {
    const key = JSON.stringify(args);
    if (!(key in cache)) cache[key] = fn(...args);
    return cache[key];
  };
}
```

**bind polyfill**

```js
Function.prototype.myBind = function (ctx, ...preArgs) {
  const fn = this;
  return function (...args) {
    return fn.apply(ctx, [...preArgs, ...args]);
  };
};
```

**map polyfill**

```js
Array.prototype.myMap = function (cb) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    result.push(cb(this[i], i, this));
  }
  return result;
};
```

**reduce polyfill**

```js
Array.prototype.myReduce = function (cb, init) {
  let acc = init;
  let i = 0;
  if (acc === undefined) { acc = this[0]; i = 1; }
  for (; i < this.length; i++) acc = cb(acc, this[i], i, this);
  return acc;
};
```

**Promise.all polyfill**

```js
function myPromiseAll(promises) {
  return new Promise((resolve, reject) => {
    const results = [];
    let done = 0;
    if (promises.length === 0) return resolve([]);
    promises.forEach((p, i) => {
      Promise.resolve(p).then(val => {
        results[i] = val;
        if (++done === promises.length) resolve(results);
      }).catch(reject);
    });
  });
}
```

**Deep clone (simple)**

```js
function deepClone(obj) {
  if (obj === null || typeof obj !== 'object') return obj;
  const copy = Array.isArray(obj) ? [] : {};
  for (const key in obj) {
    if (Object.hasOwn(obj, key)) copy[key] = deepClone(obj[key]);
  }
  return copy;
}
```

In the interview, also say: "In real projects I would use `structuredClone`."

**Flatten array**

```js
const flatten = arr =>
  arr.reduce((acc, x) => acc.concat(Array.isArray(x) ? flatten(x) : x), []);
```

**Compose and Pipe**

```js
const pipe = (...fns) => x => fns.reduce((v, f) => f(v), x);
const compose = (...fns) => x => fns.reduceRight((v, f) => f(v), x);
```

## 9. Output-Based Gotchas

```js
typeof null            // "object"
0.1 + 0.2 === 0.3      // false (0.30000000000000004)
NaN === NaN            // false, but Object.is(NaN, NaN) is true
[] + []                // ""
[] + {}                // "[object Object]"
1 + 2 + '3'            // "33"
'3' + 1 + 2            // "312"
[10, 9, 1].sort()      // [1, 10, 9]

console.log(a); var a = 5;   // undefined
console.log(b); let b = 5;   // ReferenceError

for (var i = 0; i < 3; i++) setTimeout(() => console.log(i), 0); // 3 3 3
for (let i = 0; i < 3; i++) setTimeout(() => console.log(i), 0); // 0 1 2
```

Why? With `var`, all the callbacks share one `i`. With `let`, every loop turn gets its own new `i`.

## 10. Phrases You Can Use in the Interview

- "Let me think for a moment."
- "In simple words, it means..."
- "For example, ..."
- "One thing to be careful about is..."
- "I am not 100% sure, but I think..."
- "Could you please repeat the question?"

**Answer pattern:** (1) Give a one-line definition. (2) Give a small example. (3) Mention one difference or gotcha. Speak slowly. A short, clear answer is better than a long, confused one.

All the best for your interview!
