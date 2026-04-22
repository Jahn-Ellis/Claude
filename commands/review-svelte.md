---
allowed-tools: Read(*), Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(bunx svelte-check:*), Bash(bun run check:*)
description: Review Svelte/SvelteKit code against project best-practice skills. Usage: /review-svelte [MODE] [path|ref]
argument-hint: "[MODE] [path|ref]"
---
 
# Svelte Code Review
 
Arguments: `$ARGUMENTS`
 
Take your time. Understand the code before writing findings. Explain findings
briefly and directly — no padding, no restating the code back.
 
## Parse arguments
 
`$ARGUMENTS` can contain, in order, an optional MODE and an optional target:
 
- **MODE** — uppercase token or comma-separated combination. Recognized:
  `BUGS`, `SECURITY`, `PERFORMANCE`. Example: `BUGS,SECURITY`.
- **target** — a file, directory, or git ref.
Both optional. If only one argument is given: if it matches a MODE token
(all uppercase, only recognized values or commas), it's MODE; otherwise
it's the target.
 
### MODE behavior
 
- `BUGS` — focus ONLY on logical, correctness, and reactivity bugs
- `SECURITY` — focus ONLY on security issues (auth, secrets, unvalidated
  input, CSRF, session handling, env module misuse)
- `PERFORMANCE` — focus ONLY on performance issues (unnecessary renders,
  missing `$state.raw` on heavy data, sequential loads that should be
  parallel, SSR hydration cost, oversized client bundles)
- Combined (e.g. `BUGS,SECURITY`) — review those areas together, skip others
- Anything else, or empty — **general thorough review** covering all of the
  above plus Svelte/SvelteKit idiom, maintainability, and design
### Target behavior
 
- **No target** → review the current uncommitted changes via `git diff HEAD`.
  If the working tree is clean, say so and stop.
- **File path** → review that file.
- **Directory path** → review every `.svelte`, `.svelte.ts`, `.svelte.js`,
  `.ts`, and `.js` file under it that looks like Svelte/SvelteKit code. If
  the directory is broad (`.`, `src`, project root), treat as codebase-wide
  review — explore file-by-file, understand the architecture, don't rush.
- **Git ref** (commit SHA, branch, tag) → run `git diff <ref>` against it.
If an argument is ambiguous (e.g. `main` could be a branch or a folder),
check if it exists as a path first; if not, treat as a ref.
 
State at the top of your review what you reviewed and in what mode.
 
## Standards
 
Consult the project's skills for what "good" looks like:
 
- **`modern-best-practice-svelte-components`** — component-level concerns
  inside `.svelte` files (runes, state, effects, props, event handling)
- **`modern-best-practice-sveltekit`** — framework-level concerns (routing,
  load functions, form actions, auth, env vars, adapters, hooks)
These skills are the source of truth. If a rule there conflicts with your
instincts, the skill wins. If something isn't covered by a skill, apply
general software engineering judgment but flag it as a judgment call rather
than a rule violation.
 
## Run static checks first
 
Before reviewing, run `bunx svelte-check` (or `bun run check` if the project
defines it). Include any errors or warnings in the review as pre-existing
findings under the relevant severity. Don't re-analyse them; just list them.
 
If the command fails to run or isn't installed, skip silently.
 
## What to look for
 
Focus on issues that actually matter. Scope depends on MODE. In a general
review, in rough priority order:
 
1. **Correctness** — bugs, broken reactivity (destructured props losing
   updates, `$state` mirroring props unnecessarily), wrong load function
   type (universal vs server), unvalidated session usage
2. **Security** — auth checks only on the client, secrets imported from
   the wrong `$env` module, user input flowing unvalidated into server
   operations, missing CSRF considerations
3. **SvelteKit idiom mismatches** — `+server.ts` + client `fetch` where a
   form action belongs, business logic in `+page.svelte`, React-style
   `useEffect` patterns, in-memory state in serverless deploys
4. **Svelte idiom mismatches** — `$effect` used for derived state,
   unnecessary `$state` where `$derived` would do, inline event handlers,
   legacy `on:click` syntax, missing `$state.raw` on large replaced data,
   missing keys on `{#each}` blocks
5. **Maintainability** — deeply nested markup, components doing too much,
   unclear prop APIs, business logic not colocated with its route
Skip pure style nits with no functional or clarity impact. Skip anything a
linter would catch (unless `svelte-check` surfaced it above).
 
## Output format
 
Use this structure. Omit any severity group that has no findings.
 
```
# Svelte Review
 
**Scope:** <what you reviewed>
**Mode:** <BUGS | SECURITY | PERFORMANCE | combined | general>
 
## 🚫 Blockers
 
Bugs, security problems, or behavior that will break in production.
 
### <short title>
**File:** `path/to/file.svelte:<line>`
**Problem:** <one sentence: what's wrong and why it matters>
 
<short code excerpt showing the issue>
 
**Fix:**
<concrete suggested code — actual code, not prose>
 
---
 
## ⚠️ Issues
 
Clear skill violations or meaningful design problems. Not release-blocking.
 
<same format>
 
---
 
## 💡 Nits
 
Minor improvements. Take or leave. Keep these to one or two lines each.
 
<same format, compressed>
 
---
 
## ✅ Summary
 
- **Blockers:** N · **Issues:** N · **Nits:** N
- <One or two sentences: overall impression, or a cross-cutting pattern
  worth calling out if multiple findings share a root cause.>
```
 
## Review quality guidelines
 
- **Be specific.** Cite file and line. "Uses `$effect` incorrectly" is
  useless. "Line 23 uses `$effect` to compute `filteredItems` from `items`
  and `query`, but this is a derivation" is useful.
- **Show the fix as code.** For blockers and issues, concrete replacement
  code. No prose descriptions of the change.
- **Cite the skill rule when applicable** in the Problem line.
- **One problem per finding.** Three issues in one block → three findings.
- **Quote the actual code** in excerpts, don't paraphrase.
- **Don't pad.** Omit empty severity groups. No "great work!" or
  "consider improving." Just findings.
If the target is empty or not Svelte/SvelteKit code, say so directly and
stop — don't invent findings.
