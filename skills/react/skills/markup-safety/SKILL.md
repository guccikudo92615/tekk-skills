---
name: markup-safety
description: 'Audits what the rendered markup does to the user — unsanitised HTML injected into the DOM, and interactive elements without an accessible name, keyboard path, focus handling or correct semantics. Use when reviewing any component that renders user-supplied content or builds controls out of non-semantic elements.'
---

# Skill: markup-safety

**Skill:** the output, judged by the two users who cannot defend themselves — the
one whose browser executes whatever ends up in the DOM, and the one operating
the page without a mouse or without sight. With the UX loop unregistered, this
is the catalog's only accessibility skill: treat a violation here as a real
finding, not a nicety.

## What to evaluate

1. **Unsanitised HTML.** `dangerouslySetInnerHTML` (or a `ref` writing
   `innerHTML`) fed content that is interpolated, user-supplied, fetched, or
   produced by a model. Propose the sanitiser the app already uses, or a safe
   rendering path (render as text, or a Markdown renderer with HTML disabled).
   Check the sanitiser's *configuration* too — an allowlist that permits
   `onerror`, `javascript:` URLs, `<iframe>` or `<style>` is a sanitiser in name
   only.
2. **URLs and embeds from data.** An `href`/`src` built from user data without a
   scheme check (`javascript:`, `data:`), a `target="_blank"` without
   `rel="noopener"`, an embedded iframe with no sandbox. Small fixes, real
   exposure.
3. **Accessible name.** Every control has one: an icon-only button with no
   label, an `<img>` with no `alt` (or a decorative image that should have an
   empty one), a form input with no associated `<label>`, a link whose only text
   is "here". Name the element and the fix — a `<label htmlFor>`, an
   `aria-label`, visually-hidden text.
4. **Semantics and keyboard operability.** A `<div>` or `<span>` carrying an
   `onClick`: no focus, no Enter/Space activation, no role announced. The fix is
   almost always the real element (`<button>`, `<a>`), not a pile of ARIA. Check
   also for a positive `tabIndex`, a control removed from the tab order, and
   hover-only affordances with no keyboard equivalent.
5. **Focus management.** A dialog, drawer or route change that does not move
   focus to the new content, does not trap focus while open, and does not
   restore it on close; a `:focus` outline removed in CSS with nothing visible
   put back. These are the differences between usable and unusable for keyboard
   and screen-reader users.
6. **Dynamic content that is never announced.** Toasts, inline validation,
   async results and loading states rendered with no live region and no
   association to their control (`aria-live`, `aria-describedby`,
   `aria-invalid`), so a screen-reader user gets silence where a sighted user
   gets feedback.
7. **Structure.** Heading levels skipped or used for styling, a page with no
   landmark structure, a list built from `<div>`s, a table without headers, an
   `lang` attribute missing on the document. Cheap to fix, and they set the
   whole page's navigability.

## How to verify before you claim

- **Quote the element and name the rule.** "This `<div onClick>` has no keyboard
  path (WCAG 2.1.1)" or "this `<img>` has no `alt` (1.1.1)". A finding that
  cannot name the criterion or the concrete fix is a preference.
- **Check the component library first.** If the control comes from a headless UI
  library, the name, role, focus trap and keyboard handling may already be
  provided — read the imported component before proposing ARIA on top of it.
- **For an HTML-injection finding, follow the content to its source.** Static
  copy written by the team is a different severity from a field another user can
  fill in — say which one you established.
