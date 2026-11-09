# Sidecar: build plan

**One line:** Ask the talk anything, live. The people stuck on the same thing become your hallway track.

**Team:** 3 people. **Build:** ~12:15 → 3:15 (3h). **Demo prep:** 3:15 → 5:00. **Hard deadline:** 5:30 PM MT (we target 5:00).

---

## 1. The product, end to end (a complete journey, not a feature)

| Phase | What the attendee does | Role in the demo |
|---|---|---|
| **0. Onboard** | 60-second profile: name, role, years, company type, **"what I'm working on right now"**, **"what I want to get out of this conference"**, open-to-meet toggle | Supporting (10s) |
| **1. Before: My Schedule** | Search talks in plain English ("cutting observability costs"), see "recommended for you" (from the profile), add to My Schedule. Every talk has a QR code / join link | Supporting (10s) |
| **2. During: Live Talk** ⭐ | Scan QR → live captions → **ask anything privately**, answered from what the speaker just said → ⭐ star a moment → follow a topic | **HERO** |
| **3. Connect** ⭐ | After the talk: **blind profiles** of people who were stuck on the same thing → mutual "want to meet" → reveal + huddle spot | **HERO** |
| **4. Across rooms** | "Room B is talking about your topic right now" alerts; answers that cite other talks | Wow moment (10s) |
| **5. After: My Recap** | Auto-built trip report: takeaways, people met and why, open questions, action items | Payoff (15s) |
| **6. Speaker view** | Per-minute confusion timeline + top unanswered questions | Flash (5–10s) |

Phases 0–1 are small in the demo, but they're **smart**: they create the personal profile that everything else (schedule, matching, alerts, recap) uses.

---

## 1b. The opening experience, screen by screen

### Screen 1: "Tell Sidecar about you" (≈60 seconds)
| Field | Why it matters |
|---|---|
| Name, role, years of experience, company type/size | Blind-profile card + matching |
| **GitHub username (optional)** → we read public repos, languages, topics, READMEs and write a 2-line "what you build" summary you can edit | Instant, real personalization with no typing |
| **"What are you working on / stuck on right now?"** (free text) | The strongest signal for talks *and* people |
| **Why are you here?** pick 1–2: *Solve a specific problem · Learn a new area · Meet peers · Hire · Find a job · Evaluate tools* | Changes how the schedule is ranked (below) |
| **Level** per topic chip (e.g. Kubernetes: advanced, eBPF: new to it) | Avoids too-basic / too-advanced talks |
| **Pace:** Chill · Balanced · Packed | How many slots to leave open (veterans' advice: leave ~50% free) |
| Open to meet people? (on/off) | Privacy |

### Screen 2: "Your day, planned" (the smart scheduler)
One tap, **Plan my day**, builds a full schedule. Every pick shows *why*:
> **10:30 · OTel at Scale (Room B)** ✅
> *Matches what you're stuck on: "cutting tracing costs" · Advanced, your level · 14 people with similar problems are going*
> ↔ 2 alternatives in this slot · ✕ remove

What makes it smart (not just "search and add"):
1. **Relevance to *you*:** each talk's abstract is matched by meaning against your "working on" + GitHub summary + goals.
2. **Goal-aware ranking:**
   - *Solve a problem*: talks closest to your problem, even if the title doesn't say it.
   - *Learn*: beginner/intro talks in your "new to it" topics.
   - *Meet peers / Hire / Job*: favors talks where **many people like you** are going (from the schedule graph), plus hallway breaks after them.
   - *Evaluate tools*: favors sponsor/vendor talks and adds booth time.
3. **Level fit:** down-ranks talks below or far above your level.
4. **Variety:** doesn't stack four talks on the same topic (each pick lowers the score of very similar talks).
5. **Pace:** leaves open blocks for the hallway track, labeled ("☕ Free: 3 of your matches are free now too").
6. **Clashes resolved for you:** two great talks at the same time → it picks one for you live, and the other gets **"📼 Catch it in Sidecar"**. Because every room is transcribed, you can read and *ask that talk questions* later. This ties the scheduler to the hero feature.
7. **Social proof from the schedule graph:** "14 people with similar problems are going", "People like you also added…".
8. **Re-plans as you go:** the afternoon re-ranks from what you asked and starred this morning.

### Screen 3: Talk page (from schedule or QR)
Title, room, time, speaker, level, **"Why this is for you"**, **"Who's going: 6 people share your problem (blind)"**, and a **Join live** button that opens the Live Talk page (the hero).

### Build scope for the scheduler (Person B builds the ranking query ~45 min, Person C builds the screens ~45 min)
- **Must:** onboarding form (GitHub optional), Plan my day with "why" lines, swap alternatives, clash → "Catch it in Sidecar", talk page.
- **Stretch:** re-plan as you go, booth time for *Evaluate tools*.

---

## 1c. Personalization in Elasticsearch (more than just questions)

Everything you do updates **one interest profile** per person, and every ranking reads from it.

### Signals we collect
| Signal | Strength | Source |
|---|---|---|
| "Working on / stuck on" text + GitHub summary | Strong (explicit) | Onboarding |
| Goals, levels, pace | Strong (explicit) | Onboarding |
| Talks added / removed / swapped | Medium (removal = negative) | Scheduler |
| Talk pages opened, dwell time in a live talk | Weak → medium | Activity |
| ⭐ starred moments (the transcript text) | Strong | Live Talk |
| Applied/experience questions (+ extracted topic and problem) | Strongest | Live Talk |
| Topics followed, alerts opened vs ignored | Medium | Cross-room |
| Matches accepted / ignored | Medium (who you actually want to meet) | Connect |

### How it's stored (`profiles` index)
- `working_on`, `github_summary`, `goals_text`: **`semantic_text`** (meaning-based, embeddings handled by ES)
- `interest_topics`: `{topic: weight}`, recomputed from the `activity` index with a **terms aggregation, newer actions weighted higher**. Example: starred 3 eBPF moments today → eBPF weight goes up.
- `negative_topics`: from removed talks and ignored alerts
- `levels`, `goals`, `pace`

### How ES uses it (the "smart use of search" the judges score)
1. **Talk ranking = hybrid search + boosts:** a semantic query on talk abstracts using your profile text, **combined (RRF) with** keyword search on your top interest topics, then **`function_score`** boosts for level fit and goal type and a penalty for negative topics. Filtered by time slot.
2. **"People like you also added…":** a **`significant_terms` aggregation** over the schedules of your closest matches finds talks *unusually* popular among people like you (not just popular overall). Real collaborative filtering, natively in ES.
3. **"More like this":** a `more_like_this` query on the moments you starred finds related moments in *other* talks ("📼 Catch it in Sidecar" picks).
4. **Matching** reuses the same profile fields (see 2a).
5. **Recap** ties takeaways back to `working_on`.

**The one-line pitch:** *Every tap teaches Sidecar what you care about, and every screen (your schedule, your alerts, your matches, your recap) gets smarter from it.*

---

## 2. Design decisions (the open questions, answered)

### 2a. Is "asked the same question" enough reason to meet? No, so we don't match on that alone
"What is cardinality?" from two people is **not** a reason to network. So:

**Step 1: classify every question** (one cheap LLM call when it's asked):
- `definition`: "what is X?" → answered, **not used for matching**
- `applied`: "how do I do X in my setup?" / "how does this compare to Y?" → **used for matching**
- `experience`: "has anyone run this at scale?" → **strong matching signal**

The LLM also extracts the **topic** and the **underlying problem** ("reducing LLM latency", "tracing cost").

**Step 2: match score** (computed when the talk ends, and again whenever a new question comes in):
```
score = 0.45 × similarity(my applied/experience questions, theirs)     ← same problem, same moment
      + 0.25 × similarity(my "working on", their "working on")         ← same real-world context
      + 0.15 × shared moments (both ⭐ starred or asked within the same ~2 min, across talks)
      + 0.15 × schedule overlap, rarity-weighted                       ← same interests, before any talk
```
- **Schedule overlap is weighted by rarity:** sharing a niche talk only 40 people picked counts far more than sharing the keynote everyone picked (an IDF-style weight: `log(total attendees / attendees on that talk)`, summed over shared talks). In ES this is a `terms` query on `schedule.talk_ids` plus a per-talk count aggregation.
- **Two-layer graph:**
  1. **Before the event (from schedules):** a faint "interest" graph exists from minute one. That solves the cold start: you can see "12 people share 3+ niche talks with you" and "who's going to this talk," and a blind card can say *"You're both going to 'eBPF Deep Dive' and 'OTel at Scale'."*
  2. **During talks (from questions):** shared *problems* add **strong, bright edges** on top. Schedule = what you're curious about; questions = what you're actually stuck on.
- Before the talk, the Live Talk page shows **"5 people here also picked your 2 other niche talks"**: an icebreaker before the talk even starts.
- A match card is shown when **score ≥ threshold** and **either** ≥1 shared applied/experience question **or** a strong schedule overlap (≥3 rare shared talks) + similar "working on". Schedule-only matches are labeled **"Same interests"**; question-based ones are **"Same problem"** (ranked higher). Matches get stronger across the day.
- Two match types shown on the card:
  - **"Same problem"**: peers stuck on the same thing → swap notes.
  - **"Been there"**: their profile/"working on" says they've *done* what you're asking about → they can help.

### 2b. Blind profiles: yes, this is a great addition
It fixes three things at once: privacy, the pressure of cold intros (34% lack networking confidence), and wasted meetings.

**Card (blind):**
> 🔒 **Platform engineer · 6 yrs · fintech, 500–1000 people**
> Also stuck on: *reducing tracing costs* (asked at 14:02 in "OTel at Scale")
> Working on: *migrating 200 services to OpenTelemetry*
> Match: **Same problem** · 2 shared moments
> **[Want to meet]**

- No name, no photo, no exact company, until **both** tap "want to meet". Then reveal name + LinkedIn + a **suggested opener** ("You both asked how to sample traces without losing errors") + **where** to meet ("Coffee bar, Hall B, in 10 min", from the talk's room).
- Nobody sees your exact question text unless you allow it. They see the **topic** ("reducing tracing costs").

### 2c. Cross-room: how it actually works
**Every talk at the conference writes into one shared Elasticsearch index** (`transcripts`: talk_id, room, timestamp, text). Three things use that:

1. **Answers cite other rooms.** When you ask a question, we search *this talk's last few minutes first*, then *every other talk*. The answer can say: *"The speaker just said X. Also, in 'eBPF Deep Dive' (Room B, 10:40), they explained Y."*
2. **"Follow a topic" = Elasticsearch percolator.** When you follow "LLM latency", we store it as a *saved query*. Every new transcript chunk from *any* room is checked against all saved queries (`percolate`). On a hit → *"🔔 Room B is talking about LLM latency right now"*, with the snippet and a button to jump in.
3. **Your interests follow you.** Questions from talk 1 and talk 3 add up, so matches get better over the day.

**In the demo:** a second "room" is running with a pre-recorded talk replayed through the same ingest pipeline (a `replay` script feeding transcript chunks with real timing). Mid-demo, the alert fires from Room B. That makes it look and work like a real multi-room conference.

### 2d. Who transcribes?
**One device per room** (the speaker's or room laptop) captures audio and streams the transcript. Attendees' phones just **read** captions and **ask**. Cheap, simple, and it scales.
- Primary: OpenAI transcription on 5-second audio chunks (we have $50 credits/person).
- Fallback: browser Web Speech API (Chrome), plus the replay script for demo safety.

### 2e. Personalization and the "value" recap: how we track it
Every action is an event in an `activity` index: `{user, talk, type, ts, payload}`.

| Event | Captured when |
|---|---|
| `joined_talk` / `left_talk` | QR scan / page leave (gives dwell time) |
| `asked` (+ type, topic, answer) | Every question |
| `starred_moment` (+ last 30s of transcript) | ⭐ button |
| `followed_topic`, `alert_opened` | Topic follows/alerts |
| `match_shown`, `want_to_meet`, `met` | Connect flow |

**The recap is just a query plus one LLM call:**
- **Talks attended** (dwell time)
- **Your takeaways**: built from your ⭐ moments + your applied questions and their answers, **tied to your "working on" field** ("Relevant to your OTel migration: …")
- **People you met and why** (the shared problem), with **follow-up message drafts**
- **Still open**: questions you asked that the talk didn't answer → suggested talks tomorrow / people to ask
- **Value stats**: "7 questions answered · 12 moments saved · 3 people met · 2 cross-room alerts"
- **Export** as a trip report (Markdown) to send to your manager. That solves "prove the trip was worth it."

Personalization loop: the **interest profile** (profile + questions + stars) also re-ranks **"recommended talks"** in My Schedule. The more you use it, the better your schedule gets.

### 2f. Speaker view: keep it, but small
It's one aggregation (`date_histogram` of questions per minute + top clustered topics), so it's cheap. In the demo it's a **5–10 second flash**: "and the speaker sees exactly where the room got lost."

---

## 3. 3-minute demo storyline

| Time | Beat | On screen |
|---|---|---|
| 0:00–0:20 | **Hook.** "Networking is the #1 reason people go to conferences, and the worst served: tools match you on job titles. And you forget ~70% of a talk within a day." | Title + 2 stats |
| 0:20–0:35 | **Before.** My profile → search "cut observability costs" → add to schedule | Schedule page |
| 0:35–1:20 | **During (hero).** "Scan this QR, *this pitch is the talk*." Judges' phones show live captions. I ask "how do I sample traces without losing errors?" → answer quotes what I said 20s ago, with timestamp. 🔔 **Alert:** "Room B is talking about OTel cost right now" | Live talk page + a judge's phone |
| 1:20–2:00 | **Connect (hero).** The graph starts as faint **"same interests"** links from everyone's schedules, then **bright "same problem" edges light up live** as the judges' questions come in, forming clusters in real time. Tap a node → **blind profile** → "want to meet" → mutual → reveal + opener + "coffee bar, Hall B" | Graph + card |
| 2:00–2:20 | **After.** My Recap: takeaways tied to my work, people met, open questions, export. 5s speaker confusion timeline | Recap + timeline |
| 2:20–3:00 | **Under the hood.** ES (hybrid search, percolator, semantic matching, aggregations) · Dash0 SLO dashboard + "Agent0 found the bug we shipped at 2 PM" · Radar AI investigation · Gitar PRs with our review rules | 4 quick screenshots |

**Safety:** seeded data (fake attendees, questions, a second room replay) means the demo works even if no judge scans.

---

## 4. Architecture

```
[Room laptop: mic] → ingest-svc → OpenAI transcribe → ES: transcripts ──┐
                                                    └→ percolate → alerts (SSE)
[Attendee web app] ⇄ api-svc ⇄ ES: talks, profiles, questions, activity, follows
                         └→ OpenAI (answer, classify question, recap)
matcher-worker: on talk end / new question → scores → ES: matches
All on Kubernetes (Radar cluster, Argo deploys) · OTel → Dash0 · PRs → Gitar
```

**Stack (suggested, keep it boring):** Next.js (web, RUM) + one Node/TS or Python FastAPI `api` + `ingest` + `matcher` as small separate deployments (3 backend services = a better Radar/Dash0 story). Elastic Serverless (Elasticsearch project) with **`semantic_text`** (built-in embeddings, no vector code). OpenAI `gpt-4o-mini` + transcription model. Live updates via SSE or polling (no websockets needed).

**ES indices:** `talks` (catalog, semantic_text on title/abstract) · `transcripts` (talk_id, room, ts, text: semantic_text) · `questions` (user, talk, ts, text, type, topic, problem: semantic_text) · `profiles` (role, yrs, company_type, working_on: semantic_text, goals) · `follows` (percolator field) · `activity` · `matches`.

**Seed data:** ~30 conference-style talks (titles/abstracts/rooms/times), ~25 fake attendees with profiles + questions, 1 pre-recorded talk transcript for Room B replay.

---

## 5. Who does what (3 people, parallel from minute 1)

| | **Person A: Live talk** | **Person B: Data & intelligence** | **Person C: Platform & product shell** |
|---|---|---|---|
| 12:15–12:45 | Talk page skeleton, mic capture → 5s chunks | ES project, indices + mappings, seed scripts (talks, attendees) | Repo, Gitar app + `.gitar/review/` rules + CI, Radar env + starter repo + Argo, Dash0 + OTel in all services |
| 12:45–1:45 | Ingest svc → transcription → `transcripts`; live captions (SSE); ask box; ⭐ star | `ask` API (hybrid search: this talk → all talks, LLM answer with timestamps); question classifier; schedule search + recommend | Onboarding/profile page; My Schedule page + QR per talk; deploy all services to k8s |
| 1:45–2:45 | Follow-topic UI + alert toasts; Room B **replay script** | Percolator alerts; **matcher** (score formula) → `matches`; recap query + LLM | **Connect page**: live graph (react-force-graph / d3) + blind profile cards + mutual reveal |
| 2:45–3:15 | Speaker timeline view; polish live page | Recap page data + export; activity events everywhere | Integration, deploy, Dash0 dashboard + **SLOs**, Radar check |
| 3:15–5:00 | **All:** inject a real bug → find it with Agent0 + Radar → fix through a Gitar-reviewed PR · rehearse demo ×3 · README (setup, architecture, AI disclosure, per-track section) · screenshots · submit by 5:00 | | |

**Rules for the team:**
- **Every feature = its own branch + PR through Gitar.** No direct pushes to main. (The Gitar track scores this.)
- **OTel from the first commit**, not added at the end. (The Dash0 track scores this.)
- **Deploy to the Radar cluster by ~1:45**, not at 4. (The Radar track needs real usage.)
- Never log question or transcript text (privacy; becomes a Gitar review rule).
- Keep a `DEMO_NOTES.md` with screenshots of every sponsor-tool moment as it happens.

## 6. Open questions for the team (decide by ~12:30)
1. **Stack:** Next.js + Node/TS services, or Next.js + Python FastAPI? (Pick whatever the team is fastest in.)
2. **Who is Person A / B / C?**
3. **Seed talks:** use a real public schedule (e.g. KubeCon NA abstracts) or write ~30 realistic fake ones? Real is more convincing; check the license/terms first.
4. **Transcription:** OpenAI chunked transcription (better quality) vs browser Web Speech API (zero cost, Chrome only)? Plan: OpenAI primary, Web Speech fallback.
5. **Cluster:** Radar-provided namespace (join code at the Radar table) or our own? Plan: Radar's.
6. **GitHub onboarding:** keep it (great for personalization) or cut to free text if behind?
7. **Demo:** who presents, who drives the laptop, who watches the judges' phones?

## 7. Decisions log
| Time | Decision |
|---|---|
| 11:40 | Theme is *conferences in general*. KubeCon is the setting and the demo example |
| 11:55 | Hero = live "ask the talk" + matching on what people are stuck on. Name: **Sidecar** |
| 12:05 | Matching is never on definition questions alone; blind profiles with mutual reveal |
| 12:10 | Schedule overlap (rarity-weighted) = a faint "same interests" graph layer |
| 12:15 | Scheduler is smart (goal/level/pace-aware, explains every pick) but small in the demo |

## 8. Cut list (if behind at 2:45, cut from the bottom)
1. Speaker timeline → static screenshot
2. Recap export → on-screen only
3. "Been there" match type → "Same problem" only
4. Cross-room answer citations → keep alerts only
5. **Never cut:** live captions, ask, blind-profile matching, graph, one cross-room alert
