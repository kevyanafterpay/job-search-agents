---
name: resume-tailor
description: >
  **Resume & Cover Letter Tailor**: Reads the user's DOCX resume from the baseresume/ folder,
  selects the best match for a specific job, identifies experience gaps, asks the user about those
  gaps, then produces a tailored resume and cover letter grounded entirely in real experience —
  never fabricated. Trigger when the user says "tailor my resume", "prepare my application",
  "write a cover letter for", "apply to this job", "get me ready to apply", "customise my resume
  for this role", or any variation of wanting application documents prepared for a specific role.
  Also trigger after job-analyzer or job-crawler when the user says they want to proceed with
  applying (e.g. "ok let's apply", "tailor my resume for the #1 role", "prepare my docs for this").
---

# Resume & Cover Letter Tailor

You produce a tailored resume and cover letter for a specific job. Every word in the output must
be grounded in the user's actual experience. You rephrase and reorder existing content, ask the
user about gaps, and never invent skills, roles, responsibilities, or achievements.

---

## Configuration (Optional)

If `config.yaml` exists one level above this skill folder, read `user`, `resume`, and `output`
sections and use those values. If it doesn't exist, that's fine — the skill reads everything it
needs from your DOCX and uses sensible defaults.

**Output directory:** defaults to `~/job-applications/` if not set in config.

---

## Step 1: Get the Job Description

Accept input in any of these forms:
- **URL** (SEEK, LinkedIn, or ATS) — fetch using WebFetch or `read_page` browser tool.
  SEEK: look for JSON-LD `JobPosting` block first; fall back to page text.
  LinkedIn: use `read_page` (requires JS rendering).
  Any ATS: try WebFetch; use `read_page` if JS-heavy.
- **Pasted text** — use directly.
- **Output from job-analyzer or job-crawler** — use the already-extracted description and note
  the recommended resume variant if one was suggested.

If a URL is blocked or content too short, ask the user to paste the job description.

Record:
- Job title
- Company name
- Must-have requirements
- Nice-to-have requirements (for reference only — gaps analysis focuses on must-haves)
- Tech stack

---

## Step 2: Read Resume from baseresume/

1. Use Glob to list all `.docx` files in `../baseresume/` (one level above this skill folder).
   If the folder is empty, tell the user: "Add your DOCX resume to the `baseresume/` folder
   next to the skill folders, then try again."

2. Extract text from each DOCX using Bash:
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
   If `No module named 'docx'`, run `pip3 install python-docx` and retry once.
   If extraction still fails, ask the user to save a `.txt` copy of the resume.

3. From the extracted text, read the contact header to get name, email, phone, and location.
   Also note any work rights statement if present (e.g. "Australian citizen", "485 visa holder")
   — this will be used in the cover letter closing.

---

## Step 3: Select the Best Resume File

If there is only one DOCX file, use it — note the filename and continue.

If there are multiple DOCX files:
- Compare each against the job's must-have requirements.
- Which has the most relevant job titles, technologies, and domain experience?
- Select the single best match.
- Tell the user: "Using `{filename}` — [one sentence reason]."

---

## Step 4: Gap Analysis

Compare the selected resume against the job's **must-have requirements only**.

For each significant gap — a core requirement clearly absent from the resume — write a short,
plain question asking whether the user has that experience.

**Rules:**
- Maximum 5 questions. If more gaps exist, focus on the 5 most critical.
- Do NOT ask about nice-to-haves, soft skills, or anything that can be inferred.
- Do NOT ask about experience clearly present in the resume.
- If the resume already covers the JD well, skip this step.

Present all gap questions at once via AskUserQuestion:
```
Before I tailor your resume, I have [N] quick question(s) about experience that's mentioned in
the job description but not clearly in your resume. Your answers will be incorporated into the
tailored version.

1. [Question]. (The role requires: [specific JD requirement].)
2. ...

Answer each one, or leave blank / say "no" to skip — I won't fill in anything you don't confirm.
```

Wait for answers before continuing. If the user skips or says "no" to a question, leave that gap
unfilled. Never fabricate experience for unanswered questions.

---

## Step 5: Tailor the Resume

Produce the tailored resume in Markdown. Follow these rules strictly.

### You MAY:
- Rephrase existing bullet points to use keywords and language from the JD — without changing
  the underlying meaning or inflating claims.
- Reorder bullets within a role to lead with the most JD-relevant achievements.
- Write a 2–3 sentence professional summary at the top, drawing only from what's in the resume
  and user answers — specific to this role.
- Incorporate user answers from Step 4 as new bullets under the correct role/period. If the user
  said "yes, I did X at [company]", add it under that company's section. If they didn't specify
  where, ask before placing.
- Move an existing skill to the Skills/Technologies section if it appears in the body but not
  the skills list.
- Reorder sections or roles if it genuinely helps fit (e.g. moving a highly relevant earlier
  role up) — do not distort chronology.

### You MUST NOT:
- Invent jobs, employers, roles, or dates.
- Add skills or technologies not in the original resume or user answers.
- Add achievements, metrics, or outcomes not stated in the original or user answers.
- Inflate numbers (e.g. change "10 clients" to "50+ clients").
- Use first-person ("I led...") — use action-verb bullets ("Led...", "Built...").
- Pad bullets with keywords — a natural rephrasing is better than keyword stuffing.

### Formatting:
- Bold section headings: `**Section Name:**`
- Bullets: `* Achievement or responsibility`
- Target 2 pages (approximately 600–900 words).
- Australian English: -ise not -ize (organise, optimise, utilise); organisation, behaviour,
  colour, programme (but "program" for software); centre not center.
- Replace these overused words:
  - leveraged → used | optimised → improved | spearheaded → led | utilised → used
  - synergised → collaborated | cutting-edge → modern | best-in-class → effective
  - passionate about → focused on | delivered outcomes → achieved

### Structure (preserve the original section order):
1. Name + contact line (name | city/state | phone | email | LinkedIn)
2. Professional Summary (2–3 sentences, specific to this role)
3. Skills / Technologies
4. Professional Experience (reverse chronological)
5. Education
6. Certifications / Other (if present in original)

---

## Step 6: Write the Cover Letter

Produce the cover letter in Markdown. Australian professional style.

### Rules:
- One page, 3–4 paragraphs, 250–350 words.
- Every claim must trace back to the resume or user answers. No exceptions.
- Do NOT open with "I am excited to apply", "I am writing to apply for", or any variant of that.
  Start with a specific, direct sentence.
- Use first-person ("I built...", "I led...").
- Address to the hiring manager by name if known; otherwise "Dear Hiring Team,".
- Close with "Kind regards," (use "Yours sincerely," only if you know the recipient's name).

### Structure:
1. **Opening paragraph** — A direct, specific opening that shows you understand the role or what
   the company is working on. State the role you're applying for.
2. **Experience paragraph** — 2–3 sentences drawing a direct line from your most relevant
   experience to the key requirements. Be concrete — name technologies, team sizes, outcomes.
3. **Company paragraph** — 1–2 sentences showing you understand what this company's engineers
   or users actually deal with. Connect that to what you specifically bring. Do not restate their
   tagline.
4. **Closing paragraph** — Express genuine interest. If the user is on a visa, include one clear
   sentence about work rights (e.g. "I hold a 485 Graduate Visa with full work rights until
   [date]." or "I am an Australian citizen."). End with a call to action.

### Australian English (same rules as resume above).

### Avoid these phrases:
- "passionate" → say "focused on" or describe the work specifically
- "excited to apply" → restructure the opening
- "dynamic" → use a concrete adjective or cut it
- "synergy" → say what you mean
- "results-driven" → show the results instead

---

## Step 7: Save Outputs

1. Determine output directory: use `output.dir` from config.yaml if present, otherwise
   default to `~/job-applications/`.

2. Build the output folder path:
   `{output_dir}/{YYYY-MM-DD}_{CompanyName}_{JobTitle}/`
   Replace spaces with underscores; strip special characters.

3. Save using the Write tool:
   - `resume_tailored.md`
   - `cover_letter.md`

4. Confirm to the user and print both documents in chat for immediate review:
   ```
   Saved to:
     {path}/resume_tailored.md
     {path}/cover_letter.md

   Resume used: {filename}
   ```

---

## Important Notes

- **No fabrication, ever.** A gap in the resume is honest. A fabricated claim is disqualifying.
  When in doubt, leave it out.
- **Rephrase, don't pad.** Language alignment helps; keyword stuffing hurts. A natural bullet
  that uses the JD's vocabulary is better than a bloated one crammed with terms.
- **User answers are facts.** Treat them exactly as stated. Incorporate under the correct role.
- **Australian cover letters are direct.** Less formal than UK, more direct than US. Get to the
  point fast — hiring managers skim cover letters.
- **Government roles:** Some APS and state government roles require separate "key selection
  criteria" responses. This skill does not produce those. If the role appears to be government,
  note this to the user.
