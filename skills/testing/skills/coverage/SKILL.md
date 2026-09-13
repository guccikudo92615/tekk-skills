---
name: coverage
description: 'Audits changed behaviour for the tests it does not have — new branches and error paths, bug fixes with no regression test, new edge and boundary cases, contracts whose existing tests no longer assert them, and logic with no single expected output that needs a property-based test instead. Use when product code changed and the question is whether a regression would be caught.'
---

# Skill: coverage

**Skill:** the regression that ships silently. For every behaviour this diff adds
or changes, ask: *if someone broke this next month, which test turns red?* When
the answer is "none", you have a candidate — and the finding is only worth
proposing when that silent break would actually cost something.

## What to evaluate

1. **New untested branches and error paths.** A new conditional, early return,
   `catch`, or fallback with nothing exercising it. Error paths are the usual
   gap: the happy path gets a test because it was easy to write, and the branch
   that only runs when the provider is down never does.
2. **Bug fixes with no regression test.** A change that corrects behaviour but
   pins nothing. This is the highest-value class in the whole skill: the bug
   already happened once, so its probability is not hypothetical, and without a
   test the fix can be reverted by a refactor with nobody noticing.
3. **New edge and boundary cases.** Empty, null/undefined, zero, negative, the
   maximum, duplicate input, the second concurrent call. The change introduced
   the boundary; the test suite has not caught up with it.
4. **Changed contracts.** A function, endpoint or component whose inputs or
   outputs changed while its existing tests still assert the OLD shape and pass
   anyway — the worst case, because the suite is now actively reassuring.
5. **Oracle-problem code.** Logic with no single exact expected output —
   ranking, search, transforms, encoders, calculations. A fixed-value test is
   brittle or impossible; propose a **metamorphic or property-based** test
   instead: assert how outputs *relate* under an input transformation (adding an
   item never lowers the count, reverse-then-reverse is identity, output stays
   within bounds for generated inputs). Generate the inputs rather than
   hand-listing cases.
6. **The test that cannot fail.** An assertion that holds for any
   implementation (`expect(result).toBeDefined()`), a mock asserted against
   itself, a snapshot regenerated on every change. Coverage that measures
   nothing is worse than a known gap, because it reads as covered.

## How to verify before you claim

- **Read the existing test file for the changed module first.** Half of what
  looks untested is covered from a neighbouring case, a parameterized table, or
  an integration test one level up. Name the file you checked.
- **Apply the mutation test in your head.** Take the specific line you say is
  uncovered, imagine a plausible wrong version of it (flip the comparison, drop
  the `await`, return early), and ask which existing test would fail. If one
  would, there is no gap. If your PROPOSED test would not, it is placebo — do
  not propose it.
- **Say what the regression costs.** "This branch decides whether the webhook is
  retried; if it inverts, we drop paid orders" is a sized finding. "No test for
  this function" is not.
- **Name the concrete cases, not a wish.** The happy path plus the load-bearing
  failure mode, with the inputs and the expected result written out — enough
  that the test could be typed from the proposal.
