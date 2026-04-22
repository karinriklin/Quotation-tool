# Stage 2 — Production Architecture Plan

This document outlines the technical migration path from the Stage 1 single-file demo to a fully independent, production-grade web application.

---

## Why Stage 2?

Stage 1 proves the concept and delivers real value — but it has two hard constraints:

1. **The API key is exposed** in the browser. Anyone who opens DevTools can see it and use it.
2. **Data lives in `localStorage`** — tied to one browser on one device. No backup, no sync.

Stage 2 fixes both with minimal added complexity.

---

## Architecture overview

```
┌─────────────────┐         ┌──────────────────┐        ┌─────────────────┐
│   React Frontend │  HTTPS  │  Node/Express API │  SDK   │  Anthropic API  │
│   (Vercel)       │ ──────► │  (Vercel Function)│ ─────► │  Claude Sonnet  │
└─────────────────┘         └──────────────────┘        └─────────────────┘
         │                           │
         │                    ┌──────────────┐
         └────────────────────│  Supabase DB │
                    REST      │  (PostgreSQL) │
                              └──────────────┘
```

All three services have generous free tiers. Total monthly cost at low volume: **€0**.

---

## Step-by-step migration

### Step 1 — Set up the repo structure

```bash
bauangebot/
├── frontend/           # Vite + React app
│   ├── src/
│   │   ├── App.jsx
│   │   ├── components/
│   │   │   ├── Sidebar.jsx
│   │   │   ├── Step1Input.jsx
│   │   │   ├── Step2Analysis.jsx
│   │   │   ├── Step3Shops.jsx
│   │   │   ├── Step4Review.jsx
│   │   │   └── Step5Documents.jsx
│   │   ├── lib/
│   │   │   ├── api.js        # calls to your backend
│   │   │   ├── storage.js    # Supabase client
│   │   │   └── format.js     # German number formatting
│   │   └── main.jsx
│   └── package.json
├── api/                # Vercel serverless functions
│   └── quote.js        # proxies Claude API calls
└── README.md
```

### Step 2 — Build the API proxy (30 lines)

Create `api/quote.js`:

```javascript
import Anthropic from '@anthropic-ai/sdk';

export default async function handler(req, res) {
  if (req.method !== 'POST') return res.status(405).end();

  const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

  try {
    const message = await client.messages.create({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 1000,
      system: req.body.system,
      messages: req.body.messages,
    });
    res.status(200).json(message);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
}
```

Set `ANTHROPIC_API_KEY` as an environment variable in Vercel — never in code.

### Step 3 — Set up Supabase

1. Create a free project at [supabase.com](https://supabase.com)
2. Create the `quotes` table:

```sql
create table quotes (
  id          uuid primary key default gen_random_uuid(),
  created_at  timestamptz default now(),
  client      text,
  addr        text,
  desc        text,
  emp         int,
  days        int,
  rate        numeric,
  items       jsonb,
  shops       jsonb,
  winner      text,
  savings     numeric,
  total_brutto numeric,
  assumptions text,
  exclusions  text
);
```

3. Replace the `localStorage` calls in `storage.js`:

```javascript
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_ANON_KEY
);

export async function saveQuote(quote) {
  const { data, error } = await supabase
    .from('quotes')
    .upsert(quote)
    .select();
  return data;
}

export async function loadQuotes() {
  const { data } = await supabase
    .from('quotes')
    .select('*')
    .order('created_at', { ascending: false });
  return data || [];
}
```

### Step 4 — Add real .docx generation

Install the `docx` package:

```bash
npm install docx file-saver
```

Replace the `window.print()` calls with proper Word document generation:

```javascript
import { Document, Packer, Paragraph, Table, TableRow, TableCell,
         TextRun, HeadingLevel, AlignmentType, WidthType } from 'docx';
import { saveAs } from 'file-saver';

export async function generateAngebot(quote) {
  const doc = new Document({
    sections: [{
      properties: {
        page: { size: { width: 11906, height: 16838 } } // A4
      },
      children: [
        new Paragraph({
          text: 'ANGEBOT / KOSTENVORANSCHLAG',
          heading: HeadingLevel.HEADING_1,
        }),
        // ... build full document structure
      ]
    }]
  });

  const buffer = await Packer.toBuffer(doc);
  saveAs(new Blob([buffer]), `angebot_${quote.client}_${Date.now()}.docx`);
}
```

### Step 5 — Deploy to Vercel

```bash
npm install -g vercel
cd bauangebot
vercel --prod
```

Add environment variables in the Vercel dashboard:
- `ANTHROPIC_API_KEY` — your Anthropic key
- `VITE_SUPABASE_URL` — from Supabase project settings
- `VITE_SUPABASE_ANON_KEY` — from Supabase project settings

---

## Effort estimate

| Task | Estimated time |
|---|---|
| Set up Vite + React project | 1 hour |
| Port HTML/JS to React components | 3–4 hours |
| Build API proxy + deploy to Vercel | 1 hour |
| Set up Supabase + swap storage layer | 1–2 hours |
| Implement real .docx generation | 3–4 hours |
| Testing + polish | 2 hours |
| **Total** | **~12 hours** |

---

## What does NOT change

- The AI prompt logic (same system prompt, same JSON schema)
- The pricing calculation (materials + markup + work cost + MwSt.)
- The 5-step wizard UX
- The shop comparison logic
- The German number formatting and legal references

Stage 2 is purely an infrastructure upgrade — the product stays identical.

---

## Optional future features

- **PDF generation server-side** using Puppeteer (better formatting than browser print)
- **Email delivery** — send the Angebot directly to the client via Resend or SendGrid
- **Price database** — cache Baumarkt prices in Supabase so repeated searches are faster
- **Multi-language** — the app logic is in German but the UI could offer English mode
- **Mobile app** — the HTML is already responsive; a PWA wrapper (or Capacitor) would make it installable on iOS/Android
