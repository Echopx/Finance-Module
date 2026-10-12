# Design System

<!-- HINT: Owner: Documentation member, working with the Frontend member.
     A design system is the set of rules and reusable pieces that make every screen look and behave the same way.
     All 8 modules form ONE College Management System, so ask your class or instructor whether there is a
     shared design system (logo, colors, navbar). If there is, link it below and only document what is
     specific to your module. The values in the tables are examples. Replace them. -->

- **Shared class design system:** [link, or "None, we define our own below"]
- **Design file (Figma, etc.):** [link]

## 1. Design Principles

<!-- HINT: 2–4 short rules that help you make design decisions.
     Examples: "Clarity over decoration", "Works on a phone", "Every action gives feedback". -->

1. [Principle]
2. [Principle]

## 2. Colors

<!-- HINT: Define colors as tokens (named variables). In code, always use the token, never the raw hex value.
     Check text contrast at https://webaim.org/resources/contrastchecker/ and aim for at least 4.5:1. -->

| Token | Value | Use for |
|-------|-------|---------|
| `--color-primary` | `#[hex]` | Main buttons, links, active items |
| `--color-primary-hover` | `#[hex]` | Hover state of primary items |
| `--color-secondary` | `#[hex]` | Secondary buttons, accents |
| `--color-bg` | `#[hex]` | Page background |
| `--color-surface` | `#[hex]` | Cards, modals, tables |
| `--color-text` | `#[hex]` | Main text |
| `--color-text-muted` | `#[hex]` | Helper text, placeholders |
| `--color-border` | `#[hex]` | Borders, dividers |
| `--color-success` | `#[hex]` | Success messages, "Paid" badges |
| `--color-warning` | `#[hex]` | Warnings, "Pending" badges |
| `--color-danger` | `#[hex]` | Errors, delete buttons |

How tokens look in code:

```css
:root {
  --color-primary: #1d4ed8;
  --color-danger: #b91c1c;
}

.btn-primary {
  background: var(--color-primary);
}
```

## 3. Typography

- **Font family:** [e.g., "Inter", sans-serif]
- **Where it comes from:** [e.g., Google Fonts link]

| Style | Size | Weight | Line height | Use for |
|-------|------|--------|-------------|---------|
| Heading 1 | [32px] | [700] | [1.2] | Page titles |
| Heading 2 | [24px] | [600] | [1.3] | Section titles |
| Heading 3 | [20px] | [600] | [1.4] | Card titles |
| Body | [16px] | [400] | [1.5] | Normal text |
| Small | [14px] | [400] | [1.4] | Helper text, table notes |

## 4. Spacing, Corners, and Shadows

<!-- HINT: Use a fixed scale so spacing looks consistent. A 4px or 8px base is common. -->

| Token | Value |
|-------|-------|
| `--space-1` / `--space-2` / `--space-3` / `--space-4` / `--space-6` / `--space-8` | [4 / 8 / 12 / 16 / 24 / 32 px] |
| `--radius-sm` / `--radius-md` / `--radius-lg` | [4 / 8 / 16 px] |
| `--shadow-sm` / `--shadow-md` | [e.g., `0 1px 2px rgb(0 0 0 / 0.1)`] |

## 5. Layout and Breakpoints

| Name | Screen width | Layout notes |
|------|--------------|--------------|
| Mobile | below [768px] | [e.g., one column, hamburger menu] |
| Tablet | [768px] to [1023px] | [...] |
| Desktop | [1024px] and up | [e.g., sidebar and content area, max content width 1200px] |

## 6. Components

<!-- HINT: List every reusable component. Add a screenshot of each one (save it in docs/images/). -->

| Component | Variants | States | Used on |
|-----------|----------|--------|---------|
| Button | primary, secondary, danger, text | default, hover, focus, disabled, loading | Everywhere |
| Text input | text, email, password, number | default, focus, error, disabled | Forms |
| Select / dropdown | | | |
| Table | | | |
| Card | | | |
| Modal / dialog | | | |
| Alert / toast | success, info, warning, error | | |
| Navbar / sidebar | | | |
| Badge / status pill | [e.g., Paid, Unpaid, Overdue] | | |
| Pagination | | | |

## 7. Patterns

- **Forms:** label above each input, `*` for required fields, error message in red below the field, submit button disabled while sending.
- **Every screen that loads data has 4 states:** loading (spinner or skeleton), success, empty ("No announcements yet" plus a next step), and error (message plus a "Try again" button).
- **Destructive actions** (delete, cancel) ask for confirmation in a dialog.
- **Formats:** dates as [e.g., Sep 25, 2026], money as [e.g., ₱1,500.00].

## 8. Icons and Images

- **Icon set:** [e.g., Lucide, Material Symbols]
- **Logo usage:** [where and at what size]
- Every meaningful image has alt text.

## 9. Accessibility Checklist

- [ ] Text contrast is at least 4.5:1
- [ ] Every input has a visible label
- [ ] Everything works with the keyboard alone, and you can see which element is focused
- [ ] Images have alt text
- [ ] Color is never the only signal (add an icon or text, e.g., "❌ Unpaid")

## 10. Voice and Tone

- Simple, friendly, direct.
- Buttons say what they do: "Save changes", not "OK".
- Error messages say what went wrong **and** how to fix it: "Title is required", not "Invalid input".
