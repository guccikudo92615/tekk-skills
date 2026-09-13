---
name: render-performance
description: 'Audits what a component costs to render and how widely it re-renders others — unstable props and context values, over-broad subscriptions, expensive work in render, long unvirtualised lists, and updates that block typing. Use when reviewing lists, tables, charts, modals, providers, or a component that feels slow.'
---

# Skill: render-performance

**Skill:** cost per render, and blast radius per update. Whether the state is
*correct* is the state-and-effects skill; you own how much work the correct state
causes. Bundle size, data-fetch waterfalls and server latency belong to the
Performance loop — stay inside the render.

Enrichment note: the dimensions below draw on Vercel's published React
performance rule set (`vercel-labs/agent-skills` →
`vercel-react-best-practices`), re-stated for a code-reading audit rather than a
codemod.

## What to evaluate

1. **Unstable values crossing a boundary.** A fresh object, array, function or
   JSX element created during render and passed as a prop, a context value, or a
   dependency. Every consumer re-renders even though nothing changed. The
   highest-value instance is a **context provider whose value identity churns**:
   one keystroke re-renders every consumer in the tree. Hoist constants out of
   the component, memoise the value, or split the context so consumers subscribe
   to less.
2. **Subscribing to more than the component uses.** A component that reads a
   whole store, a whole context, or a raw value it only needs inside a callback,
   and therefore re-renders on every change to any of it. Read transient values
   through a ref, subscribe to the derived slice, or move the read into the
   event handler where it is actually used.
3. **Expensive work in the render path.** Sorting, filtering, formatting,
   parsing or date maths recomputed on every render over a non-trivial list;
   `new Date()`/regex/`Intl` objects constructed inline. Memoise the computation
   with primitive dependencies — but do not memoise a trivial expression, where
   the hook costs more than the work it saves.
4. **Long lists rendered whole.** Hundreds or thousands of rows mounted at once,
   with per-row handlers and per-row memo objects. Propose virtualisation with
   the library the app already uses, or CSS `content-visibility` for the cheap
   win when the rows are simple. Say the row count you found and where it comes
   from.
5. **Updates that block interaction.** A filter, search or tab switch that
   recomputes a heavy subtree synchronously while the user is typing. Propose a
   transition (`startTransition`/`useTransition`) or a deferred value so input
   stays responsive, and use the transition's pending state for the loading
   affordance rather than a separate flag.
6. **Re-mount churn.** A `key` that changes when it need not (remounting a whole
   subtree and losing its state), an inline component definition (see
   state-and-effects §6 — it is also the single most expensive re-render bug),
   or a conditional that swaps element types unnecessarily.
7. **Render-time hazards that look cosmetic.** `{count && <Row/>}` renders a
   literal `0` when the count is zero — use a ternary or an explicit boolean.
   Static JSX rebuilt each render can be hoisted to module scope.

## How to verify before you claim

- **Name the trigger, the subtree and the frequency.** "Typing in the search box
  re-renders all 400 rows because `onSelect` is a new function each render" is a
  finding; "this could be memoised" is not.
- **Size it from what the code shows.** Row counts from the query limit or the
  fixture, subscriber counts from the consumers of that context. Do not invent a
  millisecond figure you did not measure — that is Performance's connector-fed
  job, not yours.
- **Check for the React Compiler.** If the project compiles memoisation
  automatically, manual `useMemo`/`useCallback` proposals are noise; the
  structural findings (context splitting, virtualisation, inline components)
  still stand.
