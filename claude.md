# Frontend Battle 3.0 — CLAUDE.md

## MANDATORY FIRST STEP (every session, no exceptions)

Before writing a single line of code:

1. List all files in `assets/` — SVGs specifically
2. Confirm every SVG is accounted for and plan where each one is used
3. Only then begin building

If any asset file is missing or unreadable, stop and ask before proceeding.

---

## Fonts (load via Google Fonts, both required)

```html
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;600;700&family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
```

- **JetBrains Mono** → Headers, hero title, countdown timers, code blocks, pricing numbers
- **Inter** → Body text, UI labels, nav, descriptions, form elements

```css
:root {
  --font-heading: 'JetBrains Mono', monospace;
  --font-body: 'Inter', sans-serif;
}
```

---

## Color Palette (all 5 must be used)

```css
:root {
  --forsythia:      #FFC801; /* Primary accent — CTAs, highlights, active states */
  --deep-saffron:   #FF9932; /* Secondary accent — gradients, hover states */
  --nocturnal:      #114C5A; /* Dark teal — headers, dark sections */
  --oceanic-noir:   #172B36; /* Darkest — backgrounds, cards */
  --arctic-powder:  #F1F6F4; /* Light bg — light sections */
  --mystic-mint:    #D9E8E2; /* Subtle bg — dividers, muted areas */
}
```

### Usage guide

- **Dark theme base:** `--oceanic-noir` bg, `--nocturnal` cards
- **Hero gradient:** `--forsythia` → `--deep-saffron` (yellow to orange)
- **CTAs:** `--forsythia` bg with `--oceanic-noir` text
- **Light sections:** `--arctic-powder` or `--mystic-mint` bg
- **Pricing highlight:** `--forsythia` border on recommended tier

---

## Project Brief

Build a premium AI SaaS landing page for an AI data automation platform.

**Deployment:** Must be live (Vercel/Netlify) + public GitHub repo + demo video ≤100MB.

**Required sections:**

- Hero
- Feature showcase (Bento → Accordion)
- Pricing tier matrix
- Social proof

---

## Feature 1: Currency/Billing Switcher (30 pts total)

### Data matrix — use exactly this structure, no hardcoded UI values

```js
const PRICING_MATRIX = {
  tiers: ['starter', 'pro', 'enterprise'],
  base_monthly_usd: {
    starter: 29,
    pro: 79,
    enterprise: 199
  },
  currency_tariffs: {
    USD: { symbol: '$', multiplier: 1 },
    INR: { symbol: '₹', multiplier: 83.5 },
    EUR: { symbol: '€', multiplier: 0.92 }
  },
  annual_discount: 0.20
}

function computePrice(tier, currency, cycle) {
  const base = PRICING_MATRIX.base_monthly_usd[tier]
  const tariff = PRICING_MATRIX.currency_tariffs[currency]
  const discount = cycle === 'annual' ? (1 - PRICING_MATRIX.annual_discount) : 1
  return {
    symbol: tariff.symbol,
    amount: Math.round(base * tariff.multiplier * discount)
  }
}
```

### CRITICAL — State isolation (15 pts, Chrome DevTools verified)

Changing currency or billing cycle must ONLY update price text nodes.
NEVER trigger parent re-render or layout reflow.

**React pattern:**

```jsx
const priceRefs = useRef({}) // ref on each price <span>

function updatePrices(currency, cycle) {
  PRICING_MATRIX.tiers.forEach(tier => {
    const { symbol, amount } = computePrice(tier, currency, cycle)
    if (priceRefs.current[tier]) {
      priceRefs.current[tier].textContent = `${symbol}${amount}`
    }
  })
}

// In JSX — no reactive price state
<span ref={el => priceRefs.current[tier] = el}></span>
```

**Vanilla pattern:**

```js
function updatePrices(currency, cycle) {
  PRICING_MATRIX.tiers.forEach(tier => {
    const { symbol, amount } = computePrice(tier, currency, cycle)
    document.querySelector(`[data-price="${tier}"]`).textContent = `${symbol}${amount}`
  })
}
```

---

## Feature 2: Bento → Accordion with Context Transfer (10 pts)

### Rules

- Desktop (>768px): Bento grid layout
- Mobile (≤768px): Accordion list
- ZERO external UI/animation libraries — written from scratch only
- Banned: Framer Motion, Radix, Shadcn, HeadlessUI, any pre-built component

### Context transfer constraint (the differentiator)

If user is hovering bento card at index N and resizes past mobile breakpoint,
accordion must open at index N automatically.

```js
let activeIndex = 0 // mutable ref, NOT state

// Bento hover tracking
bentoCards.forEach((card, i) => {
  card.addEventListener('mouseenter', () => { activeIndex = i })
})

// Breakpoint watcher
const ro = new ResizeObserver(([entry]) => {
  if (entry.contentRect.width <= 768) {
    openAccordionAt(activeIndex)
  }
})
ro.observe(document.documentElement)

function openAccordionAt(index) {
  accordionPanels.forEach((panel, i) => {
    panel.style.maxHeight = i === index ? panel.scrollHeight + 'px' : '0'
    panel.classList.toggle('open', i === index)
  })
}
```

### Animation constraints (enforced)

```css
/* Micro-interactions: hover, toggle */
.interactive { transition: all 150ms ease-out; }

/* Layout reflows: bento→accordion transition */
.accordion-panel { transition: max-height 300ms ease-in-out; }

/* Entry animations: must finish within 500ms total */
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(20px); }
  to   { opacity: 1; transform: translateY(0); }
}
.entry { animation: fadeUp 400ms ease-out forwards; }
```

---

## SEO (30 pts — fill every tag with real content)

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>NeuralFlow | AI-Powered Data Automation Platform</title>
  <meta name="description" content="Automate your data workflows with NeuralFlow — the AI platform that processes, transforms, and delivers insights at machine speed.">
  <meta name="keywords" content="AI automation, data platform, machine learning, workflow automation, data pipeline">
  <meta property="og:title" content="NeuralFlow | AI Data Automation">
  <meta property="og:description" content="Automate your data workflows with NeuralFlow.">
  <meta property="og:type" content="website">
  <meta property="og:url" content="https://your-deployment-url.vercel.app">
  <meta property="og:image" content="https://your-deployment-url.vercel.app/og-image.png">
  <meta name="twitter:card" content="summary_large_image">
  <meta name="twitter:title" content="NeuralFlow | AI Data Automation">
  <meta name="twitter:description" content="Automate your data workflows with NeuralFlow.">
  <link rel="canonical" href="https://your-deployment-url.vercel.app">
</head>
```

- Semantic structure: `<header>` `<nav>` `<main>` `<section>` `<footer>`
- One `<h1>` only, then `<h2>`, then `<h3>`
- All SVGs: add `aria-label="description"` or `role="img" aria-label="..."`
- No placeholder text in any meta tag

---

## Asset compliance

- Every SVG from asset pack must be placed somewhere visible
- Both fonts loaded and applied (JetBrains Mono on headings, Inter on body)
- All 5 palette colors used somewhere
- Missing assets = heavy point deduction

---

## Banned (disqualification)

- Framer Motion
- Radix UI / Shadcn / HeadlessUI / Tailwind UI components
- Hardcoded price values in markup
- Private GitHub repo
- Broken deployment link

## Allowed

- React / Vite / Next.js / Vue / Vanilla
- Tailwind CSS utility classes
- Three.js (hero enhancement)
- GSAP (hero + non-Feature-2 sections only)

---

## Submission checklist

- [ ] All SVGs from asset pack used
- [ ] JetBrains Mono on headings, Inter on body
- [ ] All 5 palette colors used
- [ ] Feature 1: pricing matrix works, state isolated (no re-render)
- [ ] Feature 2: bento→accordion + context transfer on resize
- [ ] SEO meta tags all filled with real content
- [ ] Responsive: mobile / tablet / desktop
- [ ] GitHub repo public
- [ ] Live deployment works
- [ ] Demo video ≤100MB ready
- [ ] Submitted by 5:45 PM (15 min buffer)
