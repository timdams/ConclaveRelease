# Conclave

**A council of LLMs that helps you grade programming and writing assignments.**
WPF desktop tool for Windows. You are the chairman; the AIs offer advice.

> *A conclave is a closed room where advisors deliberate until there is white smoke. The council advises — the final decision rests with one person.*

![](main.jpg)

**⚠️IMPORTANT: you need an [openrouter.com](openrouter.com) api key for this to work. The free models from openrouter can be used to test but will generally NOT SUFFICE to do any meaningfull grading assitance. The tool will tell you the estimated cost for a given combination of models (so you it burn tokens without you knowing 😉)**

---

## What does this tool do?

You have a stack of 50 student folders with the same assignment. Conclave lets 2 to 3 LLMs independently read each submission, compares their judgments, and gives you a dashboard where per student you see in seconds:

- 📊 **Proposed grade** per AI + weighted Camerlengo advice
- 🔥⚪ **White smoke** when all AIs agree — candidate for a quick accept
- 🔥⚫ **Black smoke** when they disagree — that's where you need to look
- 🚩 **Red flags** with clickable code snippets that jump to the right line
- 🤖 **AI suspicion** when something smells suspiciously like Copilot
- 📁 **Diff** against your model solution per file

After that you fill in the grade, export a CSV, and you're done.

### Which assignments?

| Type | What | Files |
|---|---|---|
| **C#** | .NET projects, final assignment Programming | `.cs`, `.csproj` |
| **Generic code** | Python, Java, C++, … | configurable extensions |
| **Web** | HTML/CSS/JS/TS projects | `.html .css .js .ts` (ignores `node_modules/`, `dist/`) |
| **SQL** | DDL/DML assignments | `.sql` |
| **Writing** | Writing assignments, reports, essays | `.md`, `.docx`, `.pdf` |

When opening a new working folder, the tool detects the type and proposes it; you confirm or pick another type.

---

## Installation

### Requirements

- **Windows 10/11**
- **.NET 10 SDK** — [download](https://dotnet.microsoft.com/download/dotnet/10.0)
- At least one API key for an LLM provider (see below)

### Installing

You can grab the latest version via [releases](https://github.com/timdams/ConclaveRelease). Future updates will install automatically.

### Configuring API keys

Conclave talks to **OpenRouter** by default (one key, access to 200+ models — Claude, GPT, Gemini, DeepSeek, …). Optionally you can also use direct Anthropic/OpenAI keys.

Three ways, in order of precedence:

1. **Through the tool itself** *(recommended)*: menu **App → App settings…**, fill in keys, restart. Stored in `%APPDATA%\Conclave\user-settings.json`.
2. **Through `appsettings.local.json`** *(handy for debugging)*: copy into `src/Conclave.App/`, gitignored. Overrides app settings.
3. **Through environment variables**: `CONCLAVE_OPENROUTER__APIKEY`, `CONCLAVE_ANTHROPIC__APIKEY`, etc.

> 💡 **Free pilot run?** Create an OpenRouter account and use `:free` models (DeepSeek, Qwen3-Coder, …) — no cost. The Council Editor has a "free only" filter.

---

## Setting up a working folder

For each assignment you have one working folder with the student folders inside it. Conclave creates a separate `_conclave/` folder alongside them for all tool data; the student folders remain completely untouched.

```
2026-prog2-finalassignment\
├── _conclave\              ← Conclave writes everything here
├── Jansen_Pieter\          ← read-only source
├── DeSmet_Lara\
└── VanDenBergh_Mohammed\
```

### Opening for the first time

1. **App → Open working folder…** → point to the parent folder
2. **Pick a type** — based on the contents the tool proposes a type (C# / Python / web / SQL / writing). Confirm or pick another.
3. **Fill in `_conclave/`:**
   - `opdracht.md` — assignment description
   - `rubric.md` — criteria + weights
   - `modeloplossing/` — your reference solution
   - `students.json` — name ↔ pseudonym (the tool creates a template)
4. **Compose the council** — the **Edit Council** button opens a window with a live model comparer (cost + quality + free-only filter). Recommended: 3 members from different model families.

> ⚠️ **Important:** Source folders are **read-only** to the tool. Conclave never touches a student file. All tool data lives in `_conclave/`. You can safely replace a student folder when a resubmission comes in — your work won't be lost.

---

## Daily flow

### Dashboard

A table with all submissions, sorted by default on **spread (σ) descending** — that's where your added value sits.

| Student | Status | Stage 1 | Camerlengo | σ | 🚩 | My grade |
|---|---|---|---|---|---|---|
| VanDenBergh_M | analyzed | 11.7/20 | 11.2/20 | **3.2** 🔥⚫ | 4 | – |
| Jansen_P | analyzed | 14.3/20 | 14.5/20 | 0.9 🔥⚪ | 1 | – |
| DeSmet_L | judged | 15.0/20 | 15.1/20 | 0.5 🔥⚪ | 0 | 15.0 |

**Actions:**
- **Run stage 1 on all pending** — each member analyzes independently
- **Run stage 2** — anonymized cross-review (automatically skipped when there's consensus)
- **Run stage 3** — optional Camerlengo note with meta-observations + advisory grade
- **Export grades (CSV)** — Excel-friendly export for your admin

### Project view (per student)

Split screen with a code browser on the left (clickable findings jump to the right line), council output on the right, and a **Diff tab** against the model solution.

**Shortcuts:**

| Key | Action |
|---|---|
| `J` / `K` | Previous / next student |
| `Enter` | Save judgment + move to next |
| `A` | Accept council grade (only with high consensus) |
| `F` | Flag for later review |
| `Ctrl+L` | Open / close live log pane |
| `?` | Help overlay |

### Live observability during a run

At the bottom there's a collapsible log pane that shows per call: stage, member, model, latency, tokens, cost. Clicking a line opens the full transcript (system prompt + user prompt + response or error). On timeout / API error the pane pops open automatically.

---

## How does the council work?

Three stages, with automatic skip logic to save costs.

**Stage 1 — First opinions.** Each member independently receives: assignment, rubric, model solution, student work. Output is markdown with grade, rubric scores, red flags (with literal snippets), and AI suspicion.

**Stage 2 — Cross-review (anonymized).** Each member reads the other answers as Reviewer A/B/C and looks for observations that don't hold up. **Skipped** when grades are close together (σ ≤ threshold) and there are no high-severity flags — saves cost on routine cases.

**Stage 3 — Camerlengo note (optional).** A separate chairman AI delivers meta-observations ("where is the biggest disagreement?", "what should you concretely check?") plus its own advisory grade. A weighted mix with the stage-1 grades appears in the **Camerlengo** column.

> White smoke 🔥⚪ = consensus, quick review. Black smoke 🔥⚫ = look yourself.

---

## Privacy & security

- **Pseudonymization**: student names are replaced via `_conclave/students.json` before every API call. The UI just shows the real name.
- **Local**: all tool data lives on disk. Nothing goes to the cloud. What goes to OpenAI / Anthropic / OpenRouter is exclusively the pseudonymized prompt at the moment of the call.
- **Log files** (`_conclave/analyses/<student>/log.jsonl`) only contain tokens/cost/latency — no prompts or responses. Full prompts and responses live in the `transcripts/` folder, readable without the tool.

---

## What if…

**…I replace a student folder with a new submission?** Source hash changes → status automatically becomes `stale`. Open the student in the dashboard and run again.

**…I want to change the assignment type after the fact?** Working folder settings → "Change assignment type…". Already-analyzed submissions are automatically marked `stale`.

**…an API call times out or fails?** Default 3-minute deadline per call. The tool writes a failure transcript so you can see afterwards why it went wrong. Other council members just continue — one hanging member never blocks the whole stack.

**…I don't have a `council.json`?** The tool falls back to a default (1 member). You get a clear warning with provider + model before a single API call goes out. Three options: continue / open Council Editor / cancel.

**…I don't have a model solution?** The tool refuses to start ("Sede vacante" message). Fill in `_conclave/modeloplossing/` and reload.

---

## Folder structure after a run

```
_conclave/
├── .conclave-config.json       ← marks this as working folder, assignment type
├── opdracht.md, rubric.md
├── council.json                ← council configuration
├── modeloplossing/             ← your reference
├── prompts/                    ← stage prompts (editable)
├── students.json               ← name ↔ pseudonym
├── extracted/                  ← cached text extraction (.docx/.pdf → .md)
├── analyses/Jansen_Pieter/
│   ├── council/
│   │   ├── 01-first-opinions/   ← stage 1 markdown per member
│   │   ├── 02-cross-review/     ← stage 2 (anonymous)
│   │   ├── 03-chairman-sidenote.md
│   │   └── transcripts/         ← per call: prompts + response + meta
│   ├── tim-oordeel.md
│   ├── metadata.json            ← hash, status, grade
│   └── log.jsonl                ← tokens/cost/latency
└── runs/                        ← cross-project reports (M4)
```

Everything markdown / JSON on disk. Readable without the tool. Git-friendly. Grep-friendly.

---

## Hard principles

1. **Source folders are read-only.** The tool writes exclusively in `_conclave/`.
2. **No AI determines the binding final judgment.** The council advises; you decide.
3. **Pseudonymization before every API call.**
4. **Markdown is the truth.** All results readable without the tool.

---


---

## Terminology

| Concept | Term |
|---|---|
| Council member | Cardinal |
| Stage-1 round | Ballotage |
| Consensus, no flags | White smoke 🔥⚪ |
| Conflict / high spread | Black smoke 🔥⚫ |
| Judgment saved | Habemus iudicium |
| Stack completed | Habemus papam |
| Working folder without model solution | Sede vacante |
| AI chairman note | Camerlengo note |
