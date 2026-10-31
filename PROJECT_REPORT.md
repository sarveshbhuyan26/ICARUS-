# PrepPilot: Project Progress Report

**Project:** PrepPilot (QUIZBOT)  
**Problem statement:** PS 2 — Adaptive AI Study Companion  
**Report date:** 31 October 2026  
**Stage:** Working local prototype for hackathon early evaluation

## 1. Executive summary

PrepPilot is an adaptive study companion that helps learners choose what to practise,
respond to questions, understand mistakes and plan their next practice step. It combines
a Python learner model with a browser-based study interface and optional Gemini support.
The adaptive engine works independently of the language-model provider.

Work completed includes the initial quiz engine, verified-bank handling, setup and
account pages, a subject/chapter dashboard, a home hub, a progress page, a placement
check, user-supplied question-paper import, interactive study buddies, reporting,
focus blocks and session summaries. Reliability work addressed report isolation,
account-file concurrency, session resume, stale timers and placement/PDF regressions.

The latest local verification passed 18 API regression tests, 11 persistence and
access-control tests, 7 bookmark tests, offline AI-provider routing tests, engine
self-checks, checks covering 154 bank questions, and PDF extraction regression checks.
Learner progress, accounts, preferences, papers and bookmarks persist in SQLite, and
every learner resource is checked against the signed-in owner. The system is suitable
for a supervised local demonstration. Independent educational validation and public
deployment hardening remain unfinished.

## 2. Problem and objectives

A fixed quiz sequence treats learners alike even when their weaknesses and reasons
for mistakes differ. Correctness alone does not distinguish a rushed attempt, a
confident misconception, a time-pressure error or a possible conceptual gap.

PrepPilot's objectives are to:

1. Estimate understanding at chapter level and select relevant practice.
2. Use response evidence beyond right/wrong answers.
3. Explain why a question was chosen and what should happen next.
4. Support focused study with manageable practice blocks and breaks.
5. Allow practice on both supplied bank items and the learner's own papers.
6. Keep selected bank-backed practice usable without an AI key.

The project uses heuristics to infer possible learning needs; it cannot directly
observe a learner's thought process or diagnose fatigue clinically.

## 3. Current learner journey

1. **Entry:** Sign up, sign in or continue as a guest.
2. **Setup:** Select class/exam, subjects, starting difficulty, explanation style,
   timed mode and focus length. Available focus choices are 25, 40 and 60 minutes.
3. **Optional placement:** Complete a five-question check to estimate starting
   subject readiness. It attempts to spread questions across selected subjects.
4. **Home hub:** See continuation and recommended-next-practice cards, subject
   cards and saved personal paper entries.
5. **Chapter selection:** Select a subject, search its chapters and start a quiz.
6. **Practice:** Answer with confidence, request hints, skip or use timed mode.
   The engine updates the learner estimate and chooses follow-up practice.
7. **Review:** Inspect question-selection reasoning, progress, error patterns and
   a session summary with a recommended next step.
8. **Study rhythm:** The focus timer begins with the quiz. When the block ends,
   the current question can be finished before the server starts a break.

Reloading resumes an existing session while the same server is running. Chapter
switching now retains the learner model rather than resetting it. An active break
is carried across preference changes. Restarting the server still resets progress.

## 4. Features implemented

### 4.1 Adaptive learner model

The engine maintains chapter abilities, subject priors, attempt counts,
misconception flags and response history. An IRT-inspired Elo model estimates:

`P(correct) = 1 / (1 + exp(-(ability - question_difficulty)))`

Question selection balances chapter weakness, exploration, misconception signals
and recent repetition. Item difficulty targets roughly 60% estimated success when
suitable items exist. Explicit chapter focus constrains selection to that chapter.

Confidence, timing and hints affect learner updates. Answer switching contributes
to diagnosis and wellbeing signals. Very fast responses receive weaker evidence
weight; hinted correct answers receive reduced credit.

### 4.2 Feedback and learning support

The system labels possible error types: concept gap, careless slip, trap option
and time pressure. Depending on the pattern, it can request an easier follow-up,
switch topics or provide a confidence-building question.

The reasoning panel explains the selection in plain language, with technical
engine details available separately. Readiness, confidence calibration, simplified
negative-marking advice, predicted scores and highest-payoff chapters are shown
as estimates. They have not been calibrated against actual exam outcomes.

### 4.3 Study interface

The interface includes landing, login, setup, study hub, quiz, progress and paper-import
pages. The dashboard uses subject cards, chapter search and availability indicators.
The quiz result and reasoning panels were simplified to reduce clutter.
Desktop and mobile flows have received focused browser checks, including a
375-pixel layout; a complete accessibility audit has not been performed.

### 4.4 Class and exam support

Syllabus maps cover Classes 1–12, JEE Main, JEE Advanced and NEET. JEE Main was
added as a separate level with its own target difficulty and +4/−1 marking model.
JEE Advanced uses a simplified +3/−1 model and NEET uses +4/−1. These settings do
not model all possible section-specific exam rules.

Syllabus breadth exceeds question-bank coverage. Some levels and chapters require
AI generation; curriculum alignment still needs review against authoritative material.

### 4.5 Question bank and optional AI

The checked-in bank has 154 items across 46 distinct chapter labels:

| Category | Items | Current check |
| --- | ---: | --- |
| Original numeric questions | 54 | Recomputed answers matched against options |
| Original conceptual questions | 26 | Keys matched to stored reviewed answers |
| Imported JEE Advanced questions | 74 | Source-provided keys retained; structure checked |

Items failing the bank checker are excluded at startup. Passing these checks does
not guarantee educational correctness, particularly for conceptual and imported
items that are not independently solved by the checker.

Gemini optionally supplies generated questions, personalized explanations and
answers to doubts. Generated questions must agree with a second solve before
being accepted. Background prefetching, fallback models and rate-limit cooldown
handling are implemented. Two model calls can share errors, so agreement is not
a correctness guarantee. Provider availability, latency and quotas remain dependencies.

### 4.6 User-supplied question papers

Learners can paste typed questions, CSV content or a Google Sheets link. The browser
can extract text from PDF, DOCX and PPTX files before import. The importer recognises
question/option structures and answer-key sections, validates entries and reports
skipped questions. Uploaded papers are saved locally and appear as selectable quizzes.
Their quiz pools remain restricted to the uploaded questions rather than adding
AI-generated questions.

Uploaded answer keys are supplied by the user, not independently certified. Scanned
PDFs and image-only slides are unsupported without OCR. Complex reading order and
unusual layouts may still need manual correction. Legacy DOC/PPT files require
conversion to DOCX/PPTX.

### 4.7 Accounts and study buddies

Accounts use salted PBKDF2-SHA256 password hashes stored in SQLite; legacy
`users.json` accounts are imported unchanged. Sign-in issues a random token that is
stored only as a SHA-256 digest with an expiry and sent as an HttpOnly cookie. API
middleware identifies the owner of every request, rejects cross-origin writes, and
sessions, papers and bookmarks are only visible to their owner. Bookmarks can only be
saved after a question is answered or skipped, because they include the answer.

Three interactive painted study buddies—Pip, Mika and Leo—provide answer reactions,
encouragement and click interactions. Their visual assets are served locally.
Reduced-motion styles are present. Leo's sad asset currently duplicates his idle
image, as documented in the handoff notes.

## 5. Architecture and data

| Component | Responsibility |
| --- | --- |
| FastAPI / `main.py` | Routes, sessions, accounts, Gemini integration and study rhythm |
| `engine.py` | Learner updates, selection, diagnosis, strategy estimates and simulation |
| `syllabus.py` | Subject/chapter maps and exam levels |
| `papers.py` | Paper parsing, validation, saving and loading |
| HTML/CSS/JavaScript | Setup, dashboard, quiz, progress, imports and study-buddy interactions |
| Bank JSON files | Original questions and imported source-key items |

The frontend uses vanilla JavaScript, KaTeX and Chart.js. Fonts and some libraries
load from external CDNs. Runtime dependencies are pinned to FastAPI 0.162.0 and
Uvicorn 0.54.0; regression tests use httpx.

State storage currently works as follows:

- **SQLite (`preppilot.sqlite3`, `storage.py`):** accounts, hashed login tokens,
  preferences, learner sessions (JSON, never pickle), question papers and bookmarks.
- **Server memory:** a cache of active learners and AI-generated questions.
- **Browser storage:** display caches only; the server cookie is the identity.

The supported operating model is one server worker, because active learners are
cached in process.

## 6. Reliability fixes completed

| Issue | Resulting behavior |
| --- | --- |
| Reports deleted shared bank questions | Questions are hidden only for the reporting session; shared bank content is retained |
| Older report buttons targeted the latest answer | Reports carry exact answer IDs and operate on the matching response |
| Incomplete report rollback | Retained answer evidence is replayed to rebuild learner state and predicted-score history |
| Concurrent sign-ups could overwrite accounts | A lock protects read/modify/write operations and atomic replacement prevents partial files |
| Browser reload discarded progress | Matching browser sessions resume while the server remains running |
| Old browser timers caused immediate breaks | New sessions ignore stale browser clock values |
| Extra focus-time overlay duplicated setup | The original setup page owns preferences; the dashboard handles chapter selection |
| Chapter changes reset learning | Existing learner sessions are reused and only the focus changes |
| Placement could resume an ordinary session | Explicit placement requests start a placement-enabled session; active checks remain resumable |
| Reporting distorted placement estimates | Replay preserves placement-specific subject updates and restores missing assessment slots |
| PDF fragments merged adjacent options | Fragment boundaries retain whitespace while explicit line endings are preserved |

Placement/PDF fixes were implemented after new tests reproduced the earlier failures.
Their checks then passed with the changes applied.

## 7. Testing and verification

The latest local validation completed successfully:

- **18 API regression tests:** reporting isolation and replay, account concurrency,
  session resume, stale clocks, focus duration, chapter selection, break preservation,
  JEE Main, paper import, placement and hub/summary behavior.
- **Engine self-check:** ability direction, subject-prior effects, response weighting,
  diagnosis, wellbeing, strategy and synthetic adaptive comparisons.
- **Bank checker:** all 154 stored items pass their applicable checks.
- **PDF JavaScript regression checks:** the real extraction function is exercised
  with PDF.js-style fragmented options, wrapped text and page boundaries.
- **Targeted parser check:** corrected inline options are accepted by the paper parser.
- **Browser checks across development:** setup-to-chapter practice, timer timing,
  resume, mobile overflow, and normal-practice-to-placement transition. The latest
  placement browser check also confirmed that reloading resumes the assessment.

GitHub Actions is configured for Python checks and the PDF JavaScript check. A
successful local run is not a claim that a particular remote GitHub workflow has
executed. PDF fragment tests do not replace testing diverse real documents.

## 8. Simulation results and interpretation

The engine self-check averages 10 synthetic runs per profile:

| Artificial learner profile | Adaptive questions | Static questions | Reported reduction |
| --- | ---: | ---: | ---: |
| Uneven abilities | 235 | 421 | 44% |
| Weak all-round | 312 | 422 | 26% |
| Nearly exam-ready | 140 | 177 | 21% |

The simulator assumes learning gain peaks when difficulty is close to true ability.
The adaptive policy aims near ability, while the static comparator cycles chapters
with a broad difficulty distribution. This demonstrates behavior under a constructed
model; it does not establish improved outcomes in real learners. The UI uses a
20-run comparison, so displayed averages may differ from this self-check table.

## 9. Documentation and repository preparation

Work completed includes a root README, developer notes, architecture and roadmap,
evaluator/demo guide, submission description and abstract, contribution/security
notes, third-party attribution, issue/PR templates and CI configuration.

Earlier handoff work also produced clean upload packaging and a combined runnable
Python distribution with a README and description. Those were snapshots at the
time of creation; they must be regenerated from current source before submission
if they do not include later features and fixes.

The project remains unlicensed at the owner's request. JEEBench's upstream notice
is preserved for imported material. Private accounts, API keys, installed environments
and build-session transcripts must not be included in a public submission. Some
older documentation retains earlier test counts or milestone descriptions; this
report describes the current reviewed working tree, including uncommitted fixes.

## 10. Limitations and unfinished work

1. One server worker only: active learners are cached in process. Expired tokens and
   old guest data are not yet cleaned up, and there are no rate limits.
2. Question papers saved as files before the SQLite migration are not imported.
3. Some chapters depend on Gemini or Groq and can be affected by quota, latency or model availability.
4. Original conceptual keys, imported mappings and AI output need educator review.
5. No real learner study has established learning gains, diagnosis accuracy or score calibration.
6. PDF extraction is text-based; OCR and general layout recovery are not implemented.
7. Comprehensive accessibility, security and browser regression coverage is incomplete.
8. Prerequisite graphs, spaced repetition, exam-date planning and peer benchmarking
   have not been implemented.

## 11. Recommended next phase

**First:** strengthen verified content in a focused set of demo chapters and obtain
teacher review. **Second:** test short sessions with a small learner group and record
usability observations without overstating effectiveness. **Third:** automate the
main browser journey and test failure recovery for AI and document import.

A hackathon demonstration should emphasize the complete adaptive loop: placement,
chapter selection, answer evidence, visible reasoning, follow-up and actionable
session summary. The interface and buddies support that story; the learner model
and trustworthy feedback are the central contribution.

## 12. Conclusion

PrepPilot has progressed from an initial adaptive quiz implementation into an
integrated local learning prototype with setup, content selection, personal papers,
progress views and study support. Its core flows have targeted automated evidence,
and several important regressions have been reproduced and corrected. The next
milestone is to strengthen persistence, access control and educational validation
while keeping the demonstration focused and reliable.
