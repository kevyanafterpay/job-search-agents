# job-analyzer

Analyzes a job posting against your resume and achievement history. Tuned for the Australian
job market — handles SEEK and LinkedIn URLs natively, shows salary as base + super, and flags
work rights requirements.

## What It Does

When you share a job posting URL, this skill:

1. **Fetches the JD** from SEEK, LinkedIn, or any ATS (Greenhouse, Lever, Ashby, Workday, etc.)
2. **Picks the best resume** from your `baseresume/` folder and explains why
3. **Suggests 2–5 surgical changes** — bullet reorders, swaps, skills additions — grounded in
   your actual experience, never fabricated
4. **Drafts an intro email** (~80–100 words) in direct Australian professional tone
5. **Compiles a company card** — what they do, ASX/funding status, SEEK reviews, visa signals
6. **Gives a compatibility assessment** with ✅/⚠️/❌ for each requirement, work rights check,
   and an honest overall verdict

## Usage

- *"What do you think of this role? https://www.seek.com.au/job/..."*
- *"Analyze this JD for me"*
- *"How's my fit for this position?"*
- *"Should I apply to this? [LinkedIn URL]"*

## Supported Job Sources

- SEEK (`seek.com.au`) — extracts JSON-LD structured data
- LinkedIn (`linkedin.com/jobs`, `au.linkedin.com/jobs`)
- Greenhouse, Lever, Ashby, Workday, SmartRecruiters, PageUp
- Any careers page (browser-based fallback)
- Pasted job description text
