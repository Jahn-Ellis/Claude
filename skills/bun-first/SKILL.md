---
name: bun-first
description: Default to Bun — its runtime, package manager, test runner, bundler, and native APIs — instead of Node.js, npm/pnpm/yarn, jest/vitest, webpack/esbuild, or Node's stdlib. Use this skill whenever the task involves running JavaScript/TypeScript, installing packages, writing a script or dev server, reading/writing files, spawning processes, querying SQLite/Postgres/MySQL/Redis, hashing passwords, bundling, or reaching for a Node.js API — even if the user doesn't explicitly say "Bun". Bun is assumed to be available.
---
 
# Bun-First Development
 
Bun is the default runtime, package manager, test runner, and bundler. It runs TypeScript and JSX directly with no build step, ships a growing set of native APIs that replace whole categories of npm dependencies, and is dramatically faster than the Node + npm combo for installs, cold starts, and most server workloads. The point of this skill is to keep you from reflexively reaching for Node-era defaults when Bun has a better built-in.
 
## Commands: use these, not their Node equivalents
 
| Instead of         | Use                      |
| ------------------ | ------------------------ |
| `node script.ts`   | `bun script.ts` (runs TS/JSX directly) |
| `ts-node`, `tsx`   | `bun` (no transpile step) |
| `npm install`      | `bun install`            |
| `npm i <pkg>`      | `bun add <pkg>`          |
| `npm i -D <pkg>`   | `bun add -d <pkg>`       |
| `npm uninstall`    | `bun remove`             |
| `npm run <script>` | `bun run <script>` (or just `bun <script>`) |
| `npx <tool>`       | `bunx <tool>`            |
| `jest`, `vitest`   | `bun test`               |
| `nodemon`, `tsx watch` | `bun --hot script.ts` |
| `webpack`, `esbuild`, `tsup` | `bun build`    |
 
Use `bun.lock` (text-based, diffable) — don't commit `package-lock.json` or `pnpm-lock.yaml`. If one of those exists when adopting Bun, delete it and run `bun install`.
 
## APIs: prefer Bun-native over Node stdlib
 
These are genuinely Bun-specific (not just Node APIs Bun re-implements). They're faster, have better ergonomics, and remove dependencies.
 
**Files** — `Bun.file(path)` returns a lazy `BunFile` (a `Blob` subclass). Nothing is read until you call `.text()`, `.json()`, `.arrayBuffer()`, `.bytes()`, or `.stream()`. To write, use `Bun.write(dest, data)` where `dest` can be a path, `BunFile`, or URL, and `data` can be a string, buffer, `Response`, or another `BunFile`.
 
```ts
const config = await Bun.file("config.json").json();
await Bun.write("out.txt", "hello");
await Bun.write("copy.png", Bun.file("original.png")); // efficient copy
```
 
Skip `fs.readFile`, `fs.writeFile`, `fs/promises` for the common cases.
 
**HTTP server** — `Bun.serve({ port, fetch, routes })` for servers. It uses standard `Request`/`Response` objects. Skip Express/Fastify for anything where their middleware ecosystem isn't load-bearing — `Bun.serve` has built-in routing, WebSockets, and HTML imports.
 
**SQL** — `import { sql } from "bun"` for Postgres and MySQL with tagged-template parameterization built in. `import { Database } from "bun:sqlite"` for SQLite. Skip `pg`, `mysql2`, `better-sqlite3`.
 
**Redis** — `import { redis } from "bun"`. Skip `ioredis`/`node-redis` unless you need a specific feature they have and Bun doesn't.
 
**S3** — `Bun.s3` / `Bun.S3Client`. Skip `@aws-sdk/client-s3` for basic object storage.
 
**Passwords** — `Bun.password.hash(pw)` / `Bun.password.verify(pw, hash)`. Uses argon2id by default. Skip `bcrypt` and `argon2`.
 
**Subprocesses** — `Bun.spawn([...])` or the shell: `` Bun.$`ls -la` ``. The shell form is cross-platform (works on Windows) and handles escaping. Skip `child_process` and `execa`.
 
**Hashing** — `Bun.hash()`, `Bun.CryptoHasher`. Skip `crypto.createHash` for non-cryptographic use.
 
**Env** — `Bun.env` (or `process.env` — both work). `.env`, `.env.local`, `.env.production` are loaded automatically. Skip `dotenv`.
 
**Glob** — `new Bun.Glob("**/*.ts").scan(".")`. Skip `fast-glob`, `glob`.
 
## When to stick with a Node API or npm package
 
Don't fight the ecosystem on things Bun doesn't have a native answer for: ORMs (Drizzle, Prisma), validation (Zod), frameworks you actually want (Hono, Elysia), and anything where a mature package is materially better than what you'd write. The rule is "prefer Bun-native when Bun has one," not "avoid all dependencies."
 
If you find yourself installing a package whose entire job is something in the table above — a SQLite driver, a password hasher, a file-reading helper, a dotenv loader — stop and use the built-in.
 
## Scripts in package.json
 
Keep them thin. Bun runs TS directly, so most "build step for dev" scripts aren't needed:
 
```json
{
  "scripts": {
    "dev": "bun --hot src/index.ts",
    "start": "bun src/index.ts",
    "test": "bun test",
    "build": "bun build src/index.ts --outdir dist --target bun"
  }
}
```
 
No `ts-node`, no `nodemon`, no separate `build:watch`.
 
