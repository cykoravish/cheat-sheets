# React: Interview Cheat Sheet

Only the most important questions. Say each answer out loud in your own voice.

---

## 1. What is React? What is the Virtual DOM?

React is a JavaScript library for building user interfaces using **components**. I describe how the UI should look for a given state, and React updates the screen for me.

The **Virtual DOM** is a light copy of the real DOM, kept as plain JavaScript objects. When state changes:

1. React builds a **new** virtual tree.
2. It **compares** it with the old one (this is called **diffing**).
3. It updates **only the parts that changed** in the real DOM. This whole process is called **reconciliation**.

Touching the real DOM is slow, so React keeps those updates small.

---

## 2. What is the difference between props and state?

- **Props** are data passed **from a parent to a child**. They are **read-only**. A child must never change its own props.
- **State** is data that a component **owns and can change** over time. When state changes, the component **re-renders**.

Data flows in one direction: **down** from parent to child.

---

## 3. How does `useState` work? Why should I use the functional update?

```js
const [count, setCount] = useState(0);
```

State updates are **batched** and applied on the **next render**. So right after `setCount(count + 1)`, the variable `count` still has the old value.

```js
// Called 3 times, but the result is +1 (all use the same old count)
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);

// Functional update: result is +3
setCount(c => c + 1);
setCount(c => c + 1);
setCount(c => c + 1);
```

Rules: I **never change state directly**. I always create a new object or array: `setUser({ ...user, name: 'Aman' })`. For complex state with many actions, I use `useReducer`.

---

## 4. How does `useEffect` work?

`useEffect` runs code **after the component renders**. I use it for side effects like fetching data, timers and event listeners.

```js
useEffect(() => {
  const id = setInterval(tick, 1000);
  return () => clearInterval(id);    // cleanup
}, [dep]);
```

The dependency array controls when it runs:

- **No array:** after **every** render.
- **`[]`:** only **once**, after the first render (mount).
- **`[a, b]`:** when `a` or `b` **changes**.

The **cleanup** function runs before the next effect and when the component unmounts. I use it to remove listeners and clear timers.

**Common mistakes:**
- Missing a dependency, which causes a **stale closure** (old value inside the effect).
- Putting an object or array in the deps that is recreated on every render, which causes an infinite loop.
- Setting state inside an effect with no deps.
- In development, **StrictMode** runs effects twice on purpose to catch missing cleanup.

**Fetching data safely** (so an old response does not overwrite a new one):

```js
useEffect(() => {
  const controller = new AbortController();
  fetch(`/api/users/${id}`, { signal: controller.signal })
    .then(res => res.json())
    .then(setUser)
    .catch(err => { if (err.name !== 'AbortError') console.error(err); });
  return () => controller.abort();
}, [id]);
```

---

## 5. What are `useRef`, `useMemo` and `useCallback`?

- **`useRef`** keeps a value that **stays the same between renders**. Changing it does **not** cause a re-render. I use it to access a DOM element (`ref.current`) or to store a timer id.
- **`useMemo`** **caches a calculated value**, so an expensive calculation does not run on every render.
- **`useCallback`** **caches a function**, so it keeps the same reference between renders.

```js
const total = useMemo(() => items.reduce((s, i) => s + i.price, 0), [items]);
const handleClick = useCallback(() => doSomething(id), [id]);
```

I do not use them everywhere. I use them when there is a real performance problem, or when I need a stable reference for a memoized child component.

---

## 6. Why does a component re-render? How do I prevent extra renders?

A component re-renders when:
1. Its **state** changes.
2. Its **parent** re-renders (even if its own props did not change).
3. A **context** it uses changes.

To prevent extra renders:
- **`React.memo`**: skips re-rendering if the props are **shallowly equal**.
- Keep state **close to where it is used**, so fewer components re-render.
- Use `useMemo` and `useCallback` for objects and functions passed as props. **Gotcha:** an object or function created inline is **new on every render**, so `React.memo` does not help unless I memoize it.
- Find the real problem first using the **React DevTools Profiler**.

---

## 7. Why do we need `key` in a list? Why not use the index?

A `key` helps React know **which item changed, was added or was removed**, so it updates only that one.

```jsx
{users.map(user => <UserCard key={user.id} user={user} />)}
```

I use a **stable and unique id**. I avoid the array index when the list can be **reordered, filtered or have items inserted or deleted**. With an index, React can attach the **wrong state** to the wrong item (for example, a wrong input value stays in a row).

---

## 8. What are controlled and uncontrolled components?

- **Controlled:** React state is the single source of truth for the input value.

```jsx
<input value={name} onChange={e => setName(e.target.value)} />
```

- **Uncontrolled:** the **DOM** keeps the value, and I read it when I need it, with a `ref` or `FormData`.

Controlled gives me full control for validation and instant feedback. Uncontrolled is simpler and causes fewer re-renders for big forms.

---

## 9. What is prop drilling? What are Context and state management?

**Prop drilling** is passing props through many layers of components that do not need them, just to reach a deep child.

**Context API** shares a value (like theme or logged-in user) with any child **without passing props**. **Gotcha:** every component that uses the context re-renders when its value changes, so I do not use it for data that changes very often.

For bigger apps:
- **Client state:** Redux Toolkit or Zustand.
- **Server state** (API data, caching, loading and error states): **TanStack Query (React Query)**.

---

## 10. What are custom hooks? What are the rules of hooks?

A **custom hook** is a function that starts with `use` and **reuses stateful logic** between components.

```js
function useDebounce(value, delay = 300) {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id);
  }, [value, delay]);
  return debounced;
}
```

**Rules of hooks:**
1. Call hooks only at the **top level**. Never inside `if`, loops or nested functions.
2. Call hooks only in **function components** or **custom hooks**.

The reason: React remembers hooks by **the order in which they are called**, so the order must be the same on every render.

---

## 11. How do I improve React performance? What is code splitting?

- **Code splitting:** load a component only when needed with `React.lazy` and `Suspense`.

```jsx
const Settings = React.lazy(() => import('./Settings'));

<Suspense fallback={<Spinner />}>
  <Settings />
</Suspense>
```

- Use `React.memo`, `useMemo` and `useCallback` where it really helps.
- **Virtualize long lists** (`react-window`).
- Use stable **keys**, and never define a component **inside** another component.
- Lazy load images, and keep bundle size small.

---

## 12. What are Error Boundaries?

An error boundary is a component that **catches errors in its children during rendering** and shows a fallback UI instead of crashing the whole app. It is built with a class component (`componentDidCatch` or `getDerivedStateFromError`), or I use the `react-error-boundary` library.

It does **not** catch errors in event handlers, async code (like `fetch`) or server-side rendering. For those I use `try...catch`.

---

## 13. What is Next.js? What are Server Components?

Next.js is a framework on top of React. It gives me:
- **File-based routing**
- **SSR, SSG and ISR** (good for SEO and fast first load)
- **API routes** (a backend inside the same project)
- Image and font optimization

**Server Components** (App Router) run **only on the server** and send no JavaScript for them to the browser. They can read from the database directly. When I need state, effects or click handlers, I mark a component with `'use client'` to make it a **Client Component**.

---

## Last-Minute Revision

- Virtual DOM: new tree, diff with old tree, update only the changes.
- Props are read-only and come from the parent. State is owned and triggers a re-render.
- Use `setCount(c => c + 1)` when the new value depends on the old one.
- `useEffect`: no array runs every time, `[]` runs once, `[a]` runs when `a` changes. Always clean up.
- `useRef` does not re-render. `useMemo` caches a value. `useCallback` caches a function.
- Re-render causes: state, parent, context. Fix with `React.memo` and stable props.
- Keys: stable unique ids, not the index.
- Context re-renders all consumers. Use React Query for server data.
- Hooks: only at the top level, only in components or custom hooks.
- `React.lazy` and `Suspense` for code splitting.