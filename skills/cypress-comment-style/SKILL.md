---
name: cypress-comment-style
description: Enforce high-signal, minimal comment hygiene across Cypress spec files (*.cy.js) and Page Objects — self-documenting test titles, zero step narration, and mechanical gotchas documented once on the Page Object.
---

# Cypress Comment Style (`cypress-comment-style`)

Keep Cypress specs (`*.cy.js`) and Page Objects high-signal and free of noisy narration. Apply these rules whenever creating or editing test files.

---

## 1. What Is Allowed in Spec Files

1. **A terse file header (2–3 lines max)** stating what flow the spec covers and its source:
   ```js
   // End-to-end checkout and order confirmation walkthrough.
   ```
2. **A single line above a non-obvious local helper function** when its behavior isn't clear from its name:
   ```js
   // Flips a checkbox away from its initial state and returns the new boolean state.
   function toggleCheckbox($el) {
   ```
3. **`describe()` and `it()` titles carry the meaning:** Write `it()` titles that state the asserted outcome (`"rejects submission when contact email is malformed"`), not the mechanical steps taken.

---

## 2. What Is Not Allowed in Spec Files

- **No step-by-step narration:** Never write `// Step 1: Log in`, `// Click submit`, or `// Navigate to dashboard`. The code already expresses the sequence.
- **No decorative section banners:** Never insert `// ── Setup ──────────` or `// === Assertions ===` inside a spec. Use a single blank line to separate setup, action, and assertion phases.
- **No restating assertions in prose:** Never put `// Verify URL includes /checkout` above `cy.url().should('include', '/checkout')`.
- **No justifying why the test exists:** If a test needs a paragraph explaining why it matters, rewrite the `it()` title.

---

## 3. Where Mechanical Explanations Belong: The Page Object

Non-obvious DOM quirks, framework rendering behaviors, and safety warnings belong **once** on the Page Object getter — right next to the selector they justify, rather than repeated across every spec:

```js
class ProfilePage {
  // Date picker renders an aria-hidden input; target the interactive spinbutton sections instead.
  get dateOfBirthSpinbuttons() {
    return cy.get('[role="spinbutton"]');
  }

  // Dispatches real external emails in staging — never click in automated suites.
  get sendInviteButton() {
    return cy.get('[data-testid="send-invite-btn"]');
  }
}

export default new ProfilePage();
```

### Page Object Comment Rules
- **Keep:** Why a fallback selector was required, a DOM/framework gotcha (e.g., collapsed accordions staying mounted), or a side-effect safety warning.
- **Drop:** Section banners (`// ── Getters ──`), comments that merely restate the selector name, or repeated boilerplate notices across multiple files.
