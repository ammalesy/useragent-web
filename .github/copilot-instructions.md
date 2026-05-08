# GitHub Copilot Instructions

## Project Overview
**useragent-web** is a static single-page website that inspects and displays the visitor's browser `User-Agent` string. It parses the raw UA into human-readable fields and token badges.

## Stack
- **HTML5** – single `index.html`, no build step
- **Tailwind CSS** – loaded via CDN (`cdn.tailwindcss.com`)
- **Vanilla JavaScript** – all logic inline in `index.html`
- **Vercel** – static deployment, configured via `vercel.json`

## Project Structure
```
useragent-web/
├── index.html          # Entire app (markup + styles + JS)
├── vercel.json         # Vercel static deployment config
└── .github/
    └── copilot-instructions.md
```

## Architecture & Key Concepts

### UA Parsing (`parseUA`)
Returns an array of field objects `{ label, value, sub, color }`.  
Add new fields by pushing to the `fields` array inside `parseUA()`.  
Browser / OS detection uses ordered regex lists; more specific entries come first.

### UA Tokenizing (`tokenizeUA`)
Splits the raw UA string into parenthesised group tokens (`paren`) and product tokens (`product`), rendered as badge pills at the bottom of the page.

### Rendering
- `renderCard(field)` → HTML string for a parsed-value card
- `renderToken(token)` → HTML string for a token badge
- `colorMap` maps color keys (e.g. `"indigo"`) to Tailwind class strings

### Copy Button
`copyUA()` copies `navigator.userAgent` to clipboard and toggles the button to a green "Copied!" state for 2 s.

## Coding Guidelines
1. Keep everything in `index.html` unless the feature genuinely warrants a new file.
2. Use Tailwind utility classes; avoid writing custom CSS unless absolutely necessary.
3. Prefer `const` / `let`; avoid `var`.
4. All UA-parsing logic lives inside `parseUA()` and `tokenizeUA()` — extend those functions rather than adding ad-hoc scripts.
5. Accessibility: use semantic HTML (`<section>`, `<header>`, `<main>`, `<footer>`), `aria-label` on interactive elements.
6. Dark theme is intentional — maintain `bg-gray-950` baseline.

## Adding a New Parsed Field
```js
// Inside parseUA(ua), before the return statement:
fields.push({
  label: "Field Label",          // shown in card header
  value: "Detected Value",       // main text
  sub:   "optional detail",      // small text below value (can be "")
  color: "indigo",               // key in colorMap
});
```

## Deployment
Push to the connected GitHub repository — Vercel auto-deploys on every push to `main`.  
No environment variables or secrets are required; the app is fully client-side.
