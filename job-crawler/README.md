# job-crawler

Crawls a company's careers page, SEEK profile, or LinkedIn jobs page to find open positions,
then ranks the top 3 roles you're most qualified for. Tuned for the Australian job market.

## What It Does

When you share a URL or mention a company name, this skill:

1. **Detects the source** — SEEK, LinkedIn, or ATS platform (Lever, Greenhouse, Ashby, Workday,
   SmartRecruiters, PageUp, Recruitee, Workable) and uses the most reliable method
2. **Fetches and filters** all open roles by location (Australian states/remote) and exclusion
   keywords
3. **Matches on transferable skills**, not just job titles — reads actual descriptions
4. **Scores each role 0–100** using: core skill overlap (35%), level fit (20%), domain relevance
   (20%), gap severity (15%), growth opportunity (10%)
5. **Presents the top 3** with match breakdowns, salary in AUD + super, resume recommendation,
   and work rights flags

## Usage

- *"What's open at Canva?"*
- *"Crawl Atlassian's careers page"*
- *"Anything good at https://www.seek.com.au/companies/..."*
- *"Find roles at Commonwealth Bank for me"*
- *"Check what Afterpay has open"*

## Supported Sources

| Source | Method |
|--------|--------|
| SEEK (`seek.com.au/companies/{slug}/jobs`) | Browser rendering + JSON-LD |
| LinkedIn (`linkedin.com/company/{slug}/jobs`) | Browser rendering |
| Lever | Public JSON API |
| Greenhouse | Public JSON API |
| Ashby | Public JSON API |
| Workable | Public JSON API |
| Recruitee | Public JSON API |
| SmartRecruiters | Public JSON API |
| PageUp | Browser rendering |
| Workday | Browser rendering |
| Other / unknown | Browser-based fallback |
