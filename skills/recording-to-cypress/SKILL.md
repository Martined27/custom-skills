---
name: recording-to-cypress
description: Convert a noisy Chrome DevTools Recorder JSON export into a clean, idiomatic end-to-end Cypress walkthrough spec using Page Objects, data-testid selectors, and text-independent assertions.
argument-hint: "Path to the Chrome Recorder JSON export, e.g. recordings/checkout-flow.json"
---

# Chrome Recorder JSON to Cypress Spec (`recording-to-cypress`)

Turn a raw Chrome DevTools Recorder JSON export into a runnable, idiomatic Cypress end-to-end spec — not a literal step-by-step transcription.

Also load the `cypress-comment-style` skill to govern comments in generated specs and Page Objects.

---

## 1. One Recording, One Walkthrough

A recording captures a user journey, so it becomes a single `it()` covering that journey end to end. Do not split a continuous flow into granular per-field specs that repeatedly re-run login and navigation.

Add a second `it()` only for a negative-path outcome that cannot coexist with the happy path (e.g., a rejected submission or blocked save).

---

## 2. Triage Recorder Noise

Read the full JSON export first. Chrome Recorder output is noisy — discard steps that do not drive application state:
- **Intermediate `change` events** on the same field (`"J"` → `"Jane"` → `"Jane Doe"`) — keep only the final committed value.
- **`keyDown` / `keyUp` pairs** for `Tab` (field traversal), modifier keys (`Meta`, `Shift`), or `Enter` when the preceding `change` step already submits or updates the field.
- **No-op toggles:** Clicks that open and immediately close a control without changing state.
- **Browser extension overlays:** Anything targeting `chrome-extension://...` (password managers, autofill popups).
- **Mis-clicks dismissed via backdrop:** Stray clicks immediately undone by clicking `.MuiBackdrop-root` or a modal overlay.

Report which steps were discarded as noise and why.

---

## 3. Selector Hierarchy (Never Trust Raw Recorder Selectors)

Recorder exports frequently emit brittle selectors: framework-generated IDs (`#:r1g:`), hashed CSS classes (`div.css-95g4uk > button`), or deep positional chains.

Prefer selectors in this strict order:
1. `[data-testid="..."]`
2. Stable semantic `id` or `name` attribute (verify it is not auto-generated per render)
3. `<label for="...">` association resolved at runtime
4. ARIA role or state attribute (`[role="combobox"]`, `[aria-invalid="true"]`)
5. Visible button text only when no structural attribute exists (flag missing `data-testid` as a testability gap)

---

## 4. Reuse Page Objects and Setup Helpers

Before writing new code:
- Check `cypress/pages/*.js` (or `cypress/e2e/<domain>/pages/*.js`) for existing Page Objects covering the target routes.
- Check `cypress/support/` or domain helpers for existing setup flows (`login()`, API data seeding) and call them instead of re-deriving login steps in UI.
- New Page Objects should use a PascalCase class, a `url` property, element getters, and a singleton export (`export default new PageName()`).

---

## 5. Text-Independent Functional Assertions

Write assertions that verify application state and navigation rather than brittle UI copy:
- **Navigation:** Assert `cy.url().should('include', Page.url)` using `assertedEvents` from the recording step.
- **Dropdowns:** Select and verify by index or `data-value` / `aria-selected`, not translated display text.
- **Checkboxes / Toggles:** Flip away from the initial state and assert the new state so the test proves it caused the mutation.
- **Validation errors:** Assert the URL did not advance plus `aria-invalid="true"`, rather than matching exact error prose.
- **Unique test data:** Generate per-run values (`uniqueValue('ITEM')`) so repeated runs do not collide on shared environments.

---

## 6. Scrub Credentials & Side Effects

Recorder files frequently contain literal typed usernames, emails, and passwords:
- Never hardcode credentials in a spec — always substitute `Cypress.env('TEST_USERNAME')` / `Cypress.env('TEST_PASSWORD')`.
- Replace any recorded personal email addresses with safe synthetic placeholders (`user@example.com`).
- Never click controls in automated runs that trigger real external emails/notifications — exercise the form state and cancel out.

---

## 7. Verify Before Reporting Done

1. Syntax-check the generated spec: `node --input-type=module --check < path/to/spec.cy.js`.
2. Run the spec headlessly: `npx cypress run --spec "path/to/spec.cy.js"`.
3. Report: discarded noise steps, reused Page Objects/helpers, newly added selectors, any missing `data-testid` gaps, and final verification status.
