---
name: modern-best-practice-svelte-components
description: Build clean, modern Svelte components that apply common best practices and avoid common pitfalls like unnecessary state or $effect usage
---
 
# Writing Svelte Components
 
We're using modern Svelte (5+) with runes, and we're following common best
practices focused on clarity, correctness, and maintainability.
 
## Component Structure & Style
 
- **PREFER** small, focused components with a single responsibility
- **PREFER** named `function` declarations over arrow functions for top-level
  handlers and helpers
  - Exception: anonymous callbacks, inline snippet bodies, and closures
- **PREFER** explicit prop typing with TypeScript (`let { ... }: Props = $props()`)
  where applicable
- Keep markup flat and readable; extract subcomponents or `{#snippet}` blocks
  instead of deeply nesting
- Group related logic together in `<script>`: props first, then state, derived
  values, handlers, and helpers
## State Management
 
- **AVOID** `$effect()`
  - See the ["You Might Not Need $effect" guide](references/you-dont-need-effect.md)
    for detailed guidance
  - **PREFER** `$derived()` (or `$derived.by()`) over synchronizing state in an effect
- **AVOID** unnecessary `$state()` usage
  - Derive values from props or other state with `$derived` when possible
  - Localize state to the lowest possible component
- **DO NOT** mirror props in state unless absolutely necessary
- `$state` is deeply reactive — mutate objects and arrays directly
  (`items.push(x)`, `user.name = 'x'`). Reassigning (`items = [...items, x]`)
  is unnecessary ceremony; trust the proxy.
- **PREFER** `$state.raw()` for large or frequently-replaced data structures
  (API responses, parsed blobs) that are swapped wholesale rather than mutated
  in place. It skips the deep-proxy wrapping and is meaningfully faster.
- For shared reactive state, put runes in a `.svelte.ts` (or `.svelte.js`)
  module and export them — this is the modern replacement for most legacy
  `writable`/`readable` store usage
## Rendering & Derivation
 
- **PREFER** `$derived` (or simple inline expressions) for computed values
- Reach for `$derived.by(() => { ... })` only when the computation needs
  multi-statement logic
- **AVOID** premature optimization — Svelte's compiler already handles most
  memoization concerns
- Keep template logic deterministic and free of side effects
## Event Handling
 
- Use Svelte 5's property-style event syntax (`onclick`, `oninput`, `onsubmit`),
  not the legacy `on:click` directive form
- **AVOID** in-line event handlers in markup
  - **PREFER**:
    ```svelte
    <script lang="ts">
      function handleClick() {
        // ...
      }
    </script>
 
    <button {onclick}>Click</button>
    <!-- or -->
    <button onclick={handleClick}>Click</button>
    ```
  - Over:
    ```svelte
    <button onclick={() => { /* ... */ }}>Click</button>
    ```
- Name handlers clearly (`handleSubmit`, `handleChange`, `handleClose`)
- Keep handlers small; extract complex logic into helpers
## Effects, Data, and Side Effects
 
- **AVOID** `$effect` for:
  - Derived state (use `$derived`)
  - Data transformations (compute them inline or with `$derived`)
  - Event-based logic that can live in handlers
- If side effects are unavoidable (e.g. imperative DOM APIs, subscriptions,
  third-party library integration), keep them minimal, isolated, and
  well-documented. Return a cleanup function from `$effect` when subscribing.
- Prefer framework-level or external abstractions (SvelteKit load functions,
  data libraries, stores with proper lifecycle) over raw `$effect`
- Inside `$effect` or `$derived`, every rune you read becomes a dependency.
  When you need to read a value *without* subscribing to it, wrap the read in
  `untrack(() => ...)` — cleaner than restructuring the code to avoid the
  dependency.
## Props & Composition
 
- **PREFER** composition via `children` and named `{#snippet}` blocks over
  configuration
- **AVOID** excessive boolean props; prefer expressive APIs (discriminated
  unions, enum-like string props)
- Use `children` and snippet props intentionally and document expected structure
- Keep prop names semantic and predictable
- Destructure props at the top of the script **for the initial read and
  defaults only**:
  ```svelte
  <script lang="ts">
    interface Props {
      label: string;
      disabled?: boolean;
      children?: import('svelte').Snippet;
    }
    let { label, disabled = false, children }: Props = $props();
  </script>
  ```
- **IMPORTANT:** destructured prop bindings are a non-reactive snapshot. If a
  prop changes in the parent, the local binding will not update. When you need
  the live value, either keep it as `props.count` (don't destructure), or wrap
  it: `const count = $derived(props.count)`. Reach for this whenever a prop
  feeds into `$derived`, `$effect`, or anywhere else reactivity matters.
## Performance & Stability
 
- Use keyed `{#each items as item (item.id)}` blocks when list items can
  reorder, be inserted, or removed; keep keys stable and meaningful
- Reach for `{#key ...}` blocks only when you genuinely need to force a
  subtree to remount
## General Principles
 
- Write code for humans first, compilers second
- Prefer explicitness over cleverness
- Optimize for readability and long-term maintenance
- If a pattern feels complex, reconsider the component boundary
