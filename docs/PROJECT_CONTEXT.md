# Period Tracker: Shared Project Context

**Context version:** v1 (23 October 2026)
**Team:** Isha + Partner (two people, both working remotely)
**Purpose of this file:** the single source of truth for both of us and for each of our separate Claude chats. The two chats do not share memory, so this file is how we stay consistent.

---

## 0. Instructions for Claude (paste this whole file into a new chat, then follow these rules)

You are helping one half of a two-person team build a period/cycle tracker web app. The other teammate is working with a *different* Claude chat on a different account. You have no access to that chat, so treat this file as the shared source of truth.

1. **Respect the status tags.** Every item below is tagged `CONFIRMED` (both of us agreed), `PROPOSED` (suggested but not yet confirmed by both), or `OPEN` (undecided). Do not silently change a `CONFIRMED` item. If you think one should change, say so and produce a **Change Request** (template in section 14) that the user can send to the other teammate.
2. **Protect the integration seams.** Section 11 lists everything the two halves of the project depend on (API contract, enums, date format, env vars, folder structure). Do not change a seam on your own. Any code you write for your half must follow the contract exactly.
3. **Stay in your lane unless asked.** Section 5 says who owns what. Help with the other person's area only to review or to adapt to their interface.
4. **Ask before assuming.** If the file does not cover something, ask the user, and suggest recording the answer as a new decision.
5. **Teach as you go.** The team is building this partly to learn and to explain it in interviews, so explain the *why* behind choices, and make sure the user could defend each piece of code.
6. **Safety and privacy come first.** This is a health-adjacent app handling sensitive data. Follow section 9 in every feature.
7. When the user says "update context," output the changed sections and a one-line changelog entry so they can share it with their teammate.

---

## 1. Project summary

A privacy-first period and cycle tracking **website first** (converted into an app later, because neither of us has app-development experience). Users log periods and symptoms, see a calendar with period dates and predicted dates, see their current cycle phase, read educational "tutorials," chat with a RAG-based educational chatbot (which does **not** diagnose), browse a "things we think you may need" shopping page that links out to third-party sites, and can share selected cycle info with one partner.

**Goal beyond the app:** a resume-ready portfolio project that shows real full-stack skills plus a properly evaluated ML/AI component (the RAG chatbot). Both teammates must be able to explain the *entire* project in interviews, not just their own part.

---

## 2. Decision log

| # | Decision | Status |
|---|---|---|
| 1 | Two-person team; each owns features end to end | CONFIRMED |
| 2 | Build as a website first, convert to an app later | CONFIRMED |
| 3 | Period prediction uses a static formula, **no ML** (to keep complexity down) | CONFIRMED |
| 4 | The chatbot is RAG-based, and it will **not diagnose** | CONFIRMED |
| 5 | The UI changes based on the user's mode (period / pregnancy / not yet had a period) | CONFIRMED |
| 6 | Tutorials are accessible to **everyone** (active users, people who haven't had a period yet, partners) | CONFIRMED |
| 7 | Shopping page is rule-based (no ML); we do not sell anything, we link out to third-party sites | CONFIRMED |
| 8 | Partner feature: one partner at a time, user controls what is shared (details in section 8) | CONFIRMED |
| 9 | Chats: the user chooses per chat whether it is deleted after 30 days or kept permanently | CONFIRMED |
| 10 | Database is **Postgres** (not MongoDB) | CONFIRMED |
| 11 | Signup asks username, password, email (email verification), plus a questionnaire; defaults 28-day cycle and 5-day period if lengths are not given | CONFIRMED |
| 12 | A note at signup tells users not to use their real government name as their username | CONFIRMED |
| 13 | Frontend: React (Vite), mobile-first, PWA, later wrapped with Capacitor | PROPOSED |
| 14 | Backend: Node.js + Express | PROPOSED |
| 15 | ORM: Prisma; hosted Postgres on Neon or Supabase (free tier) | PROPOSED |
| 16 | RAG chatbot is a **separate Python (FastAPI) service**, called by the Node backend over HTTP | PROPOSED |
| 17 | Tutorials are markdown files in the repo (`tutorials/`), rendered by the frontend and ingested by the RAG service | PROPOSED (leaning yes) |
| 18 | Hosting on free tiers: frontend on Vercel/Netlify, backend and RAG service on Render/Railway | PROPOSED |
| 19 | Partner symptom sharing is opt-in per symptom and **off by default** | PROPOSED |
| 20 | Work split in section 5 | PROPOSED (do the Phase 0 skills check first) |
| 21 | Whether ovulation estimates are included in v1 | OPEN (optional feature) |
| 22 | Real project deadline | OPEN |
| 23 | How the browser authenticates (httpOnly cookie vs bearer token) | OPEN (recommendation: httpOnly cookie) |

---

## 3. Features and pages (from our v1 planning notes)

### Basic functionality

- **Calendar:** period dates marked in one color, predicted period dates in a different color.
- **Prediction:** predict the next period date and cycle length. Optional: ovulation.
- **Current cycle status:** show the current cycle phase and days remaining until the next period (or the day of the period, if currently on it). Optional: ovulation.
- **Tutorials:** not tutorials in the strict sense, more like "all you need to know about the cycle, its phases, symptoms, etc." Accessible to everyone.
- **Chatbot (RAG):** does not diagnose.

### Pages

**Signup**
1. Have you had your period? If **no**, direct to login and then to the website (mode `PRE_PERIOD`).
2. If **yes**, ask: date of last period, how long the period lasted, cycle length. If cycle length or period length is not shared, assume a **28-day cycle** with a **5-day period**.
3. Are you pregnant? We have two modes, **pregnancy** and **period**. In pregnancy mode the calendar does not display future period dates until a period is logged, so it works like a pause.

**Login**
- Credentials: username, password, email (email used for verification).
- Note shown: please do not set your username as your actual government name. (We aren't going to do anything with the data, but the note is added for privacy.)

**Main interface (Home)**
- Navigation bar: **Home, Tutorials, Chatbot, Shopping, Partners**
- The calendar with dates marked.
- Below it: the **log period** and **log symptoms** forms.
- Date prediction uses a formula, not ML.

**Tutorials:** the basic things we think users need to be educated on.

**Chatbot:** RAG-based, educational only.

**Shopping (our unique feature)**
- Items we think the user should keep ready before or during their period (hygiene products, dark chocolate, period panties, pain relief patches, heat bag, etc.).
- We are not the provider. We link to third-party sites or delivery services.
- Matching is rule-based: cycle phase and logged symptoms map to product categories.

**Partners:** see section 8.

---

## 4. Tech stack and rationale

| Layer | Choice | Status |
|---|---|---|
| Frontend | React (Vite), mobile-first, PWA; Capacitor later for Android/iOS | PROPOSED |
| Backend API | Node.js + Express | PROPOSED |
| Database | **Postgres** via Prisma (Neon or Supabase free tier) | Postgres CONFIRMED; rest PROPOSED |
| RAG service | Python + FastAPI, embeddings, vector store (Chroma or Qdrant), optional reranker (e.g. bge-reranker), LLM API | PROPOSED |
| Prediction | A pure function with unit tests (Jest) | PROPOSED |
| Hosting | Vercel/Netlify (frontend), Render/Railway (backend + RAG), Neon/Supabase (DB) | PROPOSED |

**Stack rationale (so both of us give the same interview answers).** The pattern is *constraint, then choice, then the trade-off we accepted*.

- **Overall:** two students building a product that needs a web app, a chatbot, and eventually a mobile app, with no mobile-development experience. We optimized for tools we could learn quickly, share across the codebase, and deploy for free.
- **React + PWA, then Capacitor:** one codebase for web and mobile. A PWA gives an installable mobile-friendly app right away, and Capacitor wraps the same code into native builds later. Trade-off: less native performance and fewer native features, which is acceptable for forms, a calendar, and content.
- **Node + Express:** one language across frontend and backend, which means less context switching for a two-person team. Trade-off: Node is not ideal for heavy computation, so the ML work lives in a separate service.
- **Postgres:** our data is relational (users have cycles, symptom logs, and a partner link) and relies on strict rules (one log per user per day, one partner per tracker, permissions that must never leak). Postgres enforces these with foreign keys, unique constraints, and transactions, and has a real `DATE` type that avoids time-zone bugs. Trade-off: a steeper learning curve than a document database. Optional bonus: `pgvector` could store the chatbot's embeddings.
- **Separate Python service for RAG:** the ML ecosystem for embeddings, vector search, and evaluation is strongest in Python, and a separate service can be developed, evaluated, and deployed independently. Trade-off: extra deployment complexity and a network hop.
- **Formula-based prediction instead of ML:** there is not enough user data to train a useful model, and a rolling average is transparent and predictable, which matters in a health-adjacent app. We put our ML effort where it adds the most value, the chatbot.
- **Free-tier hosting:** it is a student project. Trade-off: cold starts and usage limits.

---

## 5. Roles and work split (PROPOSED: confirm after the Phase 0 skills check)

Principles: each person owns whole features front to back; both people must be able to explain the entire project; swap tasks if skills differ from what is assumed.

**Isha: content, chatbot, and information pages**
- RAG service (Python/FastAPI): corpus, ingestion, retrieval, reranking, LLM call, safety behavior
- RAG evaluation set and results table
- Tutorials: content, the tutorials page, and the markdown files in `tutorials/`
- Chatbot page and chat sessions (keep forever vs. delete after 30 days)
- Shopping page: products, rule-based matching, outbound links
- Signup questionnaire and login screens

**Partner: data, core tracking, and sharing**
- Postgres setup, Prisma schema, migrations, seed data
- Authentication: signup, login, password hashing, sessions, email verification
- Cycle and symptom APIs and the prediction function with its unit tests
- Home screen: calendar, current phase display, log period and log symptoms forms
- Modes: period and pregnancy
- Partner linking and permissioned sharing, including the partner's view

**Shared**
- Repo setup, API contract, deployment, README, final demo
- Code review of each other's pull requests
- Integration of the chatbot into the app
- Privacy and security review before launch

---

## 6. Phased plan (about 10 weeks at roughly 8-10 hours per week each; adjust once the deadline is known)

**Phase 0 (Week 1): setup and alignment.** Create the repo and folders, board, and issues; do the skills check (each rates comfort with React, Node, Python, SQL from 1 to 5; swap tasks if needed); finalize the API contract and the OPEN decisions; Partner sets up Postgres and Prisma; Isha collects corpus sources and outlines tutorials. *Done when:* both can run an empty frontend and backend locally.

**Phase 1 (Weeks 2-3): the vertical slice.** One path working end to end: sign up, verify email, log in, log a period, see it on the calendar with a predicted window. Partner: auth API, email verification, cycle API, prediction function with tests, calendar, log period form. Isha: signup questionnaire (with 28/5 defaults), login screen wired to the auth API, first tutorials. *Done when:* a new user can complete the whole path and each has walked the other through their code.

**Phase 2 (Weeks 4-5): core features.** Partner: symptom logging, phase display and days remaining, period/pregnancy modes. Isha: tutorials page, shopping page, RAG baseline (ingest, chunk, embed, store, answer with citations).

**Phase 3 (Weeks 6-7): partners and chatbot quality.** Partner: invite flow (hashed code, expiry, revoke), sharing settings, partner view that can only read what is shared. Isha: evaluation set of 50-100 questions with source passages, including safety cases; measure recall@k, MRR, and answer quality; try improvements one at a time (chunking, hybrid search, reranker, query rewriting) and record a results table.

**Phase 4 (Week 8): integration.** Connect the chatbot page to the RAG service through the backend; chat sessions with the keep/30-day choice and expiry job; educational-only disclaimer and a note that messages go to the LLM provider.

**Phase 5 (Weeks 9-10): hardening and launch.** Test main flows and permissions (try to read data you should not be able to); privacy and security review; deploy; PWA installability; README with architecture diagram, screenshots, RAG results table, and a privacy and safety design section; rehearse demo and interview answers.

**Scope cuts if time runs short (cut from the top):** PWA installability and native conversion, ovulation estimates, per-symptom sharing (keep period dates only), pregnancy mode, shopping page polish. **Protect at all costs:** auth, period and symptom logging, prediction, tutorials, and the chatbot with its evaluation.

---

## 7. Prediction logic (the Partner owns it; draft interface)

- Use the **average of the last 3-6 cycle lengths**; fall back to the user's default (28-day cycle, 5-day period) when there is no history.
- Return a **predicted window** (earliest to latest date), not a single date. Label ovulation and fertile-window estimates as *estimates*; the app is not a contraceptive or medical tool.
- In `PREGNANCY` mode, return no future predictions until a new period is logged (logging a period switches mode back to `PERIOD`).
- Predictions, phase, and days remaining are **computed on the fly and never stored**.
- Write it as a pure function with unit tests (month boundaries, time zones, late-logged cycles, irregular cycles, no history).

Draft signature (to be finalized in Phase 0):

```
getCycleStatus({ cycles, defaults, mode, today }) -> {
  avgCycleLength, avgPeriodLength,
  isOnPeriod, dayOfCycle, daysUntilNextPeriod,
  currentPhase,                      // see Phase enum in section 11
  predictedNextStart: { earliest, latest },   // "YYYY-MM-DD", null in pregnancy mode
  ovulationEstimate                  // optional, "YYYY-MM-DD" or null
}
```

---

## 8. Partner feature (inspired by Flo's partner feature)

- A user can share cycle info with **one partner at a time**.
- **Base level (always shared once linked):** period dates and ovulation dates (if ovulation is implemented). The shared base data is cycle and predicted dates only.
- **Customizable:** the user can choose to share other things, such as **which symptoms** (opt-in per symptom, **off by default**). No mood logs or notes are shared.
- The partner's interface shows **how she might be feeling** and **what the partner should do to make her feel better**. These tips come from the **cycle phase**, not from anything the user logged, plus the tutorials.
- Linking uses an **invite code** (stored hashed, with an expiry); the user can **revoke** access at any time, and revoking deletes the link.
- Permissions must be **enforced in backend queries**, not only hidden in the UI. Partner endpoints may read only the base data and the symptoms the user chose to share, and must never query symptom logs without filtering through the sharing settings.
- Partners are accounts too (`accountType = PARTNER`).

---

## 9. Chatbot (RAG) rules and chat retention

**Safety behavior (non-negotiable):**
- Educational only. It **never diagnoses**.
- If retrieval is weak, it says **"I don't know"** and suggests seeing a doctor.
- Red-flag questions (very heavy bleeding, severe pain, pregnancy complications) trigger a **fixed "please seek medical care" response**, not a generated one.
- Safety cases (diagnosis requests, red flags, out-of-scope questions) are included in the evaluation set.
- Answers cite their sources. The corpus is our tutorials plus reputable sources (WHO, NHS, ACOG patient pages).

**Chat retention (CONFIRMED):**
- The user decides **per chat** whether it is deleted after 30 days or kept permanently (for example, to show a doctor).
- Implementation: a `ChatSession` has `keep` and a nullable `expiresAt` (created + 30 days for temporary chats, null for permanent ones). A nightly job deletes expired sessions (messages cascade).
- Users can switch a chat between temporary and permanent, and delete any chat immediately. Deleting an account deletes everything, permanent chats included.
- Privacy note must say that chat messages are sent to the LLM provider.
- Optional later feature: export a symptom/cycle summary (PDF/print) for doctor visits.

**Privacy and security principles for the whole app:** collect the minimum data; hash passwords (bcrypt) and hash verification and invite tokens; validate all input; rate-limit login and chat; never commit secrets; full account deletion; clear consent for partner sharing.

---

## 10. Database schema (DRAFT: Prisma/Postgres; the Partner owns it and finalizes it in Phase 0)

Design rules: calendar dates use the Postgres `DATE` type (no time zones) and travel through the API as `"YYYY-MM-DD"` strings; predictions are never stored; symptoms use a fixed enum so partner sharing is per-symptom.

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

enum AccountType { TRACKER PARTNER }
enum Mode        { PRE_PERIOD PERIOD PREGNANCY }
enum Flow        { SPOTTING LIGHT MEDIUM HEAVY }
enum LinkStatus  { PENDING ACTIVE }
enum Phase       { BEFORE_PERIOD PERIOD FOLLICULAR OVULATION LUTEAL }
enum ChatRole    { USER ASSISTANT }
enum Symptom {
  CRAMPS HEADACHE BLOATING FATIGUE BACK_PAIN
  NAUSEA BREAST_TENDERNESS ACNE CRAVINGS INSOMNIA
}

model User {
  id                 String      @id @default(uuid())
  username           String      @unique
  email              String      @unique
  passwordHash       String
  emailVerified      Boolean     @default(false)
  verifyTokenHash    String?
  verifyTokenExpires DateTime?
  accountType        AccountType @default(TRACKER)
  mode               Mode        @default(PRE_PERIOD)
  defaultCycleLength Int         @default(28)
  defaultPeriodLength Int        @default(5)
  createdAt          DateTime    @default(now())
  updatedAt          DateTime    @updatedAt

  cycles       Cycle[]
  symptomLogs  SymptomLog[]
  chatSessions ChatSession[]
  ownedLink    PartnerLink?  @relation("LinkOwner")
  partnerLinks PartnerLink[] @relation("LinkPartner")
}

// One row per period. cycleLength is filled in when the NEXT period is logged.
model Cycle {
  id           String    @id @default(uuid())
  userId       String
  user         User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  startDate    DateTime  @db.Date
  endDate      DateTime? @db.Date
  periodLength Int?
  cycleLength  Int?
  createdAt    DateTime  @default(now())
  updatedAt    DateTime  @updatedAt

  @@unique([userId, startDate])
}

// One row per user per day.
model SymptomLog {
  id        String    @id @default(uuid())
  userId    String
  user      User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  date      DateTime  @db.Date
  symptoms  Symptom[]
  flow      Flow?
  mood      String?   @db.VarChar(30)
  notes     String?   @db.VarChar(500)
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt

  @@unique([userId, date])
}

// One partner per tracker (ownerId is unique). Revoking deletes the row.
model PartnerLink {
  id             String     @id @default(uuid())
  ownerId        String     @unique
  owner          User       @relation("LinkOwner", fields: [ownerId], references: [id], onDelete: Cascade)
  partnerId      String?
  partner        User?      @relation("LinkPartner", fields: [partnerId], references: [id], onDelete: SetNull)
  status         LinkStatus @default(PENDING)
  inviteCodeHash String?
  inviteExpires  DateTime?
  shareOvulation Boolean    @default(false)
  sharedSymptoms Symptom[]  @default([])   // opt-in per symptom, empty by default
  createdAt      DateTime   @default(now())
  updatedAt      DateTime   @updatedAt

  @@index([partnerId])
}

// Shopping page: rule-based matching on phase and/or symptoms, then link out.
model Product {
  id          String    @id @default(uuid())
  name        String
  category    String    // e.g. "hygiene", "comfort", "pain_relief", "snacks"
  description String?
  phases      Phase[]
  symptoms    Symptom[]
  retailer    String?
  url         String
  imageUrl    String?
  active      Boolean   @default(true)
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
}

model ChatSession {
  id        String        @id @default(uuid())
  userId    String
  user      User          @relation(fields: [userId], references: [id], onDelete: Cascade)
  title     String?
  keep      Boolean       @default(false)
  expiresAt DateTime?     // created + 30 days when keep = false; null when keep = true
  createdAt DateTime      @default(now())
  messages  ChatMessage[]

  @@index([userId, createdAt])
  @@index([expiresAt])
}

model ChatMessage {
  id        String      @id @default(uuid())
  sessionId String
  session   ChatSession @relation(fields: [sessionId], references: [id], onDelete: Cascade)
  role      ChatRole
  content   String
  sources   String[]    // tutorial slugs or document ids cited by the answer
  createdAt DateTime    @default(now())

  @@index([sessionId, createdAt])
}
```

Tutorials are **not** a table in this draft: they live as markdown files in `tutorials/` (decision 17, PROPOSED). If we change that decision, add a `Tutorial` model and update section 11.

---

## 11. Integration seams (both halves depend on these; change only through a Change Request)

### 11.1 Conventions

- **Dates:** `"YYYY-MM-DD"` strings everywhere in the API and frontend; Postgres `DATE` columns in the database. Timestamps (`createdAt`, etc.) are ISO 8601 UTC.
- **IDs:** UUID strings.
- **API base path:** `/api/v1`. JSON requests and responses.
- **Error format:** `{ "error": { "code": "STRING_CODE", "message": "Human readable" } }` with proper HTTP status codes.
- **Enums** (must match exactly across DB, API, and frontend): `Mode`, `Phase`, `Symptom`, `Flow`, `AccountType`, `LinkStatus` as defined in section 10. The frontend displays friendly labels, but sends and receives the enum values.
- **Auth transport:** OPEN (recommendation: httpOnly cookie with CORS credentials enabled).

### 11.2 Draft API contract (the Partner owns the backend routes; Isha's screens call them)

**Auth and account**
- `POST /auth/signup`: username, email, password, plus questionnaire answers (hasHadPeriod, lastPeriodStart, periodLength, cycleLength, isPregnant)
- `GET /auth/verify-email?token=...`, `POST /auth/resend-verification`
- `POST /auth/login`, `POST /auth/logout`
- `GET /me`, `PATCH /me` (mode, defaults), `DELETE /me` (delete account and all data)

**Cycles and status**
- `GET /cycles`, `POST /cycles`, `PATCH /cycles/:id`, `DELETE /cycles/:id`
- `GET /cycle-status` returns the output of the prediction function (section 7), including calendar-ready period and predicted ranges

**Symptoms**
- `GET /symptom-logs?from=YYYY-MM-DD&to=YYYY-MM-DD`
- `PUT /symptom-logs/:date`, `DELETE /symptom-logs/:date`

**Partners**
- `POST /partner/invite` (tracker creates a link and gets an invite code), `POST /partner/accept` (partner submits the code)
- `PATCH /partner/sharing` (shareOvulation, sharedSymptoms)
- `DELETE /partner/link` (either side may end it)
- `GET /partner/view` (partner role only; returns only base data, permitted symptoms, and phase-based tips)

**Products and tutorials**
- `GET /products?phase=...&symptoms=...`
- `GET /tutorials`, `GET /tutorials/:slug` (served from the markdown files; public, no login required)

**Chat (Isha owns the implementation; the browser never calls the RAG service directly)**
- `POST /chat/sessions`, `GET /chat/sessions`, `GET /chat/sessions/:id`
- `PATCH /chat/sessions/:id` (title, keep), `DELETE /chat/sessions/:id`
- `POST /chat/sessions/:id/messages`: stores the user message, calls the RAG service, stores and returns the answer

### 11.3 RAG service contract (internal; called only by the Node backend)

- `POST {RAG_SERVICE_URL}/answer` with `{ "question": string, "history": [{ "role": "USER"|"ASSISTANT", "content": string }] }`
- Response: `{ "status": "answered" | "no_answer" | "safety_redirect", "answer": string, "sources": [{ "id": string, "title": string }] }`
- `GET {RAG_SERVICE_URL}/health`
- The RAG service must not receive personal identifiers (no username or email), only the question and optional chat history.

### 11.4 Repo structure

```
/
  frontend/      React (Vite)
  backend/       Node + Express + Prisma
  rag-service/   Python + FastAPI
  tutorials/     markdown files (frontmatter: title, category, order, audience; plus a "Sources" section)
  docs/          API contract, schema notes, architecture, decision log, this file
  README.md
  .env.example
  .gitignore     (includes .env)
```

Keep a copy of this file at `docs/PROJECT_CONTEXT.md` and update it through pull requests so both teammates always start from the same version.

### 11.5 Environment variables (names are shared; values are never committed)

`DATABASE_URL`, `JWT_SECRET`, `FRONTEND_URL`, `EMAIL_API_KEY` (verification emails), `RAG_SERVICE_URL`, `LLM_API_KEY`, `VECTOR_DB_PATH` (or its URL), `NODE_ENV`

### 11.6 Tutorial file format

Each file in `tutorials/` has frontmatter:

```
---
title: ...
slug: ...
category: cycle_basics | symptoms | for_partners | ...
order: 1
audience: everyone
---
```

followed by markdown content and a `## Sources` section. The same files are rendered by the frontend and ingested by the RAG service (chunked by headings).

---

## 12. Prerequisites to learn (learn each right before it is needed, with a time limit)

**Both:** Git/GitHub workflow (branches, pull requests, merge conflicts, reviews); HTTP and REST basics; Postman; environment variables and secrets; basic web security (password hashing, input validation, access-control bugs).

**Isha:** React basics (components, state, hooks, forms, routing); Python and FastAPI; RAG fundamentals (embeddings, vector search, chunking, reranking, prompt design, evaluation with recall@k and MRR); processing markdown and frontmatter.

**Partner:** Node and Express (routing, middleware, error handling); SQL and Postgres basics (joins, constraints, indexes, transactions); Prisma (schema, migrations, queries); authentication (bcrypt, JWT or sessions, email verification); date handling and unit testing with Jest; enough React to build the calendar and forms.

---

## 13. How we coordinate

**Tools (keep it small):**
- **GitHub:** code, Issues, a Projects board, pull requests, and GitHub Actions for tests and linting. This is the center of the project.
- **WhatsApp or Discord:** day-to-day chat (Discord if we want voice rooms for pairing).
- **Google Meet or Discord voice:** weekly sync and pair sessions.
- **Shared notes (Notion or Google Docs):** decision log and meeting notes. Technical docs live in `docs/` in the repo.
- **Helpful extras:** VS Code Live Share, Figma or Excalidraw (wireframes), dbdiagram.io or the Prisma schema (database diagram), a Postman collection or OpenAPI file (API contract), Bitwarden (sharing credentials; never share secrets in chat), Google Calendar (recurring syncs).

**Rhythm:** weekly planning and demo (30-45 minutes); short async check-ins two or three times a week (done, doing next, blocked); the **45-minute rule** (stuck for 45 minutes means ask or pair); a code walkthrough at the end of every phase so each of us can explain the other's part.

**Code workflow:** `main` always works; branches are `feature/<short-name>` and merge through pull requests; keep pull requests small (ideally under 300 lines) and review within 24-48 hours; pull from `main` at the start of every session; work in your own area to avoid editing the same file at once; announce any change to the API contract, schema, enums, or env vars **before** making it; write clear commit messages (what and why).

**Definition of done:** it works, has tests where logic is involved, has been reviewed and merged to `main`, and affected docs are updated.

**Decisions:** record significant decisions (what we chose, what we considered, why) in the decision log. It prevents repeat arguments and is the raw material for interview answers.

---

## 14. Templates for keeping both chats in sync

**Change Request (send to the teammate before changing a seam or a CONFIRMED item)**
```
CHANGE REQUEST
Section/decision affected: <e.g. 11.2 Cycles API, decision #17>
Proposed change: <what exactly>
Reason: <why>
Impact on the other half: <what the other person must change>
Needs reply by: <date>
```

**Status update (post in chat 2-3 times a week)**
```
Done: <...>
Doing next: <...>
Blocked / need from you: <...>
Changed any seam? <no / yes: link to the change request>
```

**Context changelog (add a line whenever this file changes)**
- v1 (23 Oct 2026): initial shared context created from the planning chat.

---

## 15. Open questions to settle in Phase 0

1. The real project deadline (to fit the phases to it).
2. Skills check results, and whether the split in section 5 still fits.
3. Tutorials as repo files (recommended) versus a database table.
4. Auth transport (httpOnly cookie recommended).
5. Whether ovulation estimates are in v1.
6. Whether partner symptom sharing stays in v1 (opt-in per symptom, off by default) or moves to a later version.
7. The final symptom list and the final product categories.
