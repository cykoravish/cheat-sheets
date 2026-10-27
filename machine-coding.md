# Machine Coding: Interview Cheat Sheet

**Already in `javascript.md`:** flatten, deep clone, curry, compose/pipe. They are not repeated here.

**How I solve each problem in the interview:**
1. Repeat the problem in my own words and ask about edge cases.
2. Say the simple idea first, then the better idea.
3. Write the code and say the time and space complexity.

---

## 1. Move all zeros to the right

**Idea:** Use two pointers. `pos` marks where the next non-zero should go. I walk through the array, and swap each non-zero into `pos`.

```js
function moveZeros(arr) {
  let pos = 0;
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] !== 0) {
      [arr[pos], arr[i]] = [arr[i], arr[pos]];
      pos++;
    }
  }
  return arr;
}
moveZeros([0, 1, 0, 3, 12]);   // [1, 3, 12, 0, 0]
```

**Time:** O(n). **Space:** O(1).

---

## 2. Find a number in an array where each neighbor differs by +1 or -1

Example: `[4,5,6,7,6,5,4]`, find `7`.

**Idea:** If the current number is far from the target, I can **skip ahead**. The array changes by 1 at each step, so the target cannot appear before `|target - current|` steps.

```js
function findIndex(arr, target) {
  let i = 0;
  while (i < arr.length) {
    if (arr[i] === target) return i;
    i += Math.abs(target - arr[i]);
  }
  return -1;
}
findIndex([4, 5, 6, 7, 6, 5, 4], 7);   // 3
```

**Time:** better than O(n) in many cases, O(n) in the worst case. **Space:** O(1).

---

## 3. Shuffle an array

**Idea:** Fisher-Yates. Go from the end, and swap each item with a random item from the part before it.

```js
function shuffle(arr) {
  const a = [...arr];
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}
```

**Time:** O(n). **Gotcha:** `arr.sort(() => Math.random() - 0.5)` looks easy, but it gives a **biased** result, so I do not use it.

---

## 4. Fibonacci

**Idea:** Each number is the sum of the previous two.

```js
// Series of n numbers
function fibSeries(n) {
  const res = [];
  for (let i = 0; i < n; i++) res.push(i < 2 ? i : res[i - 1] + res[i - 2]);
  return res;
}
fibSeries(6);   // [0, 1, 1, 2, 3, 5]

// nth number with memoization
function fib(n, memo = {}) {
  if (n < 2) return n;
  if (memo[n]) return memo[n];
  return (memo[n] = fib(n - 1, memo) + fib(n - 2, memo));
}
```

**Time:** O(n) for both. The plain recursive version without memo is O(2^n), so I mention why memo is needed.

---

## 5. Remove duplicates from an array

```js
const unique = arr => [...new Set(arr)];                          // simplest, O(n)
const unique2 = arr => arr.filter((x, i) => arr.indexOf(x) === i); // O(n²)
```

For an array of objects, I use a `Map` or a `Set` with a key like `id`.

---

## 6. Find the missing number (from 0 to n)

**Idea:** The sum of 0 to n is `n * (n + 1) / 2`. I subtract the actual sum from it.

```js
function missingNumber(arr) {
  const n = arr.length;
  const expected = (n * (n + 1)) / 2;
  const actual = arr.reduce((a, b) => a + b, 0);
  return expected - actual;
}
missingNumber([3, 0, 1]);   // 2
```

**Time:** O(n). **Space:** O(1).

---

## 7. Convert a dot string into a nested object

Example: `"a.b.c.d.e"` becomes `{ a: { b: { c: { d: 'e' } } } }`. The last part is the value.

**Idea:** Take the last part as the value, then wrap it from right to left.

```js
function dotToObject(str) {
  const parts = str.split('.');
  let result = parts.pop();                         // 'e'
  while (parts.length) result = { [parts.pop()]: result };
  return result;
}
dotToObject('a.b.c.d.e');   // { a: { b: { c: { d: 'e' } } } }
```

---

## 8. Deep compare (deep equal)

**Idea:** If both are the same primitive, they are equal. If both are objects, compare the number of keys, then compare each key by calling the function again.

```js
function deepEqual(a, b) {
  if (a === b) return true;
  if (a !== a && b !== b) return true;                       // both NaN
  if (typeof a !== 'object' || typeof b !== 'object' || !a || !b) return false;
  if (Array.isArray(a) !== Array.isArray(b)) return false;

  const keysA = Object.keys(a);
  const keysB = Object.keys(b);
  if (keysA.length !== keysB.length) return false;

  return keysA.every(k => Object.hasOwn(b, k) && deepEqual(a[k], b[k]));
}
deepEqual({ x: [1, 2] }, { x: [1, 2] });   // true
```

I would also say: "It does not handle dates, maps, sets or circular objects yet."

---

## 9. Valid parentheses

**Idea:** Use a **stack**. Push opening brackets. For a closing bracket, the top of the stack must be its matching opening bracket.

```js
function isValid(str) {
  const pairs = { ')': '(', ']': '[', '}': '{' };
  const stack = [];
  for (const ch of str) {
    if ('([{'.includes(ch)) stack.push(ch);
    else if (stack.pop() !== pairs[ch]) return false;
  }
  return stack.length === 0;
}
isValid('{[()]}');   // true
isValid('([)]');     // false
```

**Time:** O(n). **Space:** O(n).

---

## 10. String compression

`"aaabbbcc"` becomes `"a3b3c2"`.

**Idea:** Count the repeats of the same character. When the character changes, write the character and its count.

```js
function compress(str) {
  let out = '';
  let count = 1;
  for (let i = 1; i <= str.length; i++) {
    if (str[i] === str[i - 1]) count++;
    else {
      out += str[i - 1] + count;
      count = 1;
    }
  }
  return out;
}
compress('aaabbbcc');   // "a3b3c2"
```

**Time:** O(n).

---

## 11. Write your own `new` keyword

**Idea:** Do the 4 steps that `new` does.

```js
function myNew(Constructor, ...args) {
  const obj = Object.create(Constructor.prototype);   // create + link prototype
  const result = Constructor.apply(obj, args);        // run constructor with this = obj
  const isObject = result !== null && (typeof result === 'object' || typeof result === 'function');
  return isObject ? result : obj;                     // return the object
}

function Person(name) { this.name = name; }
myNew(Person, 'Aman').name;   // "Aman"
```

---

## 12. Run promises one after another (chain)

**Idea:** Wait for each task before starting the next one.

```js
// tasks is an array of functions that return promises
async function runInSequence(tasks) {
  const results = [];
  for (const task of tasks) {
    results.push(await task());
  }
  return results;
}

// same thing with reduce
const runInSequence2 = tasks =>
  tasks.reduce(
    (chain, task) => chain.then(res => task().then(r => [...res, r])),
    Promise.resolve([])
  );
```

I would also say: "If the tasks do not depend on each other, `Promise.all` is faster, because it runs them in parallel."

**Small ones to remember:**
- Remove bad numbers (`-1`): `arr.filter(x => x !== -1)`
- Merge two arrays: `[...a, ...b]` or `a.concat(b)`

---

## Last-Minute Revision

- Move zeros: two pointers, O(n), O(1).
- +1/-1 array: jump by `|target - current|`.
- Shuffle: Fisher-Yates, never `sort(random)`.
- Fibonacci: iterative or memoized, never plain recursion.
- Missing number: expected sum minus actual sum.
- Valid parentheses: stack.
- Deep equal: compare key counts, then recurse.
- `new`: create object, link prototype, run constructor, return object.
- Sequential promises: `await` in a loop. Parallel: `Promise.all`.