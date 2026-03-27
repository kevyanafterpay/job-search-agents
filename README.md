# Job Search Agents

Three Claude Code skills for the Australian job market. Find roles, analyze fit, and produce
tailored application documents — all grounded in your real experience, never fabricated.

## Skills

| Skill | What it does |
|-------|-------------|
| **job-analyzer** | Analyzes a job posting against your resume. Gives targeted resume suggestions, a concise intro email, company research, and an honest fit assessment. Handles SEEK and LinkedIn URLs natively. |
| **job-crawler** | Crawls a company's careers page, SEEK profile, or LinkedIn jobs page. Finds the top 3 roles you're most qualified for and scores each one. |
| **resume-tailor** | Reads your DOCX or PDF resume, identifies gaps, asks you about them, then produces a tailored resume and cover letter — without making anything up. |

## Quick Start

### 1. Install the skills

```bash
cp -r job-search-agents/job-analyzer ~/.claude/skills/
cp -r job-search-agents/job-crawler ~/.claude/skills/
cp -r job-search-agents/resume-tailor ~/.claude/skills/
```

### 2. Add your resume

```bash
cp ~/Documents/my_resume.docx ~/.claude/skills/baseresume/
# or PDF:
cp ~/Documents/my_resume.pdf ~/.claude/skills/baseresume/
```

That's it. If you have multiple resume variants (e.g. one for backend roles, one for fullstack),
add each as a separate file — skills pick the best match automatically. Both DOCX and PDF are
supported; you can mix formats in the same folder.

### 3. (Optional) Configure preferences

If you want to set preferences like skill domains, exclude keywords, or a custom output folder,
copy and fill in `config.yaml.example`. Otherwise the skills infer everything from your resume.

```bash
cp config.yaml.example ~/.claude/skills/config.yaml
```

### 4. Use the skills

Talk to Claude naturally:

- *"What do you think of this role? https://www.seek.com.au/job/..."* → job-analyzer
- *"Anything good at Atlassian?"* → job-crawler
- *"Tailor my resume for this job"* → resume-tailor
- *"Get me ready to apply for the #1 role"* → resume-tailor

## Project Structure

```
job-search-agents/
  baseresume/          ← your resume file(s) go here (DOCX or PDF)
  config.yaml.example  ← copy to config.yaml and fill in
  job-analyzer/
    SKILL.md
    README.md
  job-crawler/
    SKILL.md
    README.md
  resume-tailor/
    SKILL.md
    README.md
```

## Requirements

- Claude Code with browser tools enabled
- Python 3 with `python-docx` for DOCX files (`pip3 install python-docx`)
- Python 3 with `pdfplumber` for PDF files (`pip3 install pdfplumber`)
- LinkedIn login in your browser (for job-crawler LinkedIn support)
- Google Drive access (optional — only needed for the achievements brag doc in job-analyzer)
