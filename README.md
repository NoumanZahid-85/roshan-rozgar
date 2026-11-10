# Roshan Rozgar (روشن روزگار)

> **AI-Powered Voice-First Marketplace for Skilled Trade Workers**  
> *A comprehensive technical and architectural record built during the CUI Tech Fest AI Web Forge Hackathon (10 November 2026).*

---

### Project Overview

| Attribute | Details |
| :--- | :--- |
| **Event** | CUI Tech Fest, AI Web Forge hackathon |
| **Date** | 10 November 2026 (First commit 10:53, last commit 15:43 PKT — 66 total commits) |
| **Repository** | [taciturn2021/hackathon-roshan](https://github.com/taciturn2021/hackathon-roshan) |
| **Live Site (on the day)** | [https://hackathon.taciturn2021.dpdns.org](https://hackathon.taciturn2021.dpdns.org) |
| **Team Members** | Muhammad Abdullah, Humza Masoom, Abdul Muizz, Nouman Zahid |

---

### Table of Contents

- [1. What It Is](#1-what-it-is)
- [2. Who Did What](#2-who-did-what)
  - [Timeline of the Day](#timeline-of-the-day)
  - [Branches and Pull Requests](#branches-and-pull-requests)
- [3. Screenshots and UI Walkthrough](#3-screenshots-and-ui-walkthrough)
  - [Desktop Landing Experience](#desktop-landing-experience)
  - [Public Pages & Worker Sign-in](#public-pages-and-sign-in-live-site)
  - [Worker Side: Voice Onboarding & Management (Urdu RTL)](#worker-side-demo-mode-urdu-right-to-left)
  - [Customer Side: Discovery, Chat & Review (English/Urdu)](#customer-side-demo-mode-english)
  - [Design Rules the Screens Follow](#design-rules-the-screens-follow)
  - [How Voice Onboarding Works](#how-voice-onboarding-works-screens-10-to-14)
- [4. Architecture](#4-architecture)
- [5. Tech Stack](#5-tech-stack)
- [6. AI Models and Flows](#6-ai-models-and-how-they-are-used)
  - [What Jev Is](#what-jev-is)
  - [Inline AI](#inline-ai-the-user-waits-on-the-screen-about-2-to-6-s)
  - [Queued AI](#queued-ai-runs-in-the-background-the-result-arrives-later)
  - [Safety Nets in the AI Code](#safety-nets-in-the-ai-code)
  - [What We Measured](#what-we-measured)
- [6b. Caching, Uploads and Design Rationale](#6b-caching-uploads-and-design-reasons)
- [7. Database Schema & Booking Guarantee](#7-database)
- [8. API Specification & Error Handling](#8-api)
- [9. Authentication & Firebase Setup](#9-sign-in)
- [10. Repository Map](#10-repository-map)
- [11. Running the Project Locally](#11-run-it)
  - [Environment Variables](#environment-variables-repo-root-env-never-committed)
  - [Key Test Suites](#the-tests-that-matter-most)
  - [Production Deployment](#production-deploy-as-on-the-day)
- [12. Seeded and Mocked Data](#12-seeded-and-mocked)
- [13. Known Limitations and Shortcuts](#13-known-limits-and-deliberate-shortcuts)
- [14. References & Documentation Index](#14-where-to-read-more)

---

## 1. What It Is

A mobile-first web app that connects households across Islamabad and Rawalpindi with nearby verified skilled trade workers: electricians, plumbers, AC technicians, carpenters, and mechanics.

The core differentiator: **the worker never has to read or type.** Many skilled trade workers have limited literacy, so the entire worker-facing experience is operated via spoken Urdu. Workers onboard by answering four natural-language voice questions, receive job alerts as short Urdu voice briefs, and negotiate prices via audio notes. The customer side is an intuitive bilingual English/Urdu interface.

### End-to-End User Journeys

| # | Outcome | Mechanism |
| :-: | :--- | :--- |
| **1** | **Onboarding** | Worker answers 4 spoken questions (Name, Trade, Area, Years of experience). AI parses and constructs the profile; the worker confirms with a single tap. |
| **2** | **Discovery** | Customer describes problem via text, voice, or photo. AI classifies the trade and estimates price range. Available nearby workers appear dynamically, nearest first. |
| **3** | **Communication** | In-app real-time chat with audio voice notes and transcripts. Quoted prices become structured interactive offer cards, accepted with one tap. |
| **4** | **Booking** | Accepting an offer books the job immediately. A unique PostgreSQL partial index enforces strict single-job concurrency per worker. |
| **5** | **Trust** | Star ratings accompanied by audio or text reviews on completed jobs. AI automatically analyzes sentiment, assigns tags, and updates the worker's trust card. |

---

## 2. Who Did What

Folder ownership was defined in the architecture plan (`docs/6-IMPLEMENTATION_PLAN.md`). The summary below is derived directly from the git commit history of `main` to reflect the actual delivered work.

| Team Member | GitHub | Planned Area | Delivered Scope (from Git History) |
| :--- | :--- | :--- | :--- |
| **Muhammad Abdullah** | [`taciturn2021`](https://github.com/taciturn2021) | Backend: `backend/cmd`, `internal/{platform,identity,workers,jobs,reviews}`, `db/` | Database schema design. Firebase token verification. Workers module (onboarding merge, profile, online status toggle, geospatial search). Jobs API, chat messages, offers, booking state machine with race tests. Reviews. WebSocket hub over Redis pub/sub. Asynq queue hooks. Rate limiting. Problem classification endpoint (`POST /api/problems`). Inline AI routines `Extract` and `Classify` (`internal/ai/inline.go`). Seed data generator (`backend/seed/seed.sql`). Production deployment stack (`deploy/`). Hand-off guides. **21 commits**. |
| **Humza Masoom** | [`humzamasoom`](https://github.com/humzamasoom) | Repo, Docker Compose, CI, merging to `main` | Initial repository scaffolding and planning documents. Docker Compose orchestration and CI pipeline. Migrated authentication to Firebase. Local automated test harness. Real-time API integration with web screens. Phone sign-in flows. 4-step voice onboarding interface. English/Urdu bilingual toggle. Landing page redesign, branding, headers, and footers. Worker profile editing. OpenStreetMap location picker. Urdu audio asset integration. Main branch merges. **26 commits**. |
| **Abdul Muizz** | [`a-muizz28`](https://github.com/a-muizz28) | Frontend: `web/` | Initial web UI implementation on branch `ui`: signed-in app shell, search and filter flows, real-time job chat layout, brand styling, accessibility audits and review screen styling. **6 commits, ~3,400 lines**. |
| **Nouman Zahid** *(me)* | [`NoumanZahid-85`](https://github.com/NoumanZahid-85) | AI: `backend/internal/{ai,tasks}`, seed, README, prompt audio | Core AI service (`gateway.go`, `urdu.go`): unified client for Vercel AI Gateway handling Speech-to-Text, LLM completions, Jev decision trees, and Text-to-Speech with fallbacks, caching, timeouts, and output validation. Background queue workers (`tasks.go`). 17 comprehensive unit tests. Urdu STT/TTS model benchmarking. **1 commit, 1,087 lines (`9a6d680`)**. |

> [!NOTE]
> Tasks planned for Nouman that were completed by teammates due to hackathon time constraints: `Extract` and `Classify` helpers and seed data (Abdullah), final web README (Humza & Abdul Muizz), audio prompt asset commits (Humza).

### Timeline of the Day

Planned milestones: Figma 12:00, Full flow integrated 14:30, Code freeze 15:00, Final submission 15:30 (Hard deadline: 15:45).

| Time | Key Milestone (from Git commits) |
| :---: | :--- |
| **10:53** | Scaffolding, specification documents, Docker Compose, and CI pipeline committed. |
| **11:06** | Authentication architecture shifted to Firebase Auth. |
| **11:10 – 11:46** | Identity, database schema, booking state machine, concurrency race tests, jobs, reviews, and worker API completed. |
| **11:57** | Onboarding refined to 4-question voice flow with automated Urdu audio prompts. |
| **12:08** | Phone sign-in verification flow connected. |
| **13:03** | AI Gateway service, Urdu pipeline, and 4 asynchronous queue task handlers committed (`9a6d680`). |
| **13:09 – 13:27** | Problem extraction, classification endpoints, seed records, and rate limiting integrated. |
| **13:31** | First complete frontend interface delivered (branch `ui`). |
| **13:47 – 15:27** | Web app connected to backend API, language switch, responsive landing page, map picker integrated. |
| **15:28 – 15:43** | Final UI polish, distinct sign-in gates for workers and customers committed (Submission freeze: 15:43). |

### Branches and Pull Requests

| Branch | Owner | Purpose & Contribution |
| :--- | :--- | :--- |
| `main` | Humza | Production trunk, pull-request-only merges |
| `abdullah` | Abdullah | Backend services, database, deploy pipeline (Merged via PRs #1 – #5) |
| `nouman` | Nouman | AI Gateway service and asynchronous worker tasks (Merged into `abdullah`, then PR #2) |
| `ui` | Abdul Muizz | Initial frontend layouts and app screens |
| `ui-consistency` | Humza | Polished landing page, unified UI themes, profile modification |
| `map-picker` | Humza | Interactive OpenStreetMap coordinate selector |

---

## 3. Screenshots and UI Walkthrough

All mobile captures were taken on 10 November 2026 at mobile viewport width (390 px).

> [!TIP]
> The first five screenshots are captured directly from the live deployed site. Subsequent screens utilize the application's built-in demo environment (`VITE_MOCK_API=true`), which simulates live operational state in the browser without encountering SMS rate caps or automated reCAPTCHA picture challenges.

### Desktop Landing Experience

<div align="center">
  <img src="screenshots/01-landing-desktop.png" alt="Desktop Landing Page" width="900" style="border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.15);" />
  <p><em>Desktop Landing Page: Bilingual hero section showcasing role-based doors and instant onboarding.</em></p>
</div>

---

### Public Pages and Sign-in (Live Site)

| Landing (Mobile) | Role Selection | Worker Sign-in |
| :---: | :---: | :---: |
| <img src="screenshots/02-landing-mobile.png" width="240" alt="Landing Mobile" /> | <img src="screenshots/03-role-pick.png" width="240" alt="Role Selection" /> | <img src="screenshots/04-worker-signin-phone.png" width="240" alt="Worker Sign-in" /> |

| Phone Number Typed | SMS Code Verification | Worker Home (Status Toggle) |
| :---: | :---: | :---: |
| <img src="screenshots/05-worker-signin-number-typed.png" width="240" alt="Phone Number Typed" /> | <img src="screenshots/06-worker-signin-code.png" width="240" alt="SMS Code Verification" /> | <img src="screenshots/15-worker-home.png" width="240" alt="Worker Home" /> |

---

### Worker Side (Demo Mode, Urdu, Right to Left)

| Voice Onboard: Q1 (Name & Trade) | AI Real-Time Processing | Understood & Next Question |
| :---: | :---: | :---: |
| <img src="screenshots/10-worker-onboard-question1.png" width="240" alt="Voice Onboard Question 1" /> | <img src="screenshots/11-worker-onboard-understanding.png" width="240" alt="AI Understanding" /> | <img src="screenshots/12-worker-onboard-understood.png" width="240" alt="Question Understood" /> |

| Location Confirmation | Profile Review & Confirmation | Worker Assigned Jobs |
| :---: | :---: | :---: |
| <img src="screenshots/13-worker-location.png" width="240" alt="Worker Location" /> | <img src="screenshots/14-worker-confirm.png" width="240" alt="Profile Review" /> | <img src="screenshots/16-worker-app-jobs.png" width="240" alt="Worker Jobs" /> |

<div align="center">
  <br />
  <p><strong>Worker Public Profile & Trust Summary</strong></p>
  <img src="screenshots/16-worker-app-profile.png" width="300" alt="Worker Public Profile" style="border-radius: 8px; border: 1px solid #e1e4e8;" />
</div>

---

### Customer Side (Demo Mode, English)

| Customer Sign-in | Customer Dashboard | Describe Problem |
| :---: | :---: | :---: |
| <img src="screenshots/20-customer-signin.png" width="240" alt="Customer Sign-in" /> | <img src="screenshots/21-customer-dashboard.png" width="240" alt="Customer Dashboard" /> | <img src="screenshots/22-customer-describe-problem.png" width="240" alt="Describe Problem" /> |

| Trade & Price Range Verification | Location Map Selection | Available Workers Nearby |
| :---: | :---: | :---: |
| <img src="screenshots/23-customer-check-trade.png" width="240" alt="Check Trade" /> | <img src="screenshots/24-customer-location.png" width="240" alt="Location Map" /> | <img src="screenshots/25-customer-worker-list.png" width="240" alt="Nearby Workers" /> |

| Worker Trust Card | Real-Time Job Chat | Make Structured Offer |
| :---: | :---: | :---: |
| <img src="screenshots/26-customer-worker-profile.png" width="240" alt="Worker Card" /> | <img src="screenshots/27-customer-job-chat.png" width="240" alt="Job Chat" /> | <img src="screenshots/28-customer-make-offer.png" width="240" alt="Make Offer" /> |

| Job Status Tracking | Active & Past Jobs List | Leave a Review |
| :---: | :---: | :---: |
| <img src="screenshots/29-customer-job-status.png" width="240" alt="Job Status" /> | <img src="screenshots/30-customer-jobs.png" width="240" alt="Jobs History" /> | <img src="screenshots/31-customer-review.png" width="240" alt="Leave Review" /> |

---

### Design Rules the Screens Follow

1. **Worker Interface is Pure RTL Urdu** with *Noto Nastaliq Urdu* typography. Every screen item adheres to a three-tier information hierarchy:
   - Primary visual icon and large numerical representation.
   - High-contrast Urdu textual summary.
   - Accessible loudspeaker audio icon to play vocalized instructions.
2. **Push-to-Listen Audio Model:** Nothing auto-plays unexpectedly except the sequential onboarding questions and the incoming job ringtone. All other audio plays strictly upon user tap.
3. **Flexible Customer Experience:** Customers can interact via English, Urdu script, or Roman Urdu. Interface preferences persist per profile in local storage.
4. **Ergonomic Touch Targets:** Touch elements maintain a minimum target size of 56 px. Designed mobile-first with a desktop container limit of 480 px.
5. **Universal Color Language:** 
   - 🟢 **Green:** Positive action, accept, or online status.
   - 🔴 **Red:** Cancellation, rejection, or offline status.
   - 🟡 **Amber:** Pending state or waiting on counterparty.

### How Voice Onboarding Works (Screens 10 to 14)

```mermaid
flowchart TD
    A[Play Recorded Urdu Question] --> B[Worker Taps Mic & Speaks]
    B --> C[Send WAV audio to POST /api/workers/onboard]
    C --> D[Next Question Plays Immediately while AI processes]
    D --> E{AI Understood Details?}
    E -- Yes --> F[Display Parsed Info & Advance]
    E -- No --> G[Ask Once More or Fallback to Visual Icons]
    F --> H[One-tap Profile Confirmation PUT /api/workers/me]
    G --> H
```

1. Screen plays a pre-recorded Urdu question. Worker taps the mic, records audio, and taps again.
2. The audio payload is dispatched to `POST /api/workers/onboard` with the target field parameter (`name`, `trade`, `area`, `years_exp`). The subsequent question begins playing asynchronously while the server parses the previous clip.
3. The interface renders the parsed parameters in real time. If a user provides combined answers (*"Main Akram hoon, plumber"*), multiple fields populate simultaneously and redundant questions skip automatically.
4. If an answer cannot be parsed with high confidence, the system replays what it understood once; if ambiguous a second time, the interface provides visual trade icons.
5. To avoid clipped audio at word boundaries, the browser microphone stream initializes once upon screen mount and remains active.
6. Worker confirms location and saves profile (`PUT /api/workers/me`). The voice response for trade serves as the worker's introductory voice greeting.

---

## 4. Architecture

<div align="center">
  <img src="screenshots/architecture-diagram.jpeg" alt="System Architecture Diagram" width="850" style="border-radius: 8px; border: 1px solid #e1e4e8;" />
  <p><em>Modular monolith architecture: Single Go binary powering API and Worker runtimes backed by PostgreSQL and Redis.</em></p>
</div>

- **Single Binary, Dual Mode:** `app api` serves HTTP and WebSocket traffic; `app worker` executes asynchronous AI queue tasks. Both modes share service modules and business logic guarantees.
- **Modular Monolith Design:** Internal structure partitioned across `internal/{identity,workers,jobs,reviews,problems,ai}`. Modules interact strictly via formal Go service interfaces.
- **Persistent Storage & Queues:** PostgreSQL 16 stores relational records; Redis 7 serves as the Asynq task broker, pub/sub backend for WebSocket distribution, AI response cache, and token-bucket rate limiter.
- **Managed External Integrations:** Firebase Auth (JWT identity), Vercel AI Gateway (unified AI API router), and OpenStreetMap (cartographic tiles).
- **No Python Runtime Dependency:** All AI orchestration executes as direct HTTP requests from Go to remote inference endpoints, eliminating cross-process network hops.
- **Deterministic Pipeline (No Agent Frameworks):** Execution steps are fixed and predictable: Transcribe → Jev Decision → Conditional LLM Completion.
- **Lean Messaging:** Utilizes Redis pub/sub with client-side reconnection refetching rather than heavyweight Kafka messaging.

---

## 5. Tech Stack

| Layer | Technologies Selected | Details |
| :--- | :--- | :--- |
| **Frontend** | React 19, Vite 8, TypeScript 7 | Lightweight CSS custom router (`web/src/lib/router.tsx`), Leaflet 1.9 with OpenStreetMap, Lucide icons, Inter & Noto Nastaliq Urdu fonts, Oxlint. |
| **Authentication** | Firebase Auth | Client JS SDK v13 with Admin Go SDK v4 on backend. Phone authentication with SMS OTP, Google OAuth, and Password flows. |
| **Backend** | Go 1.27, `net/http` | Zero-framework standard library routing, `pgx` v5 for high-performance PostgreSQL queries, `go-redis` v9, `coder/websocket`. |
| **Background Queue** | Asynq 0.26 | Redis-backed asynchronous queue with integrated `asynqmon` inspection dashboard. |
| **Database** | PostgreSQL 16 | Relational store with custom partial unique indexing and Haversine distance functions. |
| **Caching & Pub/Sub** | Redis 7 | 24-hour idempotent AI caching, WebSocket event broadcast, and token-bucket rate limiting. |
| **AI Gateway** | Vercel AI Gateway | Centralized proxy for transcription, classification, completions, and speech generation. |
| **Storage & Media** | Local Docker Volumes | Persistent volume mounted at `/media/*` for user audio notes and uploaded images. |
| **Orchestration** | Docker Compose | Multi-container composition (`web`, `api`, `worker`, `postgres`, `redis`, `asynqmon`). |
| **Production Host** | Oracle Cloud ARM64 | Nginx reverse proxy with SSL termination and Cloudflare Tunnel integration. |
| **CI Pipeline** | GitHub Actions | Automated checks: `go vet`, race tests, Oxlint, and Vite production bundle compilation. |

---

## 6. AI: Models and How They Are Used

All models are routed through Vercel AI Gateway authenticated via a single key (`AI_GATEWAY_API_KEY`).

| Step | Primary Model | Fallback Model | Gateway Route |
| :--- | :--- | :--- | :--- |
| **Urdu Speech-to-Text** | `microsoft/mai-transcribe-2` (`ur-PK`) | `google/gemini-3.5-transcribe` | `/v4/ai/transcription-model` |
| **Decision Trees** | `typesafe-ai/jev` | `openai/gpt-6-luna-decisions` | `/v1/decisions` |
| **Extraction & Urdu Text** | `anthropic/claude-haiku-5.5` | `openai/gpt-6-luna` | `/v1/chat/completions` |
| **Urdu Text-to-Speech** | `elevenlabs/eleven-v4` | `google/gemini-3.8-flash-tts` | `/v4/ai/speech-model` |

### What Jev Is

**Jev** is a high-speed probability decision model. Rather than generating text, it accepts raw input and a series of binary propositions, returning a confidence score between `0.0` and `1.0`. Scores $\ge 0.6$ evaluate as `true`. If confidence is below this threshold, the application falls back to manual visual pickers.

```mermaid
graph LR
    Input[User Voice/Text] --> Jev[Jev Decision Engine]
    Jev -->|Score >= 0.6| Yes[Trigger Targeted LLM Extraction]
    Jev -->|Score < 0.6| No[Fallback to Visual Touch UI]
```

- **Price Detection in Voice Notes:** An LLM is only called to extract monetary values if Jev detects a price proposal (*"پندرہ سو"* scores `0.98`; *"Kitne paise lenge?"* scores `0.08`).
- **Trade Routing:** 5 parallel binary queries, one per trade category; the highest score $\ge 0.6$ claims the job.
- **Review Content Filtering:** Evaluates flags: `on_time`, `fair_price`, `clean_work`, `polite`, `abusive_or_spam`. Reviews flagged as `abusive_or_spam` are automatically suppressed from public trust scores.

### Inline AI (Synchronous: ~2 to 6s)

| Route | Function | Pipeline Steps |
| :--- | :--- | :--- |
| `POST /api/workers/onboard` | `ai.Extract` | Speech-to-text → LLM extracts name, trade, area, years as structured JSON → Jev verifies trade category → Sector normalization (*"جی ٹین"* $\to$ `"G-10"`). |
| `POST /api/problems` | `ai.Classify` | Audio & photo input → LLM derives trade, urgency level, and summary → Jev confirms trade category. |

> [!TIP]
> If inline AI is disabled or fails, endpoints immediately return HTTP 200 with default fields, seamlessly presenting manual tap selectors.

### Queued AI (Asynchronous Background Tasks)

| Task | Event Trigger | Pipeline Steps | Target Database Columns |
| :--- | :--- | :--- | :--- |
| `transcribe` | Audio message sent | STT $\to$ Jev price check $\to$ LLM price extraction. | `messages.body`, `messages.suggested_price` |
| `message speech` | Customer types text | LLM converts to colloquial Urdu $\to$ ElevenLabs TTS. | `messages.body_ur`, `messages.speech_url` |
| `job brief` | New job created | LLM drafts concise Urdu audio brief (trade, urgency, distance) $\to$ TTS. | `jobs.brief_ur`, `jobs.brief_audio_url` |
| `review analyze` | Review posted | STT $\to$ Jev tag classification $\to$ LLM generates updated rolling trust summary. | `reviews.tags`, `worker_profiles.trust_summary` |

### Safety Nets in the AI Code

- **Dual-Model Fallbacks:** Automatic failover to secondary model upon timeout, gateway error, or malformed response.
- **Script Validation:** Urdu completions are validated against Arabic/Urdu unicode blocks; Devanagari text is automatically converted back to Urdu script.
- **Strict Timeouts:** Hard timeout of 15 seconds per external AI request.
- **Idempotency & Result Caching:** Responses cached in Redis for 24 hours based on payload hash, preventing duplicate token spend on retries.
- **Dead-Letter Handling:** Malformed payloads and deleted references immediately abort retries.
- **Global Daily Quota:** Enforces a hard budget limit of 3,000 gateway calls daily (`AI_DAILY_CAP`).

---

## 6b. Caching, Uploads and Design Reasons

### Redis Cache Policy

| Cached Resource | Expiration Strategy |
| :--- | :--- |
| AI Inference Results | 24 hours (hashed input key) |
| Standard Trade Price Ranges | 5 minutes |
| Worker Rating Stats & Totals | 10 minutes (purged on new review creation) |
| **Worker Availability & Geospatial Search** | **NEVER CACHED** — Directly queries Postgres to guarantee single-job booking concurrency. |

### Upload Constraints

- **Maximum Payload:** 20 MB per multipart request.
- **Supported Formats:** Audio (`.wav`, `.webm`, `.ogg`, `.mp3`, `.m4a`), Images (`.jpg`, `.jpeg`, `.png`, `.webp`).
- Browser records audio in native WAV format to ensure full compatibility with the Microsoft STT engine.

---

## 7. Database

Defined across five core tables and one view in `backend/db/schema.sql`:

```mermaid
erDiagram
    users ||--o| worker_profiles : "owns"
    users ||--o{ jobs : "requests (customer)"
    worker_profiles ||--o{ jobs : "assigned (worker)"
    jobs ||--o{ messages : "contains"
    jobs ||--o| reviews : "receives"

    users {
        string id PK "Firebase UID"
        string role "worker | customer"
        string phone
        string name
    }
    worker_profiles {
        string user_id FK
        string trade
        float latitude
        float longitude
        boolean is_online
        string trust_summary
    }
    jobs {
        uuid id PK
        string status "requested | confirmed | in_progress | completed"
        decimal agreed_price
    }
```

### The Booking Guarantee

Concurrency conflicts are solved directly within PostgreSQL via a partial unique index:

```sql
CREATE UNIQUE INDEX one_active_job_per_worker
  ON jobs (worker_id) WHERE status IN ('confirmed', 'in_progress');
```

> [!IMPORTANT]
> If two customers accept offers from the same worker concurrently, PostgreSQL guarantees exactly one succeeds. The second receives `409 worker_busy`. Verified under load in `backend/internal/jobs/booking_test.go`.

---

## 8. API

All requests target `/api` and require `Authorization: Bearer <Firebase ID token>`.

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `POST` | `/api/auth/session` | Establishes user session record after Firebase sign-in |
| `GET` | `/api/me` | Fetches active user profile and worker metadata |
| `POST` | `/api/workers/onboard` | Parses spoken answer into structured worker attributes |
| `PUT` | `/api/workers/me` | Updates worker profile and service preferences |
| `PUT` | `/api/workers/me/online` | Toggles worker online/offline availability |
| `GET` | `/api/workers?trade=&lat=&lng=&radius_km=5` | Retrieves nearest online and unassigned workers |
| `GET` | `/api/workers/{id}` | Fetches public worker trust card and recent reviews |
| `POST` | `/api/problems` | Submits problem description for AI classification |
| `POST` | `/api/jobs` | Initializes a new job request |
| `GET` | `/api/jobs`, `/api/jobs/{id}` | Retrieves job list or single job message thread |
| `POST` | `/api/jobs/{id}/messages` | Sends text, voice note, or structured price offer |
| `POST` | `/api/jobs/{id}/offers/{id}/accept` | Confirms job booking (or returns `409 worker_busy`) |
| `POST` | `/api/jobs/{id}/offers/{id}/reject` | Declines active price offer |
| `POST` | `/api/jobs/{id}/status` | Updates job state (`in_progress`, `completed`, `cancelled`) |
| `POST` | `/api/jobs/{id}/review` | Submits rating, voice audio, and feedback |
| `GET` | `/api/ws?token=` | Server-sent event stream over WebSocket |
| `GET` | `/api/healthz` | Health check for PostgreSQL and Redis connectivity |

---

## 9. Sign-in

- Dual sign-in doors tailored by persona: workers use simple numeric SMS verification; customers have phone, Google OAuth, and email options.
- Demo test accounts bypass SMS carrier quotas:
  - **Worker:** `+92 300 0000001` (Code: `123456`)
  - **Customer:** `+92 300 0000002` (Code: `123456`)
- Development bypass mode: Setting `AUTH_DEV=1` allows testing via `Authorization: Bearer dev:<username>` without Firebase credentials.

---

## 10. Repository Map

```
.
├── README.md                      # Comprehensive project & architecture record
├── screenshots/                   # Complete collection of 27 UI and mobile flow screenshots
├── Images/                        # Architecture diagrams and system graphic assets
├── backend/
│   ├── cmd/app/main.go            # Binary entrypoint: HTTP API and background task worker
│   ├── db/schema.sql              # PostgreSQL schemas, unique indices, and views
│   ├── internal/
│   │   ├── ai/                    # Vercel AI Gateway client, STT/TTS, Jev routing (Nouman)
│   │   ├── tasks/                 # Asynq background queue consumers (Nouman)
│   │   ├── identity/              # Firebase token validation and user state
│   │   ├── workers/               # Worker profile, geospatial search, and onboarding
│   │   ├── jobs/                  # Jobs, chat messaging, and booking state machine
│   │   ├── reviews/               # Review storage and rating aggregation
│   │   └── platform/              # HTTP middleware, rate limiting, and WebSocket hub
│   └── seed/seed.sql              # Preloaded demographic seed data
├── web/
│   ├── src/screens/               # React UI screens (Landing, Onboarding, Chat, Jobs)
│   ├── src/lib/                   # API clients, audio recorder, Firebase, and i18n
│   ├── src/contract.ts            # Frontend/backend shared API interface contract
│   └── public/audio/              # 14 localized Urdu prompt audio clips
└── deploy/                        # Production Docker Compose and Nginx reverse proxy configs
```

---

## 11. Run It

```bash
# 1. Prepare environment variables
cp .env.example .env

# 2. Launch complete service stack
docker compose up --build

# 3. Seed demo data (once)
docker compose exec -T postgres psql -U roshan -d roshan < backend/seed/seed.sql
```

| Service | Access URL |
| :--- | :--- |
| **Web Frontend** | `http://localhost:5173` |
| **Backend REST API** | `http://localhost:8080/api/healthz` |
| **Asynq Task Dashboard** | `http://localhost:8081` |

### Key Test Suites

- **Concurrency Race Test:** `backend/internal/jobs/booking_test.go` (`TestRaceTwoAcceptsSameWorker`) validates that two simultaneous offer acceptances for one worker result in exactly one booking and one 409 conflict.
- **End-to-End Flow:** `backend/cmd/app/e2e_test.go` executes the complete customer-worker lifecycle against live database instances.
- **AI & Queue Unit Tests:** `backend/internal/ai/ai_test.go` and `backend/internal/tasks/tasks_test.go` validate model failover, price extraction, Urdu transcription normalization, and daily budget counters.

---

## 12. Seeded and Mocked Data

- **Demographic Seed Data:** 40 fictional workers across Islamabad and Rawalpindi (8 per trade), 8 customers, 95 completed/cancelled jobs, and 90 historical reviews.
- **Reference Location:** Centered at Sector G-10, Islamabad (`33.6844, 73.0145`).
- **Prompt Audio:** Pre-synthesized Urdu audio prompts generated with ElevenLabs models to guarantee consistent tone and low latency.

---

## 13. Known Limits and Deliberate Shortcuts

- Web client currently relies on 4-second polling intervals for active chats; backend WebSocket pipeline is implemented but awaiting web integration.
- Media audio and photos are stored on local persistent Docker volumes; will transition to S3/GCS object storage for multi-node deployments.
- Proximity search utilizes the SQL Haversine spherical formula; planned migration to PostGIS indexing for large-scale geographic scaling.

---

## 14. Where to Read More

| Resource | Description |
| :--- | :--- |
| `docs/1-PRD.md` to `docs/6-IMPLEMENTATION_PLAN.md` | Core engineering specifications: PRD, technical architecture, and implementation roadmap. |
| `HUMZA-INTEGRATE.md` | Frontend integration guidelines: error contracts, rate limits, and audio formats. |
| `backend/HANDOFF-NOUMAN.md` | Technical interface guide for plugging AI pipelines into backend workers. |
| `screenshots/` | Full collection of 27 high-resolution phone screenshots and UI flows. |
| `Images/` | System architecture diagram and high-level graphic overviews. |

---

<div align="center">
  <p><strong>Roshan Rozgar</strong> — Developed for CUI Tech Fest AI Web Forge Hackathon</p>
  <p>Authored by <a href="https://github.com/NoumanZahid-85">NoumanZahid-85</a></p>
</div>
