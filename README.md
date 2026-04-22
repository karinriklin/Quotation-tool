# BauAngebot Berlin 🏗️

> An AI-powered construction quotation tool for small building companies operating in Berlin, Germany.  
> Built with Claude AI · Vanilla JS · No frameworks · No backend required

---

## What it does

BauAngebot Berlin is a single-page web application that turns a plain-text job description into a professional construction quotation in under two minutes.

The app guides the user through five steps:

1. **Job input** — describe the project in plain language, set the crew size, working days, and daily rate
2. **AI analysis** — Claude AI reads the description, classifies the trade, breaks it into work positions, and estimates realistic material quantities and costs for the Berlin market
3. **Shop price comparison** — the AI searches and compares material prices across four major Berlin building supply stores (Bauhaus, OBI, Hornbach, Globus Baumarkt) and recommends the cheapest full basket
4. **Review & edit** — all line items, material costs, and crew costs are editable before finalising
5. **Document export** — generates two print-ready PDFs: a client-facing *Angebot* (quotation) and an internal *Einkaufsliste* (shopping checklist)

All previous quotations are saved locally in the browser and accessible from the sidebar.

---

## Screenshots

| Step 1 — Job input | Step 3 — Shop comparison | Step 4 — Cost review |
|---|---|---|
| Enter client, address, job description, crew details | AI compares Bauhaus, OBI, Hornbach & Globus Baumarkt | Editable line items, live totals with MwSt. |

---

## Pricing calculation

The app follows the correct German construction invoicing structure:

```
Materials (netto)
+ 15% Aufschlag (overhead/markup)
+ Work cost (employees × days × €200/day)
= Netto gesamt
+ 19% MwSt. (Umsatzsteuer)
= Brutto gesamt (what the client pays)
```

The markup covers overhead costs the company incurs but doesn't bill separately — vehicle maintenance, tool replacement, insurance, admin, and a small profit buffer. The 19% VAT is always applied last, on top of the full amount.

---

## Tech stack

| Layer | Technology | Why |
|---|---|---|
| UI | Vanilla HTML/CSS/JS | No build step, no dependencies, runs anywhere |
| AI | Anthropic Claude API (`claude-sonnet-4-20250514`) | Job analysis, cost estimation, shop price research |
| Storage | Browser `localStorage` | Zero-backend persistence, works offline after first load |
| Fonts | Google Fonts (DM Serif Display + DM Sans) | Professional, characterful — not the usual developer defaults |
| Export | Browser Print API → PDF | No server-side PDF generation needed for Stage 1 |

---

## How to run it

### Option A — Open directly (simplest)
1. Download `src/index.html`
2. Open it in Chrome or Firefox
3. The app runs fully in the browser — no installation needed

> **Note:** The Claude API is called client-side using an injected key (Stage 1 / demo mode). For production deployment, see Stage 2 below.

### Option B — Serve locally
```bash
# Any simple static server works
npx serve src/
# or
python3 -m http.server 8080 --directory src/
```
Then open `http://localhost:8080`

---

## Project goals

This project was built to explore three things:

### 1. AI as a domain expert
Can a general-purpose AI model produce useful, market-calibrated outputs for a specialised trade? The answer turned out to be yes — when given the right context (Berlin market, German legal terms, correct tax structure), Claude produces plausible material lists, realistic labor rates by trade, and identifies which building supply shop has the cheapest basket.

### 2. Prompt engineering as product design
The "skill" behind this app — the structured instructions that tell Claude how to analyse a job, format its output as JSON, and pick the winning shop — was designed iteratively using a skill-based prompt engineering workflow. The key insight: the AI's output quality is directly proportional to how well the prompt specifies the *structure* of the expected response, not just the *content*.

### 3. AI-native tools vs. traditional software
Traditional estimating software for construction is expensive, complex, and requires manual price databases. This tool replaces all of that with a single AI call and a web search — at the cost of some precision, but with a massive gain in speed and accessibility. The tradeoff is deliberate.

---

## Outcomes

- **Working product** built in a single session without writing a single line of backend code
- **Accurate German pricing** — labor rates, MwSt., VOB/B references, and §35a EStG tax tips are all calibrated to Berlin 2026
- **Real document output** — the generated Angebot and Einkaufsliste follow DIN 5008 conventions and are ready to hand to a client or take to a Baumarkt
- **Persistent storage** — all quotations survive browser restarts via localStorage
- **Modular architecture** — the codebase is structured so the AI layer, storage layer, and UI layer can each be swapped independently for Stage 2

---

## Learnings

### What worked well
- **Structured JSON output from AI** — asking Claude to return strict JSON (with a schema in the system prompt) made the AI output trivially parsable and reliable
- **Fallback data** — defining a sensible fallback when the API call fails means the app degrades gracefully rather than breaking
- **Separation of concerns without a framework** — keeping state in a plain JS object (`current`) and re-rendering the whole view on each state change (like a minimal React) made the app surprisingly maintainable
- **Domain specificity beats generality** — the more Berlin-specific context I added (exact shop names, branch addresses, German legal references, Berlin labor rates), the more useful the AI outputs became

### What I'd do differently at scale
- **Move the API key server-side** — calling the Anthropic API from the browser exposes the key; a lightweight proxy (Node/Express, ~30 lines) fixes this entirely
- **Replace localStorage with a real database** — Supabase or PocketBase would give multi-device sync and proper data integrity with minimal added complexity
- **Add real .docx generation** — the `docx` npm package can produce proper Word documents with tables, headers, footers, and German number formatting; the print-to-PDF approach works but loses formatting fidelity
- **Add input validation** — the current UI trusts the user to enter sensible numbers; a production version would validate ranges and flag unlikely inputs (e.g. 50 employees, 1 day)
- **Test with real users** — the pricing assumptions are calibrated from web research; field-testing with actual Berlin contractors would reveal which numbers are consistently off

---

## Stage 2 roadmap

The app is designed so that Stage 1 → Stage 2 migration touches only the infrastructure, not the product logic.

```
Stage 1 (current)          Stage 2 (planned)
─────────────────────      ────────────────────────────
Single HTML file           React frontend (Vite)
localStorage               Supabase (PostgreSQL)
Client-side API call       Node/Express API proxy
Print-to-PDF               Real .docx via docx npm
No auth                    Optional: single-password access
Hosted on GitHub Pages     Hosted on Vercel (free tier)
```

**Estimated Stage 2 effort:** ~2–3 days of focused work.

---

## Folder structure

```
bauangebot/
├── src/
│   └── index.html          # The full application (self-contained)
├── docs/
│   └── stage2-plan.md      # Technical plan for the production version
├── public/
│   └── screenshot.png      # App screenshot for this README
├── .gitignore
├── LICENSE
└── README.md               # You are here
```

---

## License

MIT — use it, fork it, build on it.

---

## Built with

- [Anthropic Claude](https://anthropic.com) — AI backbone
- [DM Serif Display & DM Sans](https://fonts.google.com) — typography
- Vanilla JS, HTML5, CSS3 — everything else

---

*Built as a portfolio project to demonstrate AI-native product development, prompt engineering, and rapid prototyping without traditional frameworks.*
