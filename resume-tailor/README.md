# resume-tailor

Reads your DOCX resume from `baseresume/`, selects the best match for a job, asks you about
experience gaps, then produces a tailored resume and cover letter. Nothing is fabricated —
only rephrases existing content and incorporates your direct answers.

## What It Does

1. **Fetches the job description** from SEEK, LinkedIn, any ATS, or pasted text
2. **Reads your DOCX resume** from the `baseresume/` folder
3. **Selects the best file** if you have multiple resume variants
4. **Identifies gaps** — up to 5 core requirements not covered by your resume — and asks you
   about each one before writing anything
5. **Tailors the resume** — rephrases bullets to use JD language, reorders for relevance,
   incorporates your gap answers — without inventing anything
6. **Writes a cover letter** — Australian professional style, grounded in your resume facts,
   direct opener (never "I am excited to apply")
7. **Saves both files** to your configured output folder

## Usage

- *"Tailor my resume for this job: [URL or paste]"*
- *"Prepare my application for this role"*
- *"Write a cover letter for this position"*
- After job-analyzer: *"Ok let's apply — tailor my resume"*
- After job-crawler: *"Prepare my docs for the #1 role"*

## Anti-Fabrication Guarantee

The skill only:
- Rephrases existing bullets to match JD language
- Reorders content to highlight relevant experience
- Incorporates experience you directly confirm when asked

It never adds jobs, skills, metrics, or achievements you didn't have.

## Output

Saved to `{output.dir}/{date}_{company}_{role}/`:
- `resume_tailored.md`
- `cover_letter.md`
