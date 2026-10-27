# TypeScript: Interview Cheat Sheet

Only the most important questions. Say each answer out loud in your own voice.

---

## 1. What is TypeScript? Why use it?

TypeScript is **JavaScript with types**. I write types, the TypeScript compiler **checks** my code, and then it **compiles to plain JavaScript**. The browser and Node run the JavaScript, never the TypeScript.

Benefits:
- It catches many **bugs at compile time**, before the code runs.
- Better **autocomplete** and safer **refactoring** in the editor.
- The types work as **documentation**, which is great for big teams.

**Important:** types are **erased** after compiling. TypeScript does **not** check data at runtime. For API data or user input, I still need runtime validation (for example with **Zod**).

---

## 2. What is the difference between `type` and `interface`?

Both describe the shape of an object, and in many cases they work the same.

```ts
interface User {
  id: number;
  name: string;
}

type UserType = {
  id: number;
  name: string;
};
```

| | `interface` | `type` |
|---|---|---|
| Extend | `interface B extends A` | `type B = A & { ... }` |
| Declaration merging (same name twice joins) | **yes** | no |
| Union, tuple, primitive, mapped types | no | **yes** |

My rule: I use `interface` for **object shapes** (especially public APIs and classes), and `type` for **unions**, **tuples** and anything more advanced. The most important thing is to be **consistent** in the project.

---

## 3. What is the difference between `any`, `unknown` and `never`?

- **`any`** turns **off** type checking. I can do anything with it. It is unsafe, so I avoid it.
- **`unknown`** is the **safe version of `any`**. I can put any value in it, but I must **check the type first** before using it.
- **`never`** means a value that **never happens**: a function that always throws, or an impossible case.

```ts
let a: any = 'hi';
a.foo.bar();                  // no error, crashes at runtime

let u: unknown = 'hi';
// u.toUpperCase();           // error
if (typeof u === 'string') u.toUpperCase();   // OK after checking
```

---

## 4. What are generics?

Generics let me write **reusable code that works with many types**, and keep the type safety. I think of the type as a **parameter**.

```ts
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

first<number>([1, 2, 3]);    // number
first(['a', 'b']);           // string (TypeScript guesses T)

interface ApiResponse<T> {
  data: T;
  error?: string;
}
const res: ApiResponse<User[]> = { data: [] };
```

I can also limit a generic: `<T extends { id: number }>` means T must have an `id`.

---

## 5. What are union types, narrowing and discriminated unions?

A **union** means "one of these types": `string | number`.

**Narrowing** means checking the type so TypeScript knows which one it is: `typeof`, `instanceof`, `in`, or `Array.isArray`.

```ts
function print(value: string | number) {
  if (typeof value === 'string') return value.toUpperCase();   // string here
  return value.toFixed(2);                                      // number here
}
```

A **discriminated union** has a common field (like `type`) that tells the shape. It is great for API results and state:

```ts
type Result =
  | { status: 'success'; data: string }
  | { status: 'error'; message: string };

function handle(r: Result) {
  if (r.status === 'success') console.log(r.data);
  else console.log(r.message);
}
```

---

## 6. What are the common utility types?

| Utility | What it does |
|---|---|
| `Partial<T>` | makes all properties optional |
| `Required<T>` | makes all properties required |
| `Readonly<T>` | makes all properties read-only |
| `Pick<T, 'a' \| 'b'>` | keeps only the listed properties |
| `Omit<T, 'a'>` | removes the listed properties |
| `Record<K, V>` | an object with keys `K` and values `V` |
| `ReturnType<typeof fn>` | the return type of a function |

```ts
type UpdateUser = Partial<User>;                    // for PATCH requests
type PublicUser = Omit<User, 'password'>;
const roles: Record<string, number> = { admin: 1, user: 2 };
```

---

## 7. How do optional, readonly and non-null assertion work?

```ts
interface User {
  id: number;
  readonly createdAt: Date;   // cannot be changed after creation
  nickname?: string;          // optional: string | undefined
}

const el = document.getElementById('app')!;   // "!" tells TS: this is not null
```

The `!` (non-null assertion) is a **promise to TypeScript** that the value is not null. If I am wrong, it crashes at runtime. So I use a proper `if` check when I can.

---

## 8. `enum` vs union of literals

```ts
enum Role { Admin, User }                        // creates real JS code

type Role2 = 'admin' | 'user';                   // no extra JS code
```

Many teams prefer **union of string literals**, because it is simple, has no extra runtime code, and works well with narrowing.

---

## 9. How do I use TypeScript with React?

```tsx
interface ButtonProps {
  label: string;
  onClick: () => void;
  disabled?: boolean;
}

function Button({ label, onClick, disabled = false }: ButtonProps) {
  return <button onClick={onClick} disabled={disabled}>{label}</button>;
}

const [user, setUser] = useState<User | null>(null);
const inputRef = useRef<HTMLInputElement>(null);

const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  console.log(e.target.value);
};
```

---

## 10. What is `strict` mode in `tsconfig.json`?

`"strict": true` turns on a group of strict checks. The most important ones are:

- **`strictNullChecks`:** `null` and `undefined` are not allowed unless I say so. This catches the famous "cannot read property of undefined" bugs.
- **`noImplicitAny`:** I must give a type, not let it silently become `any`.

I always use strict mode in new projects.

---

## Last-Minute Revision

- TypeScript compiles to JavaScript. Types are erased and not checked at runtime (use Zod for runtime checks).
- `interface` for object shapes. `type` for unions and advanced types.
- `any` is unsafe. `unknown` needs a check first. `never` means it cannot happen.
- Generics: `function first<T>(arr: T[]): T`.
- Narrow with `typeof`, `in`, `instanceof`. Use discriminated unions for results.
- Utility types: `Partial`, `Pick`, `Omit`, `Record`, `ReturnType`.
- `!` is a promise, not a check.
- Always turn on `strict`.