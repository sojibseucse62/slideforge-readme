<div align="center">

# SlideForge MCQ

### *A Zero-Server MCQ Bank, Automated Slide Generation & Raw File Creation*

**Turn a Google Sheet of questions into a classroom-ready, math-rendered Google Slides deck or convert JSON data to understandable table format — in one click.**

![Made with Google Apps Script](https://img.shields.io/badge/Backend-Google%20Apps%20Script-4285F4?style=for-the-badge&logo=google&logoColor=white)
![Vanilla JS](https://img.shields.io/badge/Frontend-Vanilla%20JS-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![MathJax](https://img.shields.io/badge/Math-MathJax%20%2B%20LaTeX-1a936f?style=for-the-badge)
![Bangla Ready](https://img.shields.io/badge/Language-Bangla%20%2B%20English-F20F34?style=for-the-badge)
![Zero Server](https://img.shields.io/badge/Infrastructure-Serverless%20%2F%20%240%20Hosting-8A2BE2?style=for-the-badge)

<br/>

<!-- 📸 HERO SCREENSHOT — replace with your dashboard screenshot -->
<img width="1439" height="780" alt="Dashboard" src="https://github.com/user-attachments/assets/f0395bdd-e7dd-44a1-a741-04c28a5a32a0" />
*The dashboard in action — live stats, class drill-down, and one-click deck generation.*

</div>

---

## 📖 Table of Contents

- [Project Overview](#-project-overview)
- [Core Features](#-core-features)
- [How It Works (Architecture)](#-how-it-works-architecture)
- [The Rendering Engine — Deep Dive](#-the-rendering-engine--deep-dive)
- [Screenshots](#-screenshots)
- [Tech Stack](#-tech-stack)
- [Getting Started](#-getting-started)
- [Data Model](#-data-model)
- [Security & Access Control](#-security--access-control)
- [How It Makes Life Easier](#-how-it-makes-life-easier)
- [Roadmap](#-roadmap)
- [License](#-license)

---

## 🎯 Project Overview

**SlideForge MCQ** (internally codenamed *Super Prep*) is a full-stack, **completely serverless** teaching toolkit built for Class 6–10 educators. It transforms an ordinary **Google Sheet** into:

1. 🗄️ A **searchable, access-controlled MCQ question bank** with thousands of multiple-choice questions across classes, subjects, and chapters
2. 🎞️ An **automated Google Slides deck generator** that builds question + answer slides from a reusable template — complete with rendered LaTeX equations, Bangla typography, and highlighted correct answers
3. 🔐 A **multi-teacher portal** where every teacher logs in with their own credentials and sees *only* the classes and subjects they're authorized to teach

The entire system runs on **Google's free infrastructure** — no servers, no databases, no monthly bills. The backend is a single Google Apps Script web app; the frontend is three static files you can host anywhere (GitHub Pages, Netlify, or even a school computer).

---

## ✨ Core Features

### 🖥️ The Teacher Dashboard (Frontend)

| Feature | What It Does |
|---|---|
| 🔐 **Secure Login Portal** | E-mail + password authentication against the `User_Info` sheet, with signed session tokens stored locally — sessions survive page refreshes for 7 days |
| 📊 **Animated Hero Stats** | Live counters showing total MCQs and per-class breakdowns, scoped to *your* access — with smooth count-up animations |
| 🧭 **Guided Drill-Down Flow** | Breadcrumb-driven navigation: **Class → Subject → Chapter → Questions**, each step filtered by your permissions |
| 🔍 **Instant Local Search** | Filter loaded questions by keyword in real time — no extra server round-trips |
| 🎲 **Random Sampling Mode** | "Give me 25 random questions from this chapter" — perfect for surprise quizzes and balanced practice sets |
| 🧮 **Live Math Preview** | MathJax renders every `$...$`, `$$...$$`, `\(...\)` LaTeX expression right in the browser, exactly as students will see it |
| 🖼️ **Smart Drive Images** | Question diagrams stored in Google Drive lazy-load with skeleton placeholders and open in a full-screen lightbox |
| ✅ **Selection Dock** | Tick questions across pages; a floating dock tracks your selection count with select-all / clear shortcuts |
| 🎞️ **One-Click PPT Modal** | Choose your deck, pick a Bangla font (Kalpurush / Li Ador Noirrit / Anek Bangla), and generate — with progress feedback |
| 🧾 **JSON → Table Builder** | Drop in raw JSON question exports and get a clean, copy-paste-ready table or CSV — a built-in data-entry accelerator |
| 📤 **JSON Export** | Export any selection as structured JSON for quizzes, apps, or archives |
| 🌗 **Dark / Light Themes** | A polished dual-theme UI with a flash-free theme boot and persistent preference |
| 💀 **Skeleton Loading States** | Every async view renders skeleton cards first — the app *feels* fast even on slow school Wi-Fi |

### ⚙️ The Engine Room (Backend — Google Apps Script)

| Feature | What It Does |
|---|---|
| 🔑 **HMAC-Signed Tokens** | Stateless session tokens signed with HMAC-SHA256 — tamper-proof, with built-in 7-day expiry, no token database needed |
| 🛡️ **Row-Level Access Control** | Every API call re-validates the teacher's class/subject permissions — even slide generation refuses unauthorized questions |
| 🗂️ **Sheet-as-Database API** | A clean GET/POST JSON API (`login`, `getTotals`, `getClasses`, `getSubjects`, `getChapters`, `getQuestions`, `generatePPT`) over a cached, header-indexed sheet read |
| 🎨 **Template-Driven Slides** | Slide #1 of your Google Slides file *is* the design. The engine duplicates it for every question and answer, swapping `{{placeholders}}` — change the template, change every future deck |
| 🟢 **Answer-Slide Automation** | Each MCQ produces a **question slide** and an **answer slide** where the correct option's shape is auto-filled green |
| 🧮 **Hybrid Bangla + LaTeX Renderer (v9)** | The crown jewel — see [the deep dive below](#-the-rendering-engine--deep-dive) |
| 🔄 **Word-Math Migration** | Automatically converts MS Word *linear-format* equations (`■8(...)`, `█(...)` matrix syntax) into proper LaTeX `bmatrix` — paste from old Word docs and it just works |
| 🌐 **Unicode Math Normalizer** | Translates 200+ Unicode math symbols (², ₓ, ½, √, ∑, ∫, →, π, ℝ, 𝐀…) into render-safe LaTeX |
| ♻️ **Idempotent Regeneration** | Old generated slides are wiped before every run — regenerate as many times as you like, the template is never touched |
| 🚦 **Quota-Friendly Pacing** | Micro-sleeps between batches keep large decks (up to 2,000 MCQs) inside Google's API limits |

---

## 🏗️ How It Works (Architecture)

```
┌────────────────────────────────────────────────────────────────────────┐
│                            TEACHER'S BROWSER                           │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │   index.html + style.css + script.js  (static — host anywhere)   │  │
│  │                                                                  │  │
│  │   Login → Drill-down UI → MathJax preview → Selection dock       │  │
│  │   localStorage: signed auth token + theme preference             │  │
│  └───────────────┬──────────────────────────────────────────────────┘  │
└──────────────────┼─────────────────────────────────────────────────────┘
                   │  HTTPS · JSON (GET for reads, POST for generation)
                   ▼
┌────────────────────────────────────────────────────────────────────────┐
│                  GOOGLE APPS SCRIPT WEB APP  (the backend)             │
│                                                                        │
│   doGet ─► login / me / getTotals / getClasses /                       │
│            getSubjects / getChapters / getQuestions                    │
│   doPost ─► generatePPT                                                │
│                                                                        │
│   🔐 verifyAuthToken (HMAC-SHA256 + TTL)                               │
│   🛡️ userAllowsPair(class, subject) on every data path                 │
│   🧮 Rendering Engine v9 (segmenter → normalizer → CodeCogs)           │
└──────┬──────────────────────┬──────────────────────┬───────────────────┘
       │                      │                      │
       ▼                      ▼                      ▼
┌──────────────┐     ┌─────────────────┐     ┌──────────────────────┐
│ Google Sheet │     │  Google Slides  │     │  CodeCogs LaTeX API  │
│  Dashboard   │     │  Template deck  │     │  LaTeX → 150-DPI     │
│  User_Info   │     │  (slide #1 =    │     │  transparent PNG     │
│ (database)   │     │   the design)   │     │  (equation images)   │
└──────────────┘     └─────────────────┘     └──────────────────────┘
```

### The Generation Pipeline, Step by Step

1. **Authenticate** — The teacher logs in; the backend issues a base64 token: `email | timestamp | HMAC-signature`. Every subsequent request is verified statelessly.
2. **Browse & Select** — The frontend walks the Class → Subject → Chapter tree (each list pre-filtered server-side by the teacher's permissions) and loads questions with full LaTeX previews.
3. **Submit Selection** — The browser POSTs lightweight *question keys* (row number + class/subject/chapter/serial fingerprint), not full question data — the server re-reads the **source of truth** from the Sheet and re-checks authorization per question.
4. **Forge the Deck** — For each MCQ the engine duplicates the template slide twice (question + answer), then routes every placeholder through the hybrid renderer.
5. **Deliver** — The teacher receives a direct **edit link** and a **`.pptx` export link**. Open, present, done.

---

## 🧮 The Rendering Engine — Deep Dive

> *The hardest engineering problem in this project, solved elegantly.*

Google Slides has **no native equation support**. Most tools punt — they either dump raw LaTeX as text or rasterize the *entire* slide into an image (killing editability). SlideForge does neither.

**The v9 hybrid strategy:**

```
Input cell:  "একটি বস্তুর বেগ $v = u + at$ হলে ত্বরণ কত?"
                 │
                 ▼
   ┌─────────────────────────────┐
   │ 1. SEGMENTER                │   Splits content into typed segments using a
   │    bangla │ math │ bangla   │   unified delimiter grammar: [[$$..$$]],
   └─────────────────────────────┘   [$$..$$], $$..$$, $..$, \[..\], \(..\),
                 │                   bare [\latex..], plus heuristic detection
                 ▼                   for undelimited math.
   ┌─────────────────────────────┐
   │ 2. NORMALIZER               │   Word linear-format matrices → bmatrix,
   │    ■8(a&b@c&d) → \begin..   │   200+ Unicode symbols → LaTeX commands,
   │    x² → x^{2}, ½ → \frac..  │   auto-shorthand (a/b → \frac{a}{b}).
   └─────────────────────────────┘
                 │
                 ▼
   ┌─────────────────────────────┐
   │ 3. RENDERER                 │   • Bangla/plain text → stays as EDITABLE
   │    text  → editable runs    │     text in the shape (correct font applied)
   │    math  → CodeCogs PNG     │   • Math → 150-DPI transparent PNG, scaled
   └─────────────────────────────┘     so image height = font line height
                 │
                 ▼
   ┌─────────────────────────────┐
   │ 4. FLOW SIMULATOR           │   A character-width estimator (tuned for
   │    measures text flow,      │   Bangla vs Latin glyph widths) reserves
   │    reserves space-runs,     │   whitespace in the text run, computes the
   │    overlays images inline   │   wrap position, and overlays each equation
   └─────────────────────────────┘   image exactly where it belongs in the line.
```

**Resilience built in:** complex expressions (fractions, integrals, matrices) are retried with `\displaystyle` for full-size rendering; PNG responses are validated by magic bytes; and if CodeCogs ever fails, the raw text is preserved in brackets — **content is never silently lost**.

The result: a generated slide where a teacher can still click and fix a Bangla typo, while the physics equation beside it looks like it came out of a textbook.

---

## 📸 Screenshots

| Login Portal | Dashboard & Stats |
|:---:|:---:|
| <img width="420" height="493" alt="Login portal" src="https://github.com/user-attachments/assets/f3b1b929-53a8-44ce-94e7-15c64f677b9c" /> | <img width="420" height="228" alt="Dashboard" src="https://github.com/user-attachments/assets/1cb92088-5720-4e0c-8837-a92878c5559e" /> |

| Question Browser (MathJax) | PPT Generation Modal |
|:---:|:---:|
| <img width="420" height="278" alt="MCQ Preview with image" src="https://github.com/user-attachments/assets/7022afab-2a3b-4ac0-8528-82037ee49587" /> | <img width="420" height="513" alt="Slide Generation Page" src="https://github.com/user-attachments/assets/613526ff-a2b9-4a42-941c-54c0f84dee37" /> |

| JSON Data Preview | JSON to Understandable Table Format |
|:---:|:---:|
| <img width="420" height="546" alt="JSON Preview" src="https://github.com/user-attachments/assets/5a19a4b7-a5fc-4d7c-8bca-a54d9ee070be" /> | <img width="420" height="546" alt="Table View" src="https://github.com/user-attachments/assets/e093831c-08a4-4622-b6e3-3ed4b9c3c8b9" /> |

| Generated Slide |
|:---:|
| <img width="867" height="534" alt="Slide" src="https://github.com/user-attachments/assets/d0a22002-8b6c-4043-b24b-58ac960e8ef0" /> |


---

## 🧰 Tech Stack

| Layer | Technology | Why |
|---|---|---|
| **Frontend** | Vanilla HTML5 / CSS3 / ES6+ JavaScript | Zero build step, zero dependencies, loads instantly on school hardware |
| **Math (browser)** | MathJax 3 (`tex-chtml`) | Faithful in-browser LaTeX preview before generation |
| **Fonr** | Anek Bangla, Kalpurush, Li Ador Noirrit | First-class Bangla rendering everywhere |
| **Backend** | Google Apps Script (V8 runtime) | Free, serverless, lives next to the data |
| **Database** | Google Sheets | Teachers already know it — editing the question bank *is* the admin panel |
| **Slides** | Google Slides API (`SlidesApp`) | Template duplication + placeholder substitution |
| **Math (slides)** | CodeCogs LaTeX rendering service | Crisp 150-DPI transparent equation PNGs |
| **Auth** | HMAC-SHA256 signed tokens | Stateless, tamper-proof sessions without a session store |

---

## 🚀 Getting Started

### Prerequisites
- A Google account
- A Google Sheet with two tabs: `Dashboard` (question bank) and `User_Info` (teacher accounts)
- A Google Slides file whose **first slide is your design template**, containing the placeholders:
  `{{Confidential}}`

### 1️⃣ Deploy the Backend

```text
1. Open your Google Sheet → Extensions → Apps Script
2. Paste the backend script (Code.gs)
3. ⚠️ Replace AUTH_SECRET with your own long random string
4. Deploy → New deployment → Web app
   • Execute as: Me
   • Who has access: Anyone
5. Copy the /exec deployment URL
```

### 2️⃣ Wire Up the Frontend

```js
const CFG = { apiUrl: 'PASTE_YOUR_DEPLOYMENT_URL_HERE' };
```

### 3️⃣ Host the Frontend

Drop `index.html`, `style.css`, and `script.js` on **GitHub Pages**, **Netlify**, **Vercel** — or just open `index.html` locally. No build step. No `npm install`. It just runs.

### 4️⃣ Add Teachers

Fill the `User_Info` sheet:

| Name | E-mail | Password | Class | Assigned Subject | presentationId | Account_Status |
|---|---|---|---|---|---|---|
| User_1 | user@school.bd | ●●●●●● | `Class 9, Class 10` | `Physics, Math` | *(Slides URL or ID)* | `Active` |
| Admin | admin@school.bd | ●●●●●● | `*` | `*` | *(Slides URL or ID)* | `Active` |

---

## 🗃️ Data Model

**`Dashboard` sheet (the question bank)** — one row per MCQ:

```
Class │ Subject │ Segment │ Chapter │ Topic │ Reference │ SI │ type │ title │ image
options_1_answer │ options_1_is_correct │ ... │ options_4_answer │ options_4_is_correct
```

- `title` and option cells accept **mixed Bangla + LaTeX** (`$..$`, `$$..$$`, `[$$..$$]`, `\(..\)`, Word linear format, raw Unicode math — all handled)
- `image` accepts a full Google Drive share link; the file ID is extracted automatically
- `options_N_is_correct = 1` marks the right answer

---

## 🔐 Security & Access Control

- **Signed sessions, not stored sessions** — tokens are `HMAC-SHA256(email|timestamp, SECRET)`; forging one requires the server secret, and expiry is enforced server-side (7 days)
- **Defense in depth** — permissions are checked at *every* layer: stats, class lists, subject lists, chapters, questions, and again per-question during slide generation
- **Server-side truth** — the frontend sends only question *identifiers*; all content is re-read from the Sheet and fingerprint-verified (class/subject/chapter/serial) before a single slide is built
- **Instant kill-switch** — deactivating a row in `User_Info` invalidates a teacher's access on their very next request

---

## 💡 How It Makes Life Easier

| For... | The Win |
|---|---|
| **Teachers** | A week of slide-making collapses into minutes. Pick questions, click once, present. The answer slides — with correct options highlighted in green — are made for live classroom reveals. |
| **Raw File Production** | Teachers can choose any number of questions by checking or unchecking them, then export the selected questions as a JSON file with one click. |
| **Designers** | Restyle slide #1 of the template once; every deck generated afterward inherits the new look. |
| **The IT budget** | $0/month. Google hosts the backend, the data, and the decks. The frontend is three static files. |

---

## Roadmap

- [ ] Direct in-dashboard question editing (write-back to Sheet)
- [ ] Printable quiz/exam PDF export alongside slides
- [ ] Per-teacher generation analytics
- [ ] Hashed passwords + optional Google OAuth sign-in
- [ ] Full UI internationalization
- [ ] Automatic question-image placement on generated slides

---
