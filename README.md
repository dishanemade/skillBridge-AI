# SkillBridge AI

A single-page tool that matches job seekers to roles based on skill gaps. Enter the skills you already have, and it ranks a set of job profiles by fit — then shows exactly which skills are missing for each one.

## What it does

- **Skill input** — add your skills as chips (type + Enter, or click a quick-add suggestion pulled from the job dataset)
- **Role filter** — compare against every built-in role, or narrow to one target job
- **Match ranking** — each role gets a percentage score based on how many of its required skills you already have
- **Gap visualization** — expand any result to see the full "span": skills you have (filled) next to skills you're missing (dashed), plus a note on the one or two skills that would close the most ground

## Files

- `index.html` — the entire app (HTML, CSS, and vanilla JS in one file, no build step, no dependencies)

## Running it

Open `index.html` in any browser. That's it — nothing to install or serve.

## Data model

Job data lives in the `JOBS` array near the top of the `<script>` block:

```js
{ title: "Frontend Developer", org: "Product & Web team", skills: ["javascript","html","css","react","git","responsive design","rest apis"] }
```

Currently seeded with 10 mock roles (Frontend Developer, Backend Developer, Data Analyst, Data Scientist, DevOps Engineer, UX Designer, Product Manager, Project Manager, QA Engineer, Digital Marketing Specialist). To use real postings:

- Replace or extend the `JOBS` array with real title/org/skills entries
- Or swap the static array for a `fetch()` call to a live job-postings API and populate `JOBS` from the response

## Matching logic

For each role, match % = (skills the user has ∩ skills the role requires) / (total skills the role requires). Skills are matched as exact, case-insensitive strings — there's no synonym handling (e.g. "JS" vs "javascript") yet, which is the main place a v2 could add real NLP/embedding-based matching instead of exact string match.

## Design notes

Visual identity built around the bridge metaphor: navy/slate base, brass accent for "gap" skills, teal for "matched" skills. Serif (Fraunces) for display type, Inter for UI text. No frameworks — kept dependency-free intentionally so the file is portable and easy to hand off or embed.

## Possible next steps

- Fuzzy/synonym skill matching instead of exact string match
- Persist user's skill list (localStorage or a backend) so it doesn't reset on reload
- Pull job data from a real API instead of the static array
- Add a "skills to learn" resource link per gap skill
