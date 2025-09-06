---

# 🛠 Step 1: Create Repo

```bash
mkdir monitoring-platform-be
cd monitoring-platform-be
git init
npm init -y
```

**Why?**

* `mkdir` → makes a folder for the backend project.
* `git init` → starts version control (so we can track changes, rollback, and collaborate later).
* `npm init -y` → creates a `package.json` (this is your project’s manifest: lists dependencies, scripts, metadata).

---

# 🛠 Step 2: Install Dependencies

We installed two types of dependencies:

### **Regular dependencies** (needed when app runs in production):

```bash
npm install express dotenv
```

* **express** → lightweight web framework to handle routes, APIs, middlewares.
* **dotenv** → loads environment variables from `.env` file (keeps secrets like DB passwords out of code).

### **Dev dependencies** (only used while developing, not needed in production):

```bash
npm install -D typescript ts-node-dev @types/node @types/express eslint prettier eslint-config-prettier eslint-plugin-prettier
```

* **typescript** → allows us to write strongly typed code, better maintainability.
* **ts-node-dev** → runs `.ts` files directly and restarts automatically when code changes (developer productivity).
* **@types/node** → gives TypeScript type definitions for Node.js built-in APIs (e.g., `fs`, `path`).
* **@types/express** → gives Express type definitions (so TypeScript understands `req`, `res`, `next`).
* **eslint** → tool to check code for errors/bad practices (static analysis).
* **prettier** → formats code consistently (spaces, commas, quotes).
* **eslint-config-prettier + eslint-plugin-prettier** → makes ESLint and Prettier work together without conflict.

👉 **Rule of thumb:**

* If something is required at runtime (like `express`), install as normal dependency.
* If it’s only for dev (like `typescript`), install as `-D` (dev dependency).

---

# 🛠 Step 3: Setup TypeScript

```bash
npx tsc --init
```

This generates a `tsconfig.json`.
We customized it:

```json
{
  "compilerOptions": {
    "target": "ES2020",                // Which JS version to compile down to
    "module": "commonjs",              // Node.js module system
    "outDir": "dist",                  // Compiled .js files go here
    "rootDir": "src",                  // Source .ts files live here
    "strict": true,                    // Enables strict type checking
    "esModuleInterop": true,           // Lets us use modern import/export with CommonJS libs
    "skipLibCheck": true,              // Skips type-checking of node_modules for faster builds
    "forceConsistentCasingInFileNames": true // Avoids import bugs in case-sensitive OS
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

**Why?**

* TypeScript doesn’t run directly on Node, it must be compiled to JavaScript.
* `tsc` (TypeScript compiler) does this compilation.
* Config helps us control *how* code compiles.

---

# 🛠 Step 4: Setup ESLint + Prettier

We added `.eslintrc.json` and `.prettierrc` configs.

**Why?**

* ESLint catches bugs early (e.g., unused vars, missing `await`).
* Prettier keeps code format consistent, so team members don’t argue about spaces vs tabs.
* Together: consistent, safe codebase.

---

# 🛠 Step 5: Starter Folder Structure

We created minimal `src/` with `app.ts` and `server.ts`.

```
src/
 ├── app.ts
 └── server.ts
```

**Why split?**

* `app.ts` → defines Express app (routes, middleware).
* `server.ts` → actually starts the server (bootstraps app).
* This separation makes testing easier (we can import app without running server).

---

# 🛠 Step 6: NPM Scripts

In `package.json`:

```json
"scripts": {
  "dev": "ts-node-dev --respawn --transpile-only src/server.ts",
  "build": "tsc",
  "start": "node dist/server.js",
  "lint": "eslint . --ext .ts"
}
```

**Why?**

* `dev` → start dev server with hot reload.
* `build` → compiles TypeScript → JavaScript into `dist/`.
* `start` → runs compiled code (used in production).
* `lint` → run ESLint across project.

---

# 🛠 Step 7: Verify

Run:

```bash
npm run dev
```

Go to `http://localhost:4000` → see `Hello Monitoring Platform 🚀`.

---

✅ Now you understand *why each step was needed*.

👉 Next step (as we planned) will be:

* Add `config/` folder (for env + DB connection).
* Setup MongoDB connection with Mongoose.
* Add basic error handling + logging.
