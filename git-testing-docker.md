# Git, Testing and Docker: Interview Cheat Sheet

Only the most important questions. Say each answer out loud in your own voice.

---

# Git

## 1. What is the difference between `git fetch` and `git pull`?

- **`git fetch`** downloads the new changes from the remote, but it does **not change my working files**. It is safe, and I can look at the changes first.
- **`git pull`** is **`fetch` + `merge`** (or `fetch` + `rebase` with `git pull --rebase`). It downloads and also applies the changes to my current branch.

---

## 2. What is the difference between `git merge` and `git rebase`?

Both bring the changes of one branch into another.

- **`merge`** creates a **new merge commit** that joins the two histories. The history shows exactly what happened, but it can get messy.
- **`rebase`** **replays my commits on top** of the other branch. The history becomes a **straight line** and looks clean, but it **rewrites commits** (new commit ids).

**Golden rule:** I **never rebase a branch that other people are already using** (like `main`), because it rewrites shared history. I use rebase for my own local feature branch.

---

## 3. What is the difference between `git reset` and `git revert`?

- **`git revert <commit>`** creates a **new commit that undoes** the old commit. The history stays safe. This is the right choice for **shared branches**.
- **`git reset`** **moves the branch pointer back**, so commits are removed from history.
  - `--soft`: keeps my changes staged.
  - `--mixed` (default): keeps my changes but unstaged.
  - `--hard`: **deletes** my changes too. It is dangerous.

---

## 4. How do I resolve a merge conflict?

A conflict happens when two branches **change the same lines**.

1. Git marks the file with `<<<<<<<`, `=======` and `>>>>>>>`.
2. I open the file, choose or combine the correct code, and **remove the markers**.
3. Then `git add <file>` and `git commit` (or `git rebase --continue`).

To avoid conflicts: pull often, keep branches short-lived, and make small commits.

---

## 5. Other Git commands I should know

- **`git stash`:** temporarily saves my uncommitted changes, so I can switch branches. `git stash pop` brings them back.
- **`git cherry-pick <commit>`:** copies one specific commit onto my current branch.
- **`git log --oneline`**, **`git diff`**, **`git status`**, **`git blame`**.
- **`git commit --amend`:** change the last commit (only if I have not pushed it).
- **`git branch -d`** deletes a branch. **`.gitignore`** lists files Git should ignore (like `node_modules` and `.env`).

**Typical team workflow:** create a **feature branch** from `main`, make small commits with clear messages, push, open a **Pull Request**, get a code review, pass the CI checks, and merge.

---

# Testing

## 6. What are unit, integration and end-to-end (E2E) tests?

- **Unit test:** tests **one small piece** (a function or component) alone. Fast and cheap.
- **Integration test:** tests that **several parts work together** (for example an API route with the database).
- **E2E test:** tests the **whole app like a real user** in a browser (Cypress, Playwright). Slow, but the most realistic.

**Testing pyramid:** many unit tests at the bottom, fewer integration tests in the middle, and a few E2E tests at the top.

---

## 7. How do I write a test with Jest?

```js
// sum.js
function sum(a, b) { return a + b; }
module.exports = sum;

// sum.test.js
const sum = require('./sum');

describe('sum', () => {
  test('adds two numbers', () => {
    expect(sum(2, 3)).toBe(5);
  });

  test('works with negative numbers', () => {
    expect(sum(-1, -1)).toBe(-2);
  });
});
```

Useful matchers: `toBe` (same value), `toEqual` (deep equal), `toBeTruthy`, `toContain`, `toThrow`.

**Testing async code:**

```js
test('fetches the user', async () => {
  const user = await getUser(1);
  expect(user.name).toBe('Aman');
});
```

---

## 8. What is mocking? How do I test React components?

**Mocking** means replacing a real dependency (like an API call or database) with a **fake one**, so the test is fast and does not depend on outside things.

```js
jest.mock('./api');
getUser.mockResolvedValue({ id: 1, name: 'Aman' });
```

For React, I use **React Testing Library**. It tests the component **the way a user uses it**: by what they can see and click, not by the internal state.

```jsx
render(<Counter />);
await userEvent.click(screen.getByRole('button', { name: /increase/i }));
expect(screen.getByText('Count: 1')).toBeInTheDocument();
```

**What makes a good test:** it tests **behavior, not implementation**, it is independent of other tests, it is fast, and it fails for a clear reason. **TDD** means writing the failing test first, then the code to pass it.

---

# Docker

## 9. What is Docker? What is the difference between an image and a container?

Docker **packages my app with everything it needs** (code, runtime, libraries) so that it **runs the same way everywhere**. It solves the problem: "it works on my machine."

- **Image:** a **read-only template** (like a recipe or a blueprint). Built from a `Dockerfile`.
- **Container:** a **running instance** of an image (the actual dish). I can start many containers from one image.

A container is **lighter than a virtual machine**, because it shares the host's operating system kernel instead of running a full OS.

---

## 10. How do I write a Dockerfile for a Node app?

```dockerfile
FROM node:22-alpine          # base image (use a current LTS version)
WORKDIR /app
COPY package*.json ./
RUN npm ci                   # install dependencies
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]    # command to start the app
```

**Why copy `package.json` first?** Docker caches each step as a **layer**. If only my code changes, the `npm ci` layer is reused from cache, so the build is much faster.

Basic commands:

```bash
docker build -t my-app .
docker run -p 3000:3000 my-app       # map host port 3000 to container port 3000
docker ps                            # list running containers
```

I also add a **`.dockerignore`** file (with `node_modules`, `.git`, `.env`) to keep the image small and safe.

---

## 11. What are volumes and Docker Compose?

- **Volume:** a place to **store data outside the container**. A container's own files disappear when it is removed, so I use a volume for things like database data.
- **Docker Compose:** a file (`docker-compose.yml`) that runs **many containers together** with one command, like my app, a database and Redis.

```yaml
services:
  app:
    build: .
    ports: ["3000:3000"]
    environment:
      DATABASE_URL: postgres://user:pass@db:5432/mydb
    depends_on: [db]
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: pass
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

Run everything with `docker compose up`. The containers can reach each other by **service name** (`db`).

---

## Last-Minute Revision

- `fetch` downloads only. `pull` is fetch plus merge.
- `merge` keeps the full history. `rebase` makes a straight line. Never rebase shared branches.
- `revert` is safe for shared branches. `reset --hard` deletes changes.
- Conflict: edit the file, remove the markers, `add`, `commit`.
- Testing pyramid: many unit, some integration, few E2E.
- Mock outside dependencies. Test behavior, not implementation.
- Image is the template. Container is the running instance.
- Copy `package.json` first in the Dockerfile for better layer caching.
- Volumes keep data. Compose runs many containers together.