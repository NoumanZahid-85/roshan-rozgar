# Roshan Rozgar (روشن روزگار) — project record

A personal record of what we built, how it works and who did what. Written on 10 November 2026, the day of the hackathon, from the code on `main` (commit `11dc65d`) and the git history.


|                            |                                                                                     |
| ---------------------------- | ------------------------------------------------------------------------------------- |
| **Event**                  | CUI Tech Fest, AI Web Forge hackathon                                               |
| **Date**                   | 10 November 2026. First commit 10:53, last commit 15:43 (Pakistan time). 66 commits |
| **Repository**             | https://github.com/taciturn2021/hackathon-roshan                                    |
| **Live site (on the day)** | https://hackathon.taciturn2021.dpdns.org                                            |
| **Team**                   | Muhammad Abdullah, Humza Masoom, Abdul Muizz, Nouman Zahid                          |

**Contents:** 1 What it is · 2 Who did what (timeline, branches) · 3 Screenshots (design rules, voice onboarding) · 4 Architecture · 5 Tech stack · 6 AI models and flows (Jev, inline, queued, safety nets, measurements) · 6b Caching, uploads, design reasons · 7 Database · 8 API (errors, limits) · 9 Sign-in (Firebase setup) · 10 Repository map · 11 Run it (env variables, key tests, deploy) · 12 Seeded and mocked · 13 Known limits · 14 Where to read more

## 1. What it is

A mobile-first web app that connects households in Islamabad and Rawalpindi with nearby skilled trade workers: electricians, plumbers, AC technicians, carpenters and mechanics.

The idea that makes it different: **the worker never has to read or type.** Many trade workers have limited literacy, so the worker side runs on Urdu voice. The worker signs up by answering four spoken questions, receives jobs as short Urdu audio, and negotiates by voice notes. The customer side is a normal English/Urdu app.

The five things the app does (all five work, for both roles, in one flow):


| # | Outcome       | How                                                                                                                                           |
| --- | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| 1 | Onboarding    | The worker answers four spoken questions (name, work, area, years). AI builds the profile; the worker confirms with one tap                   |
| 2 | Discovery     | The customer types, speaks or photographs the problem. AI picks the trade. The list shows workers who are online, not on a job, nearest first |
| 3 | Communication | Chat with voice notes and transcripts. A price is a structured offer, accepted with one tap                                                   |
| 4 | Booking       | Accepting an offer confirms the job. The database allows one active job per worker                                                            |
| 5 | Trust         | Stars plus a voice or text review, only on a completed job. AI adds tags and a summary to the worker's profile                                |

## 2. Who did what

Folder ownership was fixed in the plan (`docs/6-IMPLEMENTATION_PLAN.md`). The "what was done" column is from the git history of `main`, so it shows the real work, including tasks that moved between people during the day.


| Person                | GitHub           | Planned area                                                                       | What they did (from git history)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ----------------------- | ------------------ | ------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Muhammad Abdullah** | `taciturn2021`   | Backend: `backend/cmd`, `internal/{platform,identity,workers,jobs,reviews}`, `db/` | Database schema. Firebase token verification. Workers module (onboarding merge, profile, online toggle, nearby search). Jobs API, chat messages, offers, the booking state machine and its race test. Reviews. WebSocket hub over Redis pub/sub. Queue contract and enqueue hooks. Rate limiting. `POST /api/problems`. The inline AI functions `Extract` and `Classify` (`internal/ai/inline.go`). The demo seed (`backend/seed/seed.sql`). The production deploy stack (`deploy/`). Hand-off guides for Nouman and Humza. 21 commits |
| **Humza Masoom**      | `humzamasoom`    | Repo, Docker Compose, CI, merging to `main`                                        | Project scaffold and the six planning documents. Docker Compose and CI. Switched auth to Firebase. Local test script. Connected the web screens to the real API. Phone sign-in on the web. One-question-at-a-time onboarding screen. English/Urdu language switch. New landing page, logo, header and footer. Worker profile editing. Map picker for the job location (OpenStreetMap). Reduced Jev calls in onboarding. Committed the Urdu prompt audio. Merged the branches. 26 commits                                               |
| **Abdul Muizz**       | `a-muizz28`      | Frontend: `web/`                                                                   | The frontend screens for both roles (branch `ui`): signed-in app shell, find-a-worker flow, job chat, green palette, audit and review fixes. 6 commits, about 3,400 lines                                                                                                                                                                                                                                                                                                                                                              |
| **Nouman Zahid** (me) | `NoumanZahid-85` | AI: `backend/internal/{ai,tasks}`, seed, README, prompt audio                      | The AI service (`gateway.go`, `urdu.go`): the only code that calls Vercel AI Gateway, with speech-to-text, LLM, Jev decisions and text-to-speech, each with a fallback model, cache, timeout and output validation. The four queue handlers (`tasks.go`). 17 unit tests for both. Speech-to-text and text-to-speech model comparison for Urdu. 1 commit, 1,087 lines (`9a6d680`)                                                                                                                                                       |

Tasks that were planned for me but done by others, because time ran out: `Extract` and `Classify` and the seed data (Abdullah), the README (Humza and Abdul Muizz), committing the prompt audio (Humza).

AI coding assistants were used by the team: Claude Code (named in commit messages) and OpenAI Codex (for the frontend design package, per the repo README). The six planning documents in `docs/` were written to be given to the assistant before building.

### Timeline of the day

Planned deadlines: Figma 12:00, full flow working 14:30, freeze 15:00, submit 15:30 (hard limit 15:45).


| Time           | What happened (from commits)                                                                                                              |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| 10:53          | Humza: project scaffold, the six planning documents, Compose, CI                                                                          |
| 11:06          | Humza: auth switched to Firebase                                                                                                          |
| 11:10 to 11:46 | Abdullah: platform, identity, booking state machine with race test, workers, jobs, reviews, end-to-end test, deploy stack, queue contract |
| 11:57          | Humza: plan changed to four-question onboarding and Urdu speech for the worker                                                            |
| 12:08          | Abdullah: phone sign-in                                                                                                                   |
| 13:03          | Nouman: AI service and the four queue handlers (`9a6d680`)                                                                                |
| 13:09 to 13:27 | Abdullah: `Extract`, `Classify`, seed data, rate limits, integration notes                                                                |
| 13:31          | Abdul Muizz: frontend screens for both roles (branch `ui`)                                                                                |
| 13:47 to 15:27 | Humza: web connected to the real API, language switch, new landing page, profile editing, map picker                                      |
| 15:28 to 15:43 | Abdullah: last UI fixes, separate sign-in doors for workers and customers (last commit 15:43)                                             |

### Branches and pull requests


| Branch           | Owner        | What it carried                                                                    |
| ------------------ | -------------- | ------------------------------------------------------------------------------------ |
| `main`           | Humza merges | The final code                                                                     |
| `abdullah`       | Abdullah     | Backend; merged by PRs #1 to #5                                                    |
| `nouman`         | Nouman       | AI service and queue handlers; merged into `abdullah`, then into `main` with PR #2 |
| `ui`             | Abdul Muizz  | First frontend screens                                                             |
| `ui-consistency` | Humza        | New landing page, restyled app, profile editing                                    |
| `map-picker`     | Humza        | Job location on a map                                                              |

Rule from the plan: short branches, small PRs, only Humza merges to `main`, everyone stays inside their own folders.

## 3. Screenshots

Taken on 10 November 2026 at phone width (390 px), except the first.

**How these were taken.** The first five are from the live site. The rest are from the app's built-in demo mode (`VITE_MOCK_API=true`), which runs the same screens on fictional data in the browser. I could not capture the signed-in screens from the live site: its sign-in showed a reCAPTCHA picture challenge to the automated browser. In demo mode the sign-in accepts the same demo number and code (`3000000001`, `123456`) without contacting Firebase. Screens in demo mode carry a "Demo mode" note at the bottom.

### Public pages and sign-in (live site)


| Landing (desktop)                       | Landing (phone)                        | Role pick                         |
| ----------------------------------------- | ---------------------------------------- | ----------------------------------- |
| ![](screenshots/01-landing-desktop.png) | ![](screenshots/02-landing-mobile.png) | ![](screenshots/03-role-pick.png) |


| Worker sign-in                              | Number typed                                       | Code step (demo mode)                      |
| --------------------------------------------- | ---------------------------------------------------- | -------------------------------------------- |
| ![](screenshots/04-worker-signin-phone.png) | ![](screenshots/05-worker-signin-number-typed.png) | ![](screenshots/06-worker-signin-code.png) |

### Worker side (demo mode, Urdu, right to left)


| Voice onboarding: question 1 of 4                | "Understanding"                                      | Understood, next question                         |
| -------------------------------------------------- | ------------------------------------------------------ | --------------------------------------------------- |
| ![](screenshots/10-worker-onboard-question1.png) | ![](screenshots/11-worker-onboard-understanding.png) | ![](screenshots/12-worker-onboard-understood.png) |


| Location                                | Confirm the profile                    | Worker home (online toggle)         |
| ----------------------------------------- | ---------------------------------------- | ------------------------------------- |
| ![](screenshots/13-worker-location.png) | ![](screenshots/14-worker-confirm.png) | ![](screenshots/15-worker-home.png) |


| My jobs                                 | Own profile                                |
| ----------------------------------------- | -------------------------------------------- |
| ![](screenshots/16-worker-app-jobs.png) | ![](screenshots/16-worker-app-profile.png) |

### Customer side (demo mode, English)


| Sign-in                                 | Dashboard                                  | Describe the problem                              |
| ----------------------------------------- | -------------------------------------------- | --------------------------------------------------- |
| ![](screenshots/20-customer-signin.png) | ![](screenshots/21-customer-dashboard.png) | ![](screenshots/22-customer-describe-problem.png) |


| Check the trade and price range              | Pick the location on a map                | Workers nearby                               |
| ---------------------------------------------- | ------------------------------------------- | ---------------------------------------------- |
| ![](screenshots/23-customer-check-trade.png) | ![](screenshots/24-customer-location.png) | ![](screenshots/25-customer-worker-list.png) |


| Worker profile (trust card)                     | Job chat with a price offer               | Offer a price (number pad)                  |
| ------------------------------------------------- | ------------------------------------------- | --------------------------------------------- |
| ![](screenshots/26-customer-worker-profile.png) | ![](screenshots/27-customer-job-chat.png) | ![](screenshots/28-customer-make-offer.png) |


| Job status                                  | Jobs list                             | Leave a review                          |
| --------------------------------------------- | --------------------------------------- | ----------------------------------------- |
| ![](screenshots/29-customer-job-status.png) | ![](screenshots/30-customer-jobs.png) | ![](screenshots/31-customer-review.png) |

### Design rules the screens follow

- **Worker side is Urdu, right to left**, font Noto Nastaliq Urdu. Every piece of information has three layers: an icon and a number first, a few words of Urdu second, a speaker button third.
- **Nothing speaks by itself** except the onboarding questions and the ringtone for a new job. Everything else plays only when the speaker button is tapped.
- **Customer side is English or Urdu.** The customer can write or speak in Urdu, Roman Urdu or English. The language choice is saved per role in the browser.
- One main action per screen. Touch targets at least 56 px. Mobile first, max width 480 px on desktop.
- Colours: green means go, accept or online; red means cancel, reject or offline; amber means waiting.

### How voice onboarding works (screens 10 to 14)

1. The screen plays a recorded Urdu question. The worker taps the mic, speaks, and taps again.
2. The clip is sent to `POST /api/workers/onboard` with the question's name (`name`, `trade`, `area`, `years_exp`). The next question plays while the answer is processed, so the worker rarely waits.
3. The screen shows what it understood and moves on. One answer can fill several fields ("Main Akram hoon, plumber"); a question already answered is skipped.
4. If an answer is not understood, the screen shows what it heard and asks once more. After that the trade falls back to the five trade icons; name, area and years are left empty.
5. The microphone is opened once when the screen opens and kept open, because opening it on every tap cut off the first words.
6. Location, then a confirm screen: one tap saves the profile (`PUT /api/workers/me`). The trade answer's audio becomes the worker's intro audio.

## 4. Architecture

![Architecture](screenshots/architecture-diagram.jpeg)

- **One Go binary, two modes.** `app api` serves HTTP and WebSocket. `app worker` runs the queued AI jobs. Both use the same services, so a result written by the worker goes through the same checks and pushes the same events.
- **Modular monolith.** `internal/{identity,workers,jobs,reviews,problems,ai}`. Each module exposes one `Service` and owns its tables. Modules call each other only through `Service` methods.
- **Postgres** holds all records. **Redis** does four jobs: the task queue, pub/sub for WebSocket events, the AI result cache and rate-limit counters.
- **Outside services:** Firebase Auth (sign-in), Vercel AI Gateway (all AI models), OpenStreetMap (map tiles).
- **No Python service.** The models are hosted, so the AI work is plain HTTP calls from Go.
- **No agent framework.** Every step is known in advance: transcribe, Jev decides, the LLM runs only when Jev says so.
- **No Kafka.** Nothing needs replay. Redis pub/sub plus "refetch after reconnect" covers delivery.

## 5. Tech stack


| Layer                       | What we used                                                                                                                                                                                                                                                                   |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Frontend                    | React 19, Vite 8, TypeScript 7, plain CSS (no Tailwind, although the plan named it). Own small router (`web/src/lib/router.tsx`), no routing library. Leaflet 1.9 with OpenStreetMap tiles for the map picker. Lucide icons. Fonts: Inter and Noto Nastaliq Urdu. Lint: oxlint |
| Auth                        | Firebase Auth (JS SDK 13 in the browser, Admin Go SDK v4 on the server). Phone number with SMS code, Google, and email with password                                                                                                                                           |
| Backend                     | Go 1.27, standard library `net/http` (no web framework). `pgx` v5 for Postgres with hand-written SQL (no sqlc, although the plan named it). `go-redis` v9. `coder/websocket`                                                                                                   |
| Queue                       | `asynq` 0.26 on Redis, with the `asynqmon` dashboard                                                                                                                                                                                                                           |
| Database                    | Postgres 16                                                                                                                                                                                                                                                                    |
| Cache, pub/sub, rate limits | Redis 7                                                                                                                                                                                                                                                                        |
| AI                          | Vercel AI Gateway, called only from `backend/internal/ai`                                                                                                                                                                                                                      |
| Media                       | Audio and photo files on a Docker volume, served at `/media/*`                                                                                                                                                                                                                 |
| Run                         | Docker Compose: `web`, `api`, `worker`, `postgres`, `redis`, `asynqmon`                                                                                                                                                                                                        |
| Production                  | Oracle cloud server (ARM64), Docker Compose in `deploy/`, nginx serving the built site and proxying `/api`, `/api/ws` and `/media`, behind a Cloudflare tunnel                                                                                                                 |
| CI                          | GitHub Actions, manual trigger only: `go vet`, `go test`, lint, web build                                                                                                                                                                                                      |

## 6. AI: models and how they are used

All models go through Vercel AI Gateway with one key (`AI_GATEWAY_API_KEY`).


| Step                        | Main model                                         | Fallback                       | Gateway endpoint             |
| ----------------------------- | ---------------------------------------------------- | -------------------------------- | ------------------------------ |
| Urdu speech to text         | `microsoft/mai-transcribe-2` (with locale `ur-PK`) | `google/gemini-3.5-transcribe` | `/v4/ai/transcription-model` |
| Yes/no decisions            | `typesafe-ai/jev`                                  | `openai/gpt-6-luna-decisions`  | `/v1/decisions`              |
| Extraction and Urdu writing | `anthropic/claude-haiku-5.5` (thinking off)        | `openai/gpt-6-luna`            | `/v1/chat/completions`       |
| Urdu text to speech         | `elevenlabs/eleven-v4`                             | `google/gemini-3.8-flash-tts`  | `/v4/ai/speech-model`        |

### What Jev is

Jev is a decision model. It does not write text. You send it a text and yes/no questions, and it returns a probability from 0 to 1 for each question. Our code treats 0.6 or more as "yes". Below that, the app does not trust the answer and the screen shows a picker instead.

Where Jev is used:

- **Is a price stated in this voice note?** Only on "yes" is the LLM asked to extract the amount. Measured: "پندرہ سو" 0.98; "kitne paise lenge?" 0.08; "G-10 mein ghar hai, 5 baje aa jao" 0.03.
- **Which trade?** Five yes/no questions, one per trade; the highest wins if it reaches 0.6.
- **Review tags:** `on_time`, `fair_price`, `clean_work`, `polite`, `abusive_or_spam`. Five questions in one call. A review tagged `abusive_or_spam` is hidden from lists and ratings.

### Inline AI (the user waits on the screen, about 2 to 6 s)


| Route                       | Function      | Steps                                                                                                                                                                |
| ----------------------------- | --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `POST /api/workers/onboard` | `ai.Extract`  | Speech to text, then the LLM fills name, trade, skills, area, years as JSON. Jev confirms the trade. Islamabad sectors are normalised ("جی ٹین" becomes "G-10") |
| `POST /api/problems`        | `ai.Classify` | Optional speech to text and photo. The LLM writes trade, urgency and a one-line description. Jev confirms the trade                                                  |

If inline AI fails or is off, the route still returns 200 with empty fields and the screen shows the trade icons to tap. The user is never blocked.

### Queued AI (runs in the background; the result arrives later)


| Task             | Trigger                  | Steps                                                                                                            | Fields filled                                   |
| ------------------ | -------------------------- | ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------- |
| `transcribe`     | Voice message sent       | Speech to text, Jev "has price?", LLM extracts the amount only on yes                                            | `messages.body`, `messages.suggested_price`     |
| `message speech` | Customer types a message | LLM rewrites it in short Urdu script, then text to speech                                                        | `messages.body_ur`, `messages.speech_url`       |
| `job brief`      | Job created              | LLM writes a short Urdu alert (trade, urgency, distance), then text to speech                                    | `jobs.brief_ur`, `jobs.brief_audio_url`         |
| `review analyze` | Review saved             | Optional speech to text, Jev tags, then the LLM writes a fresh trust summary from the worker's 20 newest reviews | `reviews.tags`, `worker_profiles.trust_summary` |

Queue settings: 5 tasks at once, 3 retries with backoff, 60 s per task, task ID `type:id` so the same task is never queued twice.

### Safety nets in the AI code

- **Fallback model per step:** an error or an invalid answer moves to the second model.
- **Validation:** Urdu output must be mostly Urdu script; a price must be valid JSON with a sensible amount. Invalid output counts as a failure.
- **Timeout:** 15 s per AI call.
- **Cache:** results are stored in Redis for 24 hours, keyed by a hash of the input, so a retry never pays twice.
- **Partial save:** if text to speech fails, the Urdu text is saved anyway and the retry adds the audio. If the price step fails, the transcript is still saved.
- **No pointless retries:** a bad payload or a deleted row is marked "never retry".
- **Daily budget:** 3,000 gateway calls a day for the whole app (`AI_DAILY_CAP`); after that the AI fields stay empty.
- **Hindi script fix:** if speech to text returns Devanagari, the LLM rewrites it in Urdu script.

### What we measured

- Same Urdu clip, one run each: the Microsoft model returned Urdu script in 1.9 s. The Gemini model took 3.6 s and returned Hindi script. So the browser records WAV, which the Microsoft model accepts (it rejects webm).
- One Jev call: about 1.3 s, 305 input and 21 output tokens, about $0.0000128.
- Asking five review questions in one Jev call gave scores within 0.03 of asking them separately.
- The wording of a Jev question matters: one early wording scored an honest complaint 0.63 as "abusive"; the reworded question scored it 0.09.

## 6b. Caching, uploads and design reasons

**What Redis caches, and what it never caches**


| Cached                                                   | How long                                                                               |
| ---------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| AI results, keyed by a hash of the input                 | 24 hours                                                                               |
| Price range per trade                                    | 5 minutes                                                                              |
| Worker rating and jobs-done numbers                      | 10 minutes, deleted when a new review is saved                                         |
| **Never cached:** worker availability and search results | A stale "free" worker would break the booking guarantee, so these always read Postgres |

**Uploads:** at most 20 MB per request. Audio: `.wav .webm .ogg .mp3 .m4a .mp4`. Photos: `.jpg .jpeg .png .webp`. The browser records WAV because the main speech model rejects webm.

**Why we chose what we chose (answers for Q&A)**

- *Why Go and not Python for the AI?* The models are hosted. The AI work is a few HTTP calls, so a separate Python service would only add a network hop.
- *Why no agents?* Every step is known in advance, so a fixed pipeline with Jev as the router is faster, cheaper and testable. Agents fit when the next step is unknown.
- *Why no Kafka?* Nothing needs replay or many consumers. We would add a Kafka-style log (Redpanda) when a second consumer, such as analytics, appears.
- *Why asynq?* It is the Go equivalent of Celery: Redis-backed, retries, a dashboard.
- *Why is the booking rule in the database and not in Go?* Two requests racing in Go can both pass a check. A unique index cannot be raced.
- *What stops working without the AI?* A worker who cannot type cannot make a profile, negotiate or be reviewed in their own words. A customer must know which trade fits the problem. Voice notes stay audio with no transcript, and prices in them never become a one-tap offer. Reviews stay raw audio with no tags or summary.

## 7. Database

Five tables and one view (`backend/db/schema.sql`):


| Table                      | Holds                                                                                                               | Owner module |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------- |
| `users`                    | Firebase user id, email, phone, role (`worker` or `customer`), name, photo                                          | identity     |
| `worker_profiles`          | Trade, skills, years, area, latitude and longitude, intro audio, online flag, trust summary                         | workers      |
| `jobs`                     | Customer, worker, trade, description, urgency, location, status, agreed price, Urdu brief and its audio             | jobs         |
| `messages`                 | Chat messages: `text`, `voice` or `offer`; transcript, Urdu text and audio, suggested price, offer price and status | jobs         |
| `reviews`                  | One per job: rating 1 to 5, text, audio, tags                                                                       | reviews      |
| `available_workers` (view) | Workers who are online and have no `confirmed` or `in_progress` job. Availability is derived, never stored          | workers      |

Job states: `requested` → `confirmed` → `in_progress` → `completed`. Any state before `completed` can go to `cancelled`.

**The booking guarantee** is one line of SQL:

```sql
CREATE UNIQUE INDEX one_active_job_per_worker
  ON jobs (worker_id) WHERE status IN ('confirmed','in_progress');
```

If two customers accept offers from the same worker at the same moment, the database lets one through. The other gets `409 worker_busy`. A test fires two accepts at once and checks that exactly one confirms (`backend/internal/jobs/booking_test.go`).

Distance is the Haversine formula in SQL (no PostGIS).

`backend/db/migrations/002_phone_and_urdu_speech.sql` adds the columns that came later in the day (`users.phone`, `jobs.brief_ur`, `messages.body_ur`, `messages.speech_url`) to a database created from an older schema. It is safe to run twice.

## 8. API

Base path `/api`. Every call carries `Authorization: Bearer <Firebase ID token>`. Errors are `{ "error": "code", "message": "..." }`.


| Method | Path                                    | Does                                                                   |
| -------- | ----------------------------------------- | ------------------------------------------------------------------------ |
| POST   | `/auth/session`                         | Called once after every sign-in. Creates the user on first call        |
| GET    | `/me`                                   | Current user and worker card                                           |
| POST   | `/workers/onboard`                      | One spoken onboarding answer → what the AI understood (saves nothing) |
| PUT    | `/workers/me`                           | Save the worker profile                                                |
| PUT    | `/workers/me/online`                    | Online/offline toggle                                                  |
| GET    | `/workers?trade=&lat=&lng=&radius_km=5` | Available workers, nearest first                                       |
| GET    | `/workers/{id}`                         | Profile with reviews and price range                                   |
| POST   | `/problems`                             | Text, audio and/or photo → trade, urgency, description, price range   |
| POST   | `/jobs`                                 | Create a job request                                                   |
| GET    | `/jobs`, `/jobs/{id}`                   | My jobs; one job with its messages                                     |
| POST   | `/jobs/{id}/messages`                   | Text, offer, or voice note                                             |
| POST   | `/jobs/{id}/offers/{message_id}/accept` | Confirms the job, or `409 worker_busy`                                 |
| POST   | `/jobs/{id}/offers/{message_id}/reject` | Rejects the offer                                                      |
| POST   | `/jobs/{id}/status`                     | `in_progress`, `completed` or `cancelled`                              |
| POST   | `/jobs/{id}/review`                     | Rating, optional audio, optional text                                  |
| GET    | `/ws?token=`                            | WebSocket, server to client only                                       |
| GET    | `/healthz`                              | Postgres and Redis status                                              |

WebSocket events: `job.created`, `job.updated`, `message.created`, `message.updated`. The API publishes them through Redis, so the worker process can push events too.

**Rate limits** (Redis token buckets): 1,200 a minute per IP; 120 sign-ins a minute per IP; 300 writes a minute per user; 30 chat messages a minute per user; 50 new jobs per 10 minutes per user; 20 inline AI calls a minute per user.

**Error codes the screens handle**


| Status | Code                                                                                         | Meaning                                                         |
| -------- | ---------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| 400    | `bad_request`                                                                                | Bad input, or an upload too large or of the wrong type          |
| 401    | `unauthorized`, `no_session`                                                                 | Token missing or expired; or `/auth/session` was not called yet |
| 403    | `forbidden`, `worker_only`, `own_offer`                                                      | Wrong role, or accepting your own offer                         |
| 404    | `not_found`                                                                                  | Also returned for other people's jobs                           |
| 409    | `worker_busy`                                                                                | The worker took another job first (the booking guarantee)       |
| 409    | `offer_not_pending`, `invalid_status`, `job_closed`, `job_not_completed`, `already_reviewed` | The state moved on: reload                                      |
| 429    | `rate_limited`                                                                               | Too many requests; wait for `Retry-After` seconds               |

**Ownership rules:** only the two parties of a job can read it or post to it. Only the other party can accept an offer. Only the worker moves a job forward. Only the job's customer can review, and only when it is completed.

## 9. Sign-in

- The browser signs in with Firebase and sends the ID token on every call. The API verifies the token and maps the Firebase user id to a row in `users`.
- Workers: phone number and SMS code (digits only, so no reading), or Google. Customers: phone, Google, or email and password.
- The Firebase plan sends at most 10 SMS a day, so the demo used **Firebase test numbers** (no SMS sent): worker `+92 300 0000001`, customer `+92 300 0000002`, code `123456` for both.
- The final code turns off reCAPTCHA (`web/src/lib/firebase.ts`), so Firebase accepts only the registered test numbers.
- Local testing without Firebase: `AUTH_DEV=1` on the API and `Authorization: Bearer dev:<any-name>`.

**Firebase setup (done once in the Firebase console):** create the project; enable the Phone, Google and Email/Password providers; add the two test numbers with code `123456` (Authentication → Sign-in method → Phone); copy the web config into `.env`; add the site's domain under Authentication → Settings → Authorized domains, or Google sign-in is refused.

## 10. Repository map

```
backend/
  cmd/app/main.go            one binary: `app api` and `app worker`; wires all modules
  db/schema.sql              tables, the booking index, the available_workers view
  internal/
    ai/                      Vercel AI Gateway client (Nouman; inline.go by Abdullah)
      gateway.go             HTTP calls, fallback, cache, timeout, daily budget, Jev
      urdu.go                Transcribe, HasPrice, Price, UrduMessage, JobBrief, ReviewTags, TrustSummary, Speak
      inline.go              Extract (onboarding) and Classify (problem)
    tasks/tasks.go           the four queue handlers (Nouman)
    identity/                Firebase token verification, users table
    workers/                 profiles, onboarding, nearby search
    jobs/                    jobs, messages, offers, booking state machine
    reviews/                 reviews and rating stats
    problems/                POST /api/problems
    platform/                auth middleware, CORS, uploads, rate limit, queue client, WebSocket hub
  seed/seed.sql              demo data
web/
  src/screens/               Landing, Auth, Onboarding, WorkerHome, Find, Job, Jobs, Profile, Dashboard
  src/lib/                   api client, mock backend, Firebase, audio recording, i18n, router
  src/contract.ts            the API types, kept in step with docs/2-TRD.md
  public/audio/              14 Urdu prompts (role, signin, ask-name, ask-trade, ask-area, ask-years,
                             again, ok, confirm, done, error, home, chat, offer) and ring.wav
deploy/                      production Docker Compose, nginx config, web image
docs/                        the six planning documents (PRD, TRD, app flow, UI brief, schema, plan)
.github/workflows/ci.yml     manual CI
```

Size: about 9,100 lines of Go (code and tests) and about 2,100 lines in the web screens. 131 Go test functions.

## 11. Run it

```bash
cp .env.example .env        # put the AI Gateway key and the Firebase values in .env
docker compose up --build
docker compose exec -T postgres psql -U roshan -d roshan < backend/seed/seed.sql   # demo data, once
```


| What            | Where                             |
| ----------------- | ----------------------------------- |
| Web app         | http://localhost:5173             |
| API             | http://localhost:8080/api/healthz |
| Queue dashboard | http://localhost:8081             |

- The API will not start without `FIREBASE_PROJECT_ID` (or `AUTH_DEV=1` for local testing).
- Without `AI_GATEWAY_API_KEY` the app still runs: inline AI is off, the screens show pickers, and queued AI tasks wait.
- To see the screens with no backend at all: in `web/`, run `npm ci` and then `VITE_MOCK_API=true npx vite`.
- Tests: `backend/scripts/test-local.sh` runs `go vet` and the whole Go suite with the race detector against throwaway Postgres and Redis containers. It needs only Docker.
- After changing `schema.sql`, reset the database with `docker compose down -v`.

### Environment variables (repo-root `.env`, never committed)


| Variable                                                                                                 | Used for                                                                |
| ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| `AI_GATEWAY_API_KEY`                                                                                     | Vercel AI Gateway key. Empty = AI off, the app still works with pickers |
| `STT_MODEL`, `STT_FALLBACK_MODEL`, `LLM_MODEL`, `LLM_FALLBACK_MODEL`, `JEV_MODEL`, `TTS_MODEL`           | Override the model names (defaults are the ones in section 6)           |
| `AI_DAILY_CAP`                                                                                           | Gateway calls allowed per day for the whole app (default 3000)          |
| `FIREBASE_PROJECT_ID`                                                                                    | API: verify Firebase tokens                                             |
| `VITE_FIREBASE_API_KEY`, `VITE_FIREBASE_AUTH_DOMAIN`, `VITE_FIREBASE_PROJECT_ID`, `VITE_FIREBASE_APP_ID` | Browser: Firebase web config                                            |
| `GOOGLE_APPLICATION_CREDENTIALS`                                                                         | Optional service-account file                                           |
| `CORS_ORIGINS`                                                                                           | Extra browser origins allowed by the API                                |
| `AUTH_DEV`                                                                                               | `1` accepts `dev:<name>` tokens. Local testing only, never on a server  |
| `VITE_DEV_AUTH`, `VITE_MOCK_API`                                                                         | Web: dev sign-in, or the in-browser mock backend                        |
| `DATABASE_URL`, `REDIS_ADDR`, `MEDIA_DIR`, `PORT`                                                        | Set by Docker Compose                                                   |

### The tests that matter most


| Test                                                                      | Proves                                                                                                                                                                                                                                                                                                                     |
| --------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `TestRaceTwoAcceptsSameWorker` (`jobs/booking_test.go`)                   | Two accepts at the same moment for one worker: exactly one confirms, the other gets 409                                                                                                                                                                                                                                    |
| `TestFullFlow` (`cmd/app/e2e_test.go`)                                    | The whole flow through the real router, Postgres and Redis: sign in both roles, save the worker profile, go online, describe the problem, search, create a job, chat (text and voice), offer, accept, the worker disappears from search, complete, review (and a second review is refused), WebSocket events, queued tasks |
| `TestRouteRateLimits` (`cmd/app/ratelimit_test.go`)                       | Each limit is on the right route                                                                                                                                                                                                                                                                                           |
| `internal/ai/ai_test.go`, `internal/tasks/tasks_test.go` (mine, 17 tests) | Fallback models, Urdu locale, Hindi-script fix, price parsing, Jev threshold and refusals, text-to-speech format, partial saves, no retry on bad input                                                                                                                                                                     |
| `internal/ai/inline_test.go`                                              | Onboarding extraction and problem classification                                                                                                                                                                                                                                                                           |

### Production deploy (as on the day)

- Oracle cloud server (ARM64), clone at `~/hackathon`, secrets in `~/hackathon/deploy/.env` (not in git).
- Redeploy: `ssh oracle 'cd ~/hackathon && git pull && docker compose -f deploy/docker-compose.yml up -d --build'`
- Only `127.0.0.1:9200` (nginx: site plus `/api`) and `127.0.0.1:9201` (API) reach the host. A Cloudflare tunnel maps them to `https://hackathon.taciturn2021.dpdns.org` and `https://hackathon-api.taciturn2021.dpdns.org`.
- nginx re-resolves the API through Docker's DNS on every request, so recreating the API container does not cause 502 errors.
- The browser microphone needs HTTPS, which is why the phone demo went through the tunnel.

## 12. Seeded and mocked

- **Seed data:** 40 workers (8 per trade, across Islamabad and Rawalpindi), 8 customers, 95 past jobs (completed or cancelled) and 90 reviews. All fictional. Their tags and trust summaries are written by the seed, not by the AI. Nobody can sign in as a seeded account.
- **Demo location:** G-10, Islamabad (33.6844, 73.0145), where the seeded workers are.
- **Spoken prompts** in `web/public/audio/`: generated once with the text-to-speech model, not recorded by a person.
- **Mock mode:** `VITE_MOCK_API=true` runs the screens on an in-browser mock backend. Off by default.

## 13. Known limits and deliberate shortcuts

- The web app polls every 4 to 5 seconds for new messages and job changes. The API has a WebSocket, but the web app does not use it yet.
- Urdu speech quality was checked with generated speech, not with many real speakers and accents.
- Media files are on the server's disk. Move to S3-style storage when the API runs on more than one machine.
- Distance uses Haversine in SQL. Move to a PostGIS index for very large numbers of workers.
- The price range is per trade only, not per kind of job.
- AI-written text (problem summaries, trust summaries) is English only. Spoken prompts are Urdu only.
- Not built on purpose: payments, CNIC verification, scheduling for later, live GPS tracking, an admin panel, a native app.
- I did not run the full Docker stack myself on the day; the queue handlers and AI service were checked with unit tests and separate live calls to the gateway.

## 14. Where to read more


| File                                                      | What it has                                                                                                                    |
| ----------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `docs/1-PRD.md` to `docs/6-IMPLEMENTATION_PLAN.md`        | The six planning documents: product, technical design and API contract, screens, UI brief, schema, plan and time table         |
| `HUMZA-INTEGRATE.md`                                      | Frontend integration notes: errors, limits, uploads, AI timing, seed, deploy                                                   |
| `backend/HANDOFF-NOUMAN.md`                               | How the AI and queue code plugs into the backend                                                                               |
| `D:\Hackathons\nouman-ai-guide.html`                      | My own study guide for the AI part: workflows step by step, Jev with real examples, about 58 edge cases, 22 practice questions |
| `backend/internal/ai/speechlab/`                          | My local speech test tool (not committed)                                                                                      |
| `D:\Hackathons\hackathon-roshan\Roshan Rozgar final.pptx` | The final presentation (local, not committed)                                                                                  |
| `D:\Hackathons\hackathon-roshan\Images\`                  | The architecture diagram and a frontend picture (local, not committed)                                                         |
