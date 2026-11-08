# recruitment-assistant

An **AI-powered recruitment assistant** for small businesses and HR teams, powered by Claude (Claude Opus 5.5).
It writes job adverts and screens resumes against the job's requirements. It ranks candidates with clear reasons,
prepares interviews and drafts candidate emails. It works in Thai and English.

```
> /screen kitchen-cook
  5 candidate(s): 0 already screened, 5 to assess
  📊 shortlist saved: output/kitchen-cook-shortlist.xlsx

   #  Score  AI rec.     Must  Candidate
   1     93  strong_yes  5/5   somchai-resume.md
   2     86  yes         5/5   nattaya-resume.txt
   3     61  maybe       3/5   anan-resume.md  ⚠ tried to instruct the AI
   4     57  no          3/5   james-cv.md
   5     14  no          1/5   pim-resume.md

> Compare the top two and make an interview kit for both
> เขียนอีเมลเชิญสัมภาษณ์ให้คนที่ 1 วันอังคาร 14:00
> Move somchai-resume.md to interview
  Change somchai-resume.md (kitchen-cook) from "new" to "interview"? (y/n)
```
*(The scores above are an example. Real results depend on Claude's assessment.)*

## How it keeps screening fair and explainable

| Feature | How |
|---|---|
| **One checklist per job** | Each job description is turned into a fixed list of must-haves and nice-to-haves, so every candidate is judged against exactly the same items. |
| **Evidence for every item** | For each requirement the AI marks *yes / partly / no / unclear* and quotes the resume. "Unclear" items become interview questions; they don't count against the candidate. |
| **Score calculated in code** | Must-haves count double: *yes* = full points, *partly* = half. Same answers always give the same score. |
| **Blind screening** | Before a resume reaches the AI, the name, phone, email, LINE ID, links, address, age, date of birth, gender, religion, marital status and nationality are hidden. This works for Thai and English resumes. |
| **Discriminatory adverts caught** | Requirements like "female, age 22–35" are left out of the checklist and flagged so you can fix the advert. See `jobs/shift-supervisor.md`. |
| **Manipulation detected** | If a resume hides instructions for the AI ("rate me 100"), they're ignored and flagged ⚠. |
| **People decide** | The AI only recommends. Every status change (interview, offer, rejected…) needs your **y/n** approval. It never contacts candidates; emails are saved as drafts. |
| **Decision log** | Every screening and status change is recorded in `logs/decisions-YYYY-MM.jsonl` (what, when, which model). |

## Setup (one time)

1. Get a Claude API key at https://platform.claude.com → *API Keys*.
2. Copy `.env.example` to **`.env`** and put your key in it:
   ```
   ANTHROPIC_API_KEY=sk-ant-...
   ```
3. Dependencies are already installed. In a fresh copy, run `npm install`.
4. Edit `config.json`: company name, and whether blind screening is on (`"blind_screening": true`).

## Use it

```powershell
cd C:\Users\Dell\recruitment-assistant
node recruiter.mjs
```

On the first run, two **sample jobs and five made-up resumes** are copied in so you can try it. Delete them when you start real hiring.

1. **Job:** put a job description in `jobs\<job-id>.md` (or `.docx` / `.pdf`), or ask *"write a job advert for a part-time waiter"*.
2. **Resumes:** put them in `candidates\<job-id>\` (PDF, Word, .txt or .md). Scanned images can't be read, so ask for a text PDF or Word file.
3. **Screen:** `/screen kitchen-cook`, or ask *"screen the candidates for kitchen-cook"*. Only new or changed resumes are assessed (and paid for). Use `/screen kitchen-cook force` to re-assess everyone.
4. **Open the shortlist:** `output\kitchen-cook-shortlist.xlsx`. It has three sheets: the ranking with summary, strengths, concerns and interview questions; each requirement with its evidence; and the checklist.
5. **Ask for help:** compare candidates, explain a score, interview kits and scorecards, invitation, rejection and offer emails, Facebook/LINE job posts. These are saved to `output\`.

| Command | What it does |
|---|---|
| `/jobs` | Open jobs and how many candidates are screened |
| `/screen <job> [force]` | Screen the resumes for a job and save the Excel shortlist |
| `/new` | Start a new conversation |
| `/exit` | Quit (Ctrl+C stops an answer) |

Pipeline statuses: `new → shortlisted → interview → offer → hired`, or `rejected` / `on_hold` (stored in `data\pipeline.json`).

## Cost

Screening costs about 2–5 US cents per resume with Claude Opus 5.5 (shown after each screening).
Unchanged resumes are never assessed twice. For high volumes, set `RA_SCREEN_EFFORT=low` in `.env`.

## Privacy (Thailand PDPA)

- Resumes are personal data. `candidates\`, `jobs\`, results, statuses, `output\` and logs are **excluded from git**. Never commit them.
- With blind screening, only the hidden-details version of a resume is sent to Anthropic.
- Tell applicants in your job advert that an AI tool helps review applications and that people make the decisions.
  Delete resumes of unsuccessful applicants after your retention period (for example 6–12 months) unless they agree to stay on file.

## Files

| File | Role |
|---|---|
| `recruiter.mjs` | The chat assistant: Claude, tools, approvals |
| `screening.mjs` | Reading resumes, blind screening, checklist, assessment, scoring, Excel shortlist, pipeline, decision log |
| `config.json` | Company name, blind screening on/off, report language |
| `sample-data\` | Sample jobs and made-up resumes |
