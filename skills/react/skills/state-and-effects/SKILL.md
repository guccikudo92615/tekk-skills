---
name: state-and-effects
description: 'Audits where component state lives and when effects run — array-index keys, missing or unstable effect dependencies, state that should be derived during render, effects doing work that belongs in an event handler, unhandled async races, and hooks called conditionally. Use when reviewing hooks, providers, forms, or any component holding state.'
---

# Skill: state-and-effects

**Skill:** identity and timing. Two questions asked of every component that
changed: *does React know which thing is which* (keys, reconciliation), and *does
this state exist at the right moment, from the right source* (derivation,
dependencies, effects). Render cost is the render-performance skill; you own
whether the state is right at all.

## What to evaluate

1. **Keys and identity.** An array index — or any value that changes on reorder,
   filter or insert — used as a list `key`. React then reuses the wrong DOM node
   and the wrong component state: the input keeps the previous row's text, the
   checked box moves. Propose a stable id from the data. Watch for the same bug
   in `<Fragment key>` and in keyed animation wrappers.
2. **Derived state stored in state.** A value computed from props or other state
   held in `useState` and re-synced by an effect. It renders one frame stale and
   drifts whenever a path forgets to sync. Derive it during render; if the
   computation is genuinely expensive, memoise the computation — not the storage.
   Related: subscribing to a raw value only to compare it, when the component
   could subscribe to the derived boolean and re-render far less often.
3. **Effect dependency arrays.** A referenced value missing from the array (a
   stale closure reading last render's props) or an unstable value in it (a new
   object/array/function each render, so the effect runs forever). Prefer
   depending on *primitives* rather than objects, and prefer the restructure
   that deletes the effect entirely over a longer dependency list. Split an
   effect whose dependencies are independent — one runs when the other's inputs
   change.
4. **Effects doing event work.** Logic that belongs in the handler that caused it
   — a POST fired by an effect watching a flag the click just set, analytics
   sent by an effect on mount, a toast triggered by watching state. Effects are
   for synchronising with something outside React; anything caused by an
   interaction belongs in the interaction. This also removes a whole class of
   double-fire under StrictMode's development double-invocation.
5. **Async effects with no race handling.** A fetch inside an effect whose
   response is written to state without checking that it is still the current
   request. Rapid input or navigation lands responses out of order and the older
   one wins. Propose an `AbortController` or an `ignore` flag set in the cleanup
   — and check the cleanup exists at all for subscriptions, timers and
   listeners.
6. **Rules of hooks.** A hook behind a condition, an early return, a loop or a
   callback; a hook order that changes between renders. Also: a component
   defined *inside* another component — every parent render creates a new
   component type, so React unmounts and remounts the whole subtree, losing its
   state (a correctness bug, not just a slow one).
7. **State initialisation and updates.** An expensive initial value computed on
   every render because it was passed as a value rather than a function to
   `useState`; an update that reads the current value from the closure instead of
   the functional form, which both goes stale under batching and forces the
   callback's identity to change.

## How to verify before you claim

- **Name the sequence that breaks it.** "Reorder the list and row 2's input
  keeps row 1's text"; "type twice quickly and the first response overwrites the
  second". A dependency-array finding without a scenario is a lint opinion.
- **Check for a compiler or lint rule first.** If the project runs the React
  Compiler, `eslint-plugin-react-hooks`, or a framework preset that already
  enforces the rule, a manual finding is redundant — say so and move on.
- **Read the whole hook, not the diff hunk.** The value that makes a dependency
  unstable is usually defined a few lines above the effect.
