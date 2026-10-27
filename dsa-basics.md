# DSA Basics: Interview Cheat Sheet

Only the most common patterns that web developer interviews ask. Code is in JavaScript.

**Already covered in `machine-coding.md`:** remove duplicates, missing number, valid parentheses, string compression, move zeros, fibonacci.

**How I solve any DSA problem:**
1. Repeat the problem and ask about **edge cases** (empty input, one item, duplicates, negative numbers).
2. Say the **simple (brute force) solution** first and its complexity.
3. Then improve it. Ask: can a **hash map**, **two pointers**, **sliding window** or **sorting** help?
4. Write the code, test it with a small example out loud, and say the **time and space complexity**.

---

## 1. What is Big-O notation?

Big-O tells **how the running time (or memory) grows** when the input size `n` grows. It looks at the **worst case** and ignores small details.

| Big-O | Name | Example |
|---|---|---|
| O(1) | constant | array access `arr[5]`, hash map get/set |
| O(log n) | logarithmic | binary search |
| O(n) | linear | one loop, `indexOf` |
| O(n log n) | linearithmic | good sorting (merge sort) |
| O(n²) | quadratic | loop inside a loop |
| O(2ⁿ) | exponential | plain recursive fibonacci |

Useful facts: JS `Array.sort` is O(n log n). `push` and `pop` are O(1). `shift` and `unshift` are O(n). `Map` and `Set` operations are O(1) on average. `includes` on an array is O(n), but `has` on a Set is O(1).

---

## 2. Two Sum (Hash Map)

**Problem:** Find two numbers in an array that add up to a target. Return their indexes.

**Brute force:** two loops, O(n²).
**Better idea:** For each number, I need `target - number`. I store the numbers I have seen in a **Map**, so I can check in O(1).

```js
function twoSum(nums, target) {
  const seen = new Map();                    // number -> index
  for (let i = 0; i < nums.length; i++) {
    const need = target - nums[i];
    if (seen.has(need)) return [seen.get(need), i];
    seen.set(nums[i], i);
  }
  return [];
}
twoSum([2, 7, 11, 15], 9);   // [0, 1]
```

**Time:** O(n). **Space:** O(n).

---

## 3. Palindrome check (Two Pointers)

**Idea:** One pointer at the start, one at the end. Move them toward each other and compare.

```js
function isPalindrome(str) {
  const s = str.toLowerCase().replace(/[^a-z0-9]/g, '');   // ignore spaces and symbols
  let left = 0;
  let right = s.length - 1;
  while (left < right) {
    if (s[left] !== s[right]) return false;
    left++;
    right--;
  }
  return true;
}
isPalindrome('A man, a plan, a canal: Panama');   // true
```

**Time:** O(n). **Space:** O(n) because of the cleaned string (the pointer part alone is O(1)).

The two pointers pattern also works for: reverse an array, remove duplicates from a **sorted** array, and Two Sum on a **sorted** array.

---

## 4. Maximum sum of a subarray of size k (Sliding Window)

**Idea:** Instead of adding k numbers again for every position, I **slide** the window. I add the new number that enters and remove the one that leaves.

```js
function maxSum(arr, k) {
  if (arr.length < k) return null;
  let windowSum = 0;
  for (let i = 0; i < k; i++) windowSum += arr[i];

  let max = windowSum;
  for (let i = k; i < arr.length; i++) {
    windowSum += arr[i] - arr[i - k];
    max = Math.max(max, windowSum);
  }
  return max;
}
maxSum([2, 1, 5, 1, 3, 2], 3);   // 9
```

**Time:** O(n). **Space:** O(1).

---

## 5. Longest substring without repeating characters (Sliding Window + Map)

**Idea:** Keep a window `[start, i]` with no repeats. If the current character was already seen **inside the window**, move `start` just after its last position.

```js
function lengthOfLongestSubstring(s) {
  const lastSeen = new Map();     // char -> last index
  let start = 0;
  let best = 0;

  for (let i = 0; i < s.length; i++) {
    const ch = s[i];
    if (lastSeen.has(ch) && lastSeen.get(ch) >= start) {
      start = lastSeen.get(ch) + 1;
    }
    lastSeen.set(ch, i);
    best = Math.max(best, i - start + 1);
  }
  return best;
}
lengthOfLongestSubstring('abcabcbb');   // 3
```

**Time:** O(n). **Space:** O(k), where k is the number of different characters.

---

## 6. Binary Search

**Idea:** In a **sorted** array, look at the middle. If the target is smaller, search the left half. If bigger, search the right half. Each step removes half of the data.

```js
function binarySearch(arr, target) {
  let lo = 0;
  let hi = arr.length - 1;
  while (lo <= hi) {
    const mid = Math.floor((lo + hi) / 2);
    if (arr[mid] === target) return mid;
    if (arr[mid] < target) lo = mid + 1;
    else hi = mid - 1;
  }
  return -1;
}
binarySearch([1, 3, 5, 7, 9, 11], 7);   // 3
```

**Time:** O(log n). **Space:** O(1). The key condition is that the array must be **sorted**.

---

## 7. Maximum subarray sum (Kadane's Algorithm)

**Problem:** Find the contiguous subarray with the largest sum.

**Idea:** At each number, I decide: **start fresh** from this number, or **continue** the previous sum. I keep the best seen.

```js
function maxSubArray(nums) {
  let current = nums[0];
  let best = nums[0];
  for (let i = 1; i < nums.length; i++) {
    current = Math.max(nums[i], current + nums[i]);
    best = Math.max(best, current);
  }
  return best;
}
maxSubArray([-2, 1, -3, 4, -1, 2, 1, -5, 4]);   // 6
```

**Time:** O(n). **Space:** O(1).

---

## 8. Reverse a linked list

A linked list node looks like `{ val, next }`.

**Idea:** Walk through the list, and flip each `next` pointer to point to the previous node. I keep 3 variables: `prev`, `curr`, `next`.

```js
function reverseList(head) {
  let prev = null;
  let curr = head;
  while (curr) {
    const next = curr.next;   // save the next node
    curr.next = prev;         // flip the pointer
    prev = curr;
    curr = next;
  }
  return prev;                // new head
}
```

**Time:** O(n). **Space:** O(1).

---

## 9. Detect a cycle in a linked list (Fast and Slow pointers)

**Idea:** Use two pointers. `slow` moves 1 step, `fast` moves 2 steps. If there is a cycle, `fast` will eventually catch `slow`. If there is no cycle, `fast` reaches the end.

```js
function hasCycle(head) {
  let slow = head;
  let fast = head;
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow === fast) return true;
  }
  return false;
}
```

**Time:** O(n). **Space:** O(1).

---

## 10. Binary tree: max depth (DFS) and level order (BFS)

A tree node looks like `{ val, left, right }`.

- **DFS (Depth First):** go deep first. Uses recursion or a stack.
- **BFS (Breadth First):** go level by level. Uses a **queue**.

```js
// Max depth (DFS, recursion)
function maxDepth(node) {
  if (!node) return 0;
  return 1 + Math.max(maxDepth(node.left), maxDepth(node.right));
}

// Level order (BFS, queue)
function levelOrder(root) {
  if (!root) return [];
  const result = [];
  const queue = [root];
  while (queue.length) {
    const size = queue.length;
    const level = [];
    for (let i = 0; i < size; i++) {
      const node = queue.shift();
      level.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    result.push(level);
  }
  return result;
}
```

**Time:** O(n) for both. **Space:** O(h) for DFS (h is the height), O(w) for BFS (w is the widest level).

Tree traversals to remember: **Inorder** (left, root, right) gives sorted order in a BST. **Preorder** (root, left, right). **Postorder** (left, right, root).

---

## 11. Merge Sort (Divide and Conquer)

**Idea:** Split the array in half, sort each half (recursion), then **merge** the two sorted halves.

```js
function mergeSort(arr) {
  if (arr.length <= 1) return arr;
  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));

  const merged = [];
  let i = 0, j = 0;
  while (i < left.length && j < right.length) {
    merged.push(left[i] <= right[j] ? left[i++] : right[j++]);
  }
  return [...merged, ...left.slice(i), ...right.slice(j)];
}
mergeSort([5, 2, 9, 1, 3]);   // [1, 2, 3, 5, 9]
```

**Time:** O(n log n) always. **Space:** O(n).

For comparison: **Quick sort** is O(n log n) on average, O(n²) in the worst case, but it sorts in place. **Bubble sort** is O(n²) and only good for learning.

---

## 12. Quick data structure choices

- **Array:** fast access by index. Slow to insert in the middle.
- **Hash Map / Set:** fast lookup, count items, find duplicates.
- **Stack (LIFO):** undo, brackets, DFS. `push` and `pop`.
- **Queue (FIFO):** BFS, task scheduling. (`shift` on an array is O(n), a linked list queue is O(1).)
- **Linked list:** fast insert and delete at a known place. Slow to search.
- **Tree / BST:** hierarchical data, fast search when balanced (O(log n)).
- **Heap:** quick access to the smallest or largest item (priority queue).

---

## Last-Minute Revision

- Always say the brute force first, then improve it.
- Need to find something fast: **hash map**. Need a pair in sorted data: **two pointers**. Need a range or subarray: **sliding window**. Sorted data: **binary search**.
- Two Sum: Map of `number -> index`, O(n).
- Sliding window: add the new item, remove the old item.
- Binary search needs a sorted array. O(log n).
- Linked list: reverse with `prev`, `curr`, `next`. Cycle with slow and fast pointers.
- Tree: DFS with recursion, BFS with a queue.
- Merge sort: split, sort, merge. O(n log n).
- Say time and space complexity after every solution.