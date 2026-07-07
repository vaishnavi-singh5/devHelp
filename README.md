# DevHelp — AI Software Engineer Workspace

A light, IDE-inspired workspace for engineers, built with **Next.js 16 (App
Router) + TypeScript + Tailwind CSS v4**, powered by the **Gemini API**, with
a real **SQLite database** for Notes & Knowledge Base.


<img width="1918" height="1078" alt="Screenshot 2026-07-08 003807" src="https://github.com/user-attachments/assets/d0a53f9a-438a-4648-9c7f-9bc848706d55" />


## What's working right now

| Module | Route | Backed by |
|---|---|---|
| **AI Chat** | `/chat` | Gemini (`/api/chat`) |
| **PDF Assistant** | `/modules/pdf-assistant` | Gemini + PDF text extraction (`/api/pdf`) |
| **Resume Optimizer** | `/modules/resume-optimizer` | Gemini (`/api/resume-optimizer`) |
| **ATS Checker** | `/modules/ats-checker` | Gemini (`/api/ats-checker`) |
| **Job Match Analyzer** | `/modules/job-match` | Gemini (`/api/job-match`) |
| **Notes & Knowledge Base** | `/modules/notes` | SQLite database (`/api/notes`) + Gemini for "Ask your notes" |

Plus: `/login`, `/dashboard`, `/settings` (profile, appearance, and a
"Test Gemini connection" button). **Interview Question Generator** is the one
remaining "coming soon" placeholder — everything else listed above is fully
functional end to end.

Code Assistant, GitHub Repository Analyzer, and LeetCode Coach have been
removed from this build per your latest request.

## 1. Install

Requires **Node.js 18.18+** (Node 20 LTS recommended) and npm.

```bash
unzip devhelp.zip
cd devhelp
npm install
```

## 2. Add your Gemini API key

1. Get a free key at https://aistudio.google.com/apikey
2. Copy the example env file and paste your key in:
   ```bash
   cp .env.local.example .env.local
   ```
   ```env
   GEMINI_API_KEY=your-key-here
   GEMINI_MODEL=gemini-flash-latest
   ```
The key is only ever read on the server (inside `src/lib/gemini.ts` and the
`/api/*` routes) — it's never sent to or stored in the browser.

## 3. Run it

```bash
npm run dev
```

Open http://localhost:3000 → redirects to `/login`. Any email/password (or
"continue as guest") gets you into the dashboard — there's no real auth yet,
by design, since you didn't ask to build that out.

Once running, go to **Settings → API keys → Test Gemini connection** to
confirm your key works.

## The database (Notes & Knowledge Base)

Notes are stored in a real **SQLite** database at `data/devhelp.db`, created
automatically the first time you save a note. It survives server restarts —
open `/modules/notes`, add a note, restart `npm run dev`, and it's still there.

**Why SQLite via `sql.js` instead of a native driver:** this keeps the
project running with zero setup — no Postgres server, no native compilation
step, nothing to install beyond `npm install`. The database logic is fully
isolated in **`src/lib/db.ts`** (`listNotes`, `createNote`, `updateNote`,
`deleteNote`) — the API routes and UI never touch SQL directly, so swapping
the storage engine later doesn't touch anything else.

### Moving to PostgreSQL later

Your original roadmap called for FastAPI + PostgreSQL + SQLAlchemy as a
separate backend (Phase 2). If you want a real hosted Postgres database
instead of the local SQLite file:

1. Provision a Postgres instance (Supabase, Neon, Railway, RDS, or your own).
2. Install a driver: `npm install pg` (or use an ORM like Prisma/Drizzle).
3. Rewrite the functions inside `src/lib/db.ts` to run SQL against Postgres
   instead of `sql.js` — the function signatures (`listNotes`, `createNote`,
   etc.) can stay identical, so no other file needs to change.
4. Add `DATABASE_URL` to `.env.local`.

I'm happy to build that swap out for you — just say the word and tell me
which Postgres provider you'd like to use.

## Project structure

```
src/
  app/
    login/page.tsx
    (workspace)/layout.tsx           # sidebar shell for all logged-in routes
    (workspace)/dashboard/page.tsx
    (workspace)/chat/page.tsx
    (workspace)/settings/page.tsx
    (workspace)/modules/
      pdf-assistant/page.tsx
      resume-optimizer/page.tsx
      ats-checker/page.tsx
      job-match/page.tsx
      notes/page.tsx
      [slug]/page.tsx                # "coming soon" placeholder (interview-questions)
    api/
      chat/route.ts
      pdf/route.ts                   # PDF upload + grounded Q&A
      extract-pdf/route.ts           # shared PDF-to-text used by resume/ATS/job-match uploaders
      resume-optimizer/route.ts
      ats-checker/route.ts
      job-match/route.ts
      notes/route.ts                 # GET (list) / POST (create)
      notes/[id]/route.ts            # PUT (update) / DELETE
      notes/ask/route.ts             # Gemini Q&A grounded in your saved notes
  components/                         # Sidebar, Topbar, PdfUploadButton, ScoreRing, TagList, ...
  lib/
    modules.ts                       # single source of truth for sidebar/dashboard modules
    gemini.ts                        # Gemini API client (text + strict-JSON modes)
    pdf.ts                           # PDF text extraction (pdf-parse)
    db.ts                            # SQLite data-access layer for notes
data/
  devhelp.db                         # created automatically, gitignored
```

## Design

Light "workbench" theme: white/paper surfaces, amber accent for primary
actions, teal for AI/secondary elements, monospace labels for that
editor/file-tree feel (see `src/app/globals.css` for all color tokens).

## Commands

```bash
npm run dev     # start the dev server
npm run build   # production build (also type-checks)
npm run start   # run the production build
npm run lint    # eslint
```
