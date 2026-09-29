# SkillBridge AI — Project Plan
### Skill-Gap Matching Tool for Job Seekers

---

## 1. Overview

**SkillBridge AI** is a tool that compares a job seeker's skills against a set of target roles, scores how closely they match each one, and visually shows the "gap" — the specific skills standing between them and a strong fit. The current build is a self-contained, dependency-free prototype (single HTML file, vanilla JS, static job dataset). This plan covers both what's built and the path to a fuller, data-backed version.

---

## 2. Core Features

**Built (v1 — prototype):**
1. Chip-based skill input (type-to-add, quick-add suggestions)
2. Optional single-role filter, or compare against all roles at once
3. Match-percentage ranking across a job dataset
4. Expandable "bridge span" visualization — matched skills vs. missing skills per role
5. "Closest to finished" note highlighting the 1–2 skills that close the most gap

**Planned (v2+):**
6. Real job postings (via API or uploaded dataset) instead of the static mock list
7. Fuzzy / semantic skill matching (e.g. "JS" ≈ "JavaScript"), not just exact string match
8. Persisted skill profile across sessions
9. Per-gap-skill learning resource links
10. Resume import to auto-populate the skill list

---

## 3. System Architecture

**Current (v1):**
```
┌─────────────────────────────────────────────┐
│              index.html (browser)            │
│  ┌───────────┐  ┌────────────┐  ┌─────────┐  │
│  │ Skill chip │─▶│  Matching  │─▶│ Bridge  │  │
│  │   input    │  │   engine   │  │  view   │  │
│  └───────────┘  └─────┬──────┘  └─────────┘  │
│                       │                       │
│              ┌────────▼────────┐              │
│              │ Static JOBS[]   │              │
│              │ (in-file array) │              │
│              └─────────────────┘              │
└─────────────────────────────────────────────┘
```

**Planned (v2):**
```
┌─────────────┐      ┌──────────────────┐      ┌─────────────────────┐
│  Frontend   │─────▶│   Backend API     │─────▶│  Matching Engine     │
│ (current    │◀─────│   (FastAPI /      │◀─────│  (exact + semantic   │
│  HTML or    │      │    Node)          │      │   skill similarity)  │
│  React)     │      └──────────────────┘      └──────────┬───────────┘
└─────────────┘                                           │
                                        ┌──────────────────┼──────────────────┐
                                        ▼                  ▼                  ▼
                                 ┌────────────┐    ┌───────────────┐  ┌─────────────┐
                                 │  Job data  │    │ Embeddings /  │  │  Skill      │
                                 │  source    │    │ synonym store │  │  taxonomy   │
                                 │ (API/DB)   │    │ (vector or    │  │  (aliases)  │
                                 └────────────┘    │  lookup table)│  └─────────────┘
                                                    └───────────────┘
```

---

## 4. End-to-End Data Flow

**Current (v1):**
1. User adds skills as chips → stored in an in-memory `userSkills` array
2. User optionally selects a role filter
3. On "Find my matches," the matching function runs against the static `JOBS` array
4. For each role: intersection = matched skills, difference = gap skills, percentage = matched / required
5. Results sorted by percentage and rendered as expandable cards

**Planned (v2):**
1. User's skills come from manual input and/or a parsed resume
2. Job data is fetched from a live source (API or a maintained dataset) instead of a hardcoded array
3. Skills on both sides are normalized against a shared taxonomy (resolving synonyms/aliases)
4. Matching engine computes similarity using both exact match and embedding-based similarity for near-matches
5. Backend returns a structured result (score, matched, gap, closest-skills) to the frontend
6. Frontend renders the same bridge-span visualization, now backed by real data

---

## 5. Role of Skill Normalization (the "RAG-equivalent" layer)

Where CareerReady AI uses RAG to resolve skill synonyms and ground suggestions, SkillBridge AI's equivalent grounding layer is a **skill taxonomy**, used wherever raw string matching would miss an obvious equivalence:

| Use Case | Knowledge Source |
|---|---|
| Synonym resolution | Skill alias table (e.g. "JS" = "JavaScript", "GCP" = "Google Cloud Platform") |
| Near-match detection | Embedding similarity between skill phrases (sentence-transformer) |
| Role requirement freshness | Live job-postings data instead of a static list |

**v2 matching pipeline:** normalize skill strings → exact-match pass → embedding similarity pass for anything unmatched → combine into matched / partial / gap buckets.

---

## 6. Technology Stack

| Layer | Current (v1) | Planned (v2) |
|---|---|---|
| Frontend | Vanilla HTML/CSS/JS, single file | Same file, or React if the app grows past single-page scope |
| Backend | None (all client-side) | Python + FastAPI |
| Matching logic | Exact string match, in-browser JS | Exact match + embedding similarity (`all-MiniLM-L6-v2` or similar) |
| Job data | Static in-file array (10 mock roles) | Live API or a maintained/refreshable dataset |
| Skill taxonomy | None yet | Small alias/synonym table (JSON) |
| Persistence | None (resets on reload) | LocalStorage (simple) or a lightweight DB if accounts are added |

---

## 7. API Design (planned, once a backend exists)

| Endpoint | Method | Purpose |
|---|---|---|
| `/skills/normalize` | POST | Resolve raw skill strings to canonical taxonomy entries |
| `/jobs` | GET | Fetch the current role dataset |
| `/match` | POST | Run full matching pipeline for a given skill list (+ optional role filter) |
| `/match/{id}` | GET | Retrieve a past match result, if persistence is added |

**Example `/match` response shape:**
```json
{
  "results": [
    {
      "title": "Frontend Developer",
      "pct": 71,
      "matched": ["javascript", "html", "css", "git"],
      "gap": ["react", "responsive design", "rest apis"],
      "closest_skills": ["react", "responsive design"]
    }
  ]
}
```

---

## 8. Folder Structure

**Current (v1):**
```
skillbridge-ai/
├── index.html          # entire app: markup, styles, matching logic
└── README.md
```

**Planned (v2):**
```
skillbridge-ai/
├── backend/
│   ├── app/
│   │   ├── main.py               # FastAPI entrypoint
│   │   ├── routes/
│   │   │   ├── jobs.py
│   │   │   └── match.py
│   │   ├── matching/
│   │   │   ├── normalizer.py     # skill taxonomy lookup
│   │   │   ├── similarity.py     # embedding-based near-match scoring
│   │   │   └── scoring.py
│   │   └── data/
│   │       ├── skill_taxonomy.json
│   │       └── jobs_source.py    # API client or dataset loader
│   └── requirements.txt
├── frontend/
│   └── index.html                # current build, or migrated to React if needed
├── PROJECT_PLAN.md
└── README.md
```

---

## 9. Development Phases

**Phase 1 — Prototype** *(done)*
- Static job dataset, in-browser exact-match scoring
- Chip input, role filter, bridge-span visualization

**Phase 2 — Skill Taxonomy**
- Build a small alias/synonym table for common skill variants
- Wire it into the existing matching function before adding embeddings

**Phase 3 — Real Job Data**
- Replace the static `JOBS` array with a live source (API integration or a refreshable dataset)
- Handle messier real-world skill lists (inconsistent casing, phrasing)

**Phase 4 — Semantic Matching**
- Add embedding-based similarity for skills that don't exact-match
- Introduce a "partial match" bucket between matched and gap

**Phase 5 — Backend & Persistence**
- Move matching logic server-side (FastAPI)
- Persist a user's skill profile across sessions

**Phase 6 — Enrichment**
- Add learning-resource links per gap skill
- Optional resume import to auto-populate skills

**Phase 7 — Polish & Testing**
- Test against messy/real skill lists and edge cases (empty input, no overlap, single-skill users)
- Accessibility and mobile pass

---

## 10. Open Decisions (to confirm before v2 implementation)

- Job data source: build a small maintained dataset vs. integrate a live job-postings API
- Embeddings: local (`sentence-transformers`, free) vs. hosted API (higher quality, costs money)
- Persistence: LocalStorage only, or a real backend + accounts
- Scope: stay single-page, or migrate to a React app once the backend exists
