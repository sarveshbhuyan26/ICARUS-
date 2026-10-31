# PrepPilot: practice that adapts to every answer

**Hackathon submission · PS 2: Adaptive AI Study Companion**

PrepPilot is an adaptive study companion for **Class 1–12 (CBSE/NCERT), JEE Main, JEE Advanced and NEET**. It picks every question from what you got wrong, **explains why it chose it**, diagnoses *how* you fail, spaces your reviews so they stick, and enforces healthy study breaks. It runs as a website and an installable phone app.

> **Judges, the quickest way in:** run it (2 commands below), open http://localhost:8000, click **Continue as guest**, then answer a few questions in a chapter like *Laws of Motion*. Watch the **Live adaptation** graph and **Why this question** panel on the right.

---

## What to look at (5-minute tour)
| Where | What it shows |
|---|---|
| **Landing page** `/` | 3D *knowledge galaxy* (subjects are planets, chapters orbit them, coloured by readiness) and the pitch |
| **Home** `/study` | Recommended next chapter, mistakes due for review, daily goal, streak, exam plan |
| **Quiz** `/app` | Adaptive questions. After each answer: right/wrong animation, an **error diagnosis** (concept gap, careless slip, trap option, time pressure), the **live adaptation graph** with ↑ harder / ↓ easier, and **Why this question** |
| **Progress tab** (quiz, right side) | Score estimate, mistake patterns, **confidence vs reality** with Attempt/Skip advice for negative marking, ability vs difficulty graph |
| **Mistake review** `/review` | Spaced repetition: missed questions return after 1, 3 and 7 days |
| **AI tutor** `/tutor` | Ask anything, or upload notes (PDF/Word/PowerPoint) to get concepts explained and turned into a quiz |
| **Question papers** `/papers` | Paste or upload your own paper (or a Google Sheet) and it becomes an adaptive quiz |
| **Breaks** | `/app?demo=1` uses 2-minute focus blocks, so you can see the enforced break |

## What makes it different
1. **Explainable adaptation:** every question shows *why* it was picked, plus a live graph of skill vs difficulty.
2. **Understands how you fail:** confidence, time, hints and answer switching feed a per-chapter skill model, and each miss gets a specific fix.
3. **Exam strategy:** confidence calibration and negative-marking advice (JEE Advanced +3/−1, JEE Main and NEET +4/−1), plus a predicted score.
4. **Wellbeing built in:** 40-minute focus blocks, **server-enforced breaks**, and fatigue and frustration detection.
5. **Your own material:** question papers and notes become adaptive practice; disputed answer keys are checked by two independent AI solves.
6. **Honest content:** 154 verified questions, including **74 real JEE Advanced questions with official keys** (JEEBench, credited). AI questions are tagged and checked by a second solve. **It works with no AI at all.**

In simulation, adaptive selection reaches the readiness target with **21–44% fewer questions** than a fixed bank (`python engine.py`).

## Run it locally
Requires **Python 3.11+**.
```bash
cd preppilot
pip install -r requirements.txt
python -m uvicorn main:app --port 8000
```
Open **http://localhost:8000** and click **Continue as guest**.

**Optional AI** (tutor, explanations and extra questions): get a free Groq key at console.groq.com, then:
```bash
# from the repository root
cp .env.example .env.local      # then put GROQ_API_KEY=... (and optionally GEMINI_API_KEY=...) in it
cd preppilot && python start_local.py
```
Groq is used first, with Gemini as fallback. Without keys, everything except the AI features works.

**Install on a phone:** open the site over HTTPS (for example via a tunnel such as `cloudflared tunnel --url http://localhost:8000`), then use *Install app* (Android) or *Share → Add to Home Screen* (iPhone).

## Checks (all pass)
Run from `preppilot/`:
```bash
python engine.py            # adaptive engine self-check + simulation, prints "engine ok"
python verify_bank.py       # re-checks every answer key, prints "all answer keys verified"
python -B test_regressions.py
python -B test_persistence.py
python -B test_bookmarks.py
python -B test_reviews.py
python -B test_ai_providers.py
python -B test_notes.py
python -B test_account_extras.py
node test_pdf_import.js
```
These also run in GitHub Actions (`.github/workflows/checks.yml`). All tests run offline, with the AI faked.

## How it works
| Part | File |
|---|---|
| Learner model and policy: per-chapter IRT/Elo ability, question difficulty targeted at ~60% chance of success, chapter ranking (weakness + exploration + misconceptions), error diagnosis, wellbeing, calibration, predicted score, spaced-review schedule, simulation | `preppilot/engine.py` (pure Python, assert-based self-check) |
| API, sessions, focus blocks and breaks, AI routing, accounts, reviews, bookmarks, papers, notes tutor | `preppilot/main.py` (FastAPI) |
| Storage: accounts (PBKDF2 hashes), hashed expiring login tokens, sessions, preferences, papers, bookmarks, reviews | `preppilot/storage.py` (SQLite) |
| Syllabus maps (Class 1–12, JEE, NEET) | `preppilot/syllabus.py` |
| Question bank and answer-key verification | `preppilot/questions*.json`, `preppilot/verify_bank.py` |
| Pages | `preppilot/*.html`, `site.css`; 3D galaxy in `galaxy.js` (Three.js); installable app via `manifest.webmanifest`, `sw.js` |

**Stack:** Python, FastAPI, SQLite, vanilla JavaScript, Three.js, Chart.js, KaTeX. No build step.
More detail: [`preppilot/README.md`](preppilot/README.md), [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md), [`docs/PITCH_FACTS.md`](docs/PITCH_FACTS.md), [`PROJECT_REPORT.md`](PROJECT_REPORT.md). The pitch deck is in `docs/PrepPilot_Pitch_Deck.pptx`.

## Honest limits
- The verified question bank is small (154). Other chapters use AI-generated, self-checked questions, which need an AI key.
- No video lessons, and no real-student study yet.
- It's a single-server prototype; learner data lives in a local SQLite file.

## Credits
- **JEE Advanced questions:** JEEBench (Arora et al., EMNLP 2023), MIT licence. See `docs/JEEBENCH_LICENSE.txt` and `docs/THIRD_PARTY_NOTICES.md`.
- **Libraries** (from cdnjs): Three.js, Chart.js, KaTeX, pdf.js, mammoth, JSZip. **Fonts** from Google Fonts.
- **Study buddy art** (Pip, Mika, Leo): original characters, generated with Canva's AI image generator.

## Team
_Add team members and roles here._

## License
MIT. See [LICENSE](LICENSE).
