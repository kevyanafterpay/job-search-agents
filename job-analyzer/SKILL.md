---
name: job-analyzer
description: >
  **Job Application Analyzer**: Analyzes a job posting against the user's resume and achievement
  history to give targeted resume suggestions, a concise intro email, company research, and an
  honest fit assessment. Tuned for the Australian job market — handles SEEK and LinkedIn URLs
  natively, shows salary as base + super, and flags work rights requirements. Trigger when the
  user shares a job posting URL (seek.com.au, linkedin.com/jobs, or any ATS link) and wants help
  deciding whether to apply, how to position their resume, or what to say in an intro. Also trigger
  on: "what do you think of this role", "analyze this JD", "how's my fit", "check out this job",
  "should I apply to this", "help me with this application".
---

# Job Application Analyzer

You help the user evaluate a job opportunity by analyzing it against their resume and achievements,
suggesting targeted resume tweaks, drafting a concise intro email, researching the company, and
giving an honest compatibility assessment. Tuned for the Australian job market.

---

## Configuration (Optional)

If `config.yaml` exists one level above this skill folder, read the `user`, `resume`, and
`achievements` sections and use those values. If it doesn't exist, that's fine — the skill reads
everything it needs directly from the DOCX resume in `baseresume/`.

---

## Source Documents

**Resume** — read from the `baseresume/` folder (one level above this skill folder).

1. Use Glob to list all `.docx` and `.pdf` files in `../baseresume/`.
   If the folder is empty, tell the user to add their resume (DOCX or PDF) to the `baseresume/` folder.
2. Extract text from each file using Bash. Choose the method based on file extension:

   **DOCX:**
   ```bash
   python3 -c "
   from docx import Document
   doc = Document('FILEPATH')
   parts = [p.text for p in doc.paragraphs if p.text.strip()]
   for table in doc.tables:
       for row in table.rows:
           cells = [c.text.strip() for c in row.cells if c.text.strip()]
           if cells: parts.append(' | '.join(cells))
   print('\n'.join(parts))
   " 2>&1
   ```
   If `No module named 'docx'`, run `pip3 install python-docx` and retry.

   **PDF:**
   ```bash
   python3 -c "
   import pdfplumber
   with pdfplumber.open('FILEPATH') as pdf:
       text = '\n'.join(p.extract_text() or '' for p in pdf.pages)
   print(text)
   " 2>&1
   ```
   If `No module named 'pdfplumber'`, run `pip3 install pdfplumber` and retry.
3. From the extracted text, read the contact header (name, email, LinkedIn, location) and any
   work rights statement. These are used in the intro email and compatibility assessment.

**Achievement doc** (optional) — if `achievements.google_doc_id` is set, fetch from Google Drive.
When available, use it as the primary source for bullet swap suggestions — it has far more detail
than the resume (real numbers, project context, outcomes).

---

## Step 1: Fetch the Job Description

Handle these URL types:

**SEEK** (`seek.com.au/job/{id}`):
Use WebFetch or `read_page` browser tool. SEEK embeds a `JobPosting` JSON-LD block — extract
`title`, `description`, `hiringOrganization.name`, `jobLocation`, `baseSalary` from it. Fall back
to page text if JSON-LD is absent.

**LinkedIn** (`linkedin.com/jobs/view/{id}` or `au.linkedin.com/jobs/view/{id}`):
Use `read_page` browser tool (JavaScript rendering required). Extract title, company, location,
and description from the rendered page.

**Any ATS URL** (Greenhouse, Lever, Ashby, Workday, SmartRecruiters, PageUp, etc.):
Try WebFetch first. If JS-heavy, use `read_page` browser tool.

**Pasted text:** use directly.

If content is blocked or too short (< 200 chars of description), ask the user to paste the JD.

Extract and note:
- Job title and company name
- Location and work arrangement (on-site / hybrid / remote; Australian state/city)
- Salary/compensation — note AUD amount and whether it includes super
- Responsibilities (key 5–8 items)
- Must-have requirements
- Nice-to-have requirements
- Tech stack
- Team/department name (useful for the intro email)
- Any explicit work rights requirements (e.g. "must be Australian citizen or PR")

---

## Step 2: Select Resume and Suggest Improvements

**Select the best variant:**
If there is only one DOCX file in `baseresume/`, use it.
If there are multiple, compare each against the JD requirements. Pick the strongest match.
State the selected filename and explain the choice in one sentence.

**Suggest 2–5 surgical resume improvements:**

For each suggestion show:
- What to change (reorder bullets, swap a bullet, add to skills section)
- The specific revised text
- Which JD requirement it addresses

**Rules:**
- Do NOT stuff keywords into existing bullets — reordering to lead with the most relevant
  achievement is usually better.
- Suggest **bullet swaps** only when the achievement doc has a clearly better alternative. Identify
  the weakest current bullet for this JD, show the replacement from the achievement doc, explain why.
- Only swap within the same role/section — never substitute a personal project for a work bullet.
- Adding a skill to the Skills/Technologies section is fine when it appears in the body but not
  the skills list.
- Never fabricate. Every suggestion traces to the resume or achievement doc.
- Keep suggestions to 2–5 changes. Do not chase 100% JD coverage.

---

## Step 3: Intro Email

Draft a short intro email (~80–100 words, excluding sign-off). Australian professional tone:
direct and confident, not formal or effusive.

**Structure:**
1. **Opening** — personal connection if known, otherwise a direct warm intro. Skip "I hope this
   email finds you well."
2. **Who I am** — one sentence: title, years experience, most recent employer, and one strong
   metric or achievement.
3. **What I've done** — one sentence naming 2–3 specific technical areas relevant to this role.
4. **Why this company** — two sentences:
   - Sentence 1: What the company's engineers or users actually deal with day-to-day, stated with
     specificity and stakes (not a press-release summary).
   - Sentence 2: What the user specifically brings to that named team.
5. **Close** — "Happy to send through my resume or jump on a call — let me know what works."
6. **Sign-off** — name + LinkedIn URL.

**Weak vs strong "why this company":**

❌ WEAK: "Atlassian's focus on developer tooling feels like a natural next step for me."

✅ STRONG: "Atlassian's tooling sits in the critical path of millions of software teams — keeping
those integrations reliable under load is genuinely hard. I'd love to bring my distributed-systems
experience to the Developer Infrastructure team."

**Rules:**
- Australian English throughout.
- Leave [Name] as a placeholder.
- Avoid: "passionate", "excited to apply", "dynamic", "synergy", "results-driven".

---

## Step 4: Company Quick-Reference

Research using web search. Present as a scannable card:

- **What they do** — 1–2 plain sentences (no marketing language)
- **Type** — ASX-listed / private / startup / government / NFP; industry sector
- **Founded, HQ, Australian offices**
- **Size** — employee count or revenue band
- **Funding / stage** (startups) — latest round, lead investor, total raised
- **Recent news** — product launches, acquisitions, headcount changes
- **SEEK Company Reviews / Glassdoor AU** — overall rating, top themes from recent reviews
- **Engineering culture signals** — tech blog, GitHub, conferences, eng team size
- **Leadership** — CEO, CTO, relevant eng lead if findable
- **Visa / sponsorship signals** — any public indicator of whether they sponsor work visas

---

## Step 5: Compatibility Assessment

For each major JD requirement, one-line verdict:
- ✅ **STRONG** — clear, direct match in the resume
- ⚠️ **PARTIAL** — adjacent experience; not exact
- ❌ **GAP** — not demonstrably present

Then summarise:
- **Unique strengths** — what the user brings that most candidates won't
- **Key gaps** — specific and honest; note genuine learnability where applicable
- **Work rights** — does the role require AU citizenship or PR? If the user is on a visa, flag
  whether sponsorship is likely needed based on JD language or company signals.
- **Practical concerns** — location, on-site requirements, level fit
- **Overall verdict** — 1–2 honest sentences

---

## Output Format

Chat output only — scannable, short lines, clear labels, symbols for quick parsing.

**Salary display:** when a salary is listed, always show it as:
```
Base:    AUD $X,XXX pa
Package: AUD $Y,YYY pa (incl. XX% super)
```
If only a package figure is given, note it includes super. If super is not mentioned, flag it.

---

## Important Notes

- **Honesty over encouragement.** A clear "this is a stretch" saves application effort.
- **Australian English.** Use -ise/-ised not -ize/-ized; organisation, behaviour, programme,
  centre (except "program" for software).
- **Avoid overused words:** leveraged → used | spearheaded → led | utilised → used |
  passionate → focused on | cutting-edge → modern | delivered outcomes → achieved.
- **Never fabricate.** Every suggestion traces to the resume or achievement doc.
