---
name: job-crawler
description: >
  **Job Posting Crawler & Matcher**: Crawls a company's careers page, SEEK company profile, or
  LinkedIn jobs page to find open positions, then ranks the top 3 roles the user is most qualified
  for based on their resume and skill domains. Tuned for the Australian job market — handles
  seek.com.au and linkedin.com/company natively alongside standard ATS platforms. Trigger when
  the user shares a careers URL or company name and wants to find best-fit roles, or says "what's
  open at [company]", "crawl [company] careers", "find roles at [company]", "anything good at
  [company]", "scan their jobs page", "check what [company] has", or pastes a careers/jobs URL
  and asks which roles to apply for.
---

# Job Posting Crawler & Matcher

You help the user find the best-fit roles at a specific company by crawling their career page,
filtering for relevant positions, and ranking the top 3 based on their resume and skill domains.
Tuned for the Australian job market.

---

## Configuration (Optional)

If `config.yaml` exists one level above this skill folder, read `resume`, `skills`, and
`preferences` from it. If it doesn't exist, that's fine — the skill reads your resume from
`baseresume/` and infers your skills, experience level, and specialty from the content.

---

## Source Documents

Read the user's resume from the `baseresume/` folder (one level above this skill folder).

Use Glob to list `.docx` and `.pdf` files in `../baseresume/`. Extract text using Bash,
choosing the method based on file extension:

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

From the extracted text, infer:
- **Skill domains** — technologies, platforms, and problem areas present in the experience section
- **Experience level** — estimate from total years and seniority of most recent roles
- **Primary specialty** — dominant pattern across job titles and responsibilities
- **Location** — from the contact header

If `config.yaml` exists and has `skills.domains`, `preferences.level`, or `preferences.primary_specialty`
set, use those values instead (more accurate than inference).

---

## Step 1: Identify the Source

The user provides a careers URL or a company name. Determine the best source to query.

### Australian Job Boards

**SEEK** (`seek.com.au`) — primary Australian job board:

- Company jobs page: `https://www.seek.com.au/companies/{slug}/jobs`
  Use `read_page` browser tool to render and extract job listings.
- If the user gives a search URL like `seek.com.au/jobs?...`, use `read_page` on it.
- SEEK pages contain JSON-LD `JobPosting` objects — extract these for structured data.
- If you only have a company name, try constructing the SEEK company URL:
  `https://www.seek.com.au/companies/{company-name-slug}/jobs`

**LinkedIn** (`linkedin.com/company/{slug}/jobs`):

- Use `read_page` browser tool (requires JavaScript).
- LinkedIn company jobs pages list open roles with titles and locations.
- If the user is logged in, more detail is available. If not, extract what's publicly visible.
- Also try: `https://au.linkedin.com/company/{slug}/jobs`

### ATS Platforms (direct API — more reliable than scraping)

Detect which ATS the company uses from the URL or embedded page source, then hit the API:

| Platform | URL pattern | API endpoint |
|----------|------------|--------------|
| **Lever** | `jobs.lever.co/{company}` | `https://api.lever.co/v0/postings/{company}?mode=json` |
| **Greenhouse** | `boards.greenhouse.io/{company}` | `https://api.greenhouse.io/v1/boards/{company}/jobs?content=true` |
| **Ashby** | `jobs.ashbyhq.com/{company}` | `https://api.ashbyhq.com/posting-api/job-board/{company}?includeCompensation=true` |
| **Workable** | `apply.workable.com/{company}` | `https://apply.workable.com/api/v1/widget/accounts/{company}` |
| **Recruitee** | `{company}.recruitee.com` | `https://{company}.recruitee.com/api/offers` |
| **SmartRecruiters** | `careers.smartrecruiters.com/{company}` | `https://api.smartrecruiters.com/v1/companies/{company}/postings` |

**PageUp** (used heavily in Australia by banks, universities, and government):
- URL patterns: `{company}.pageuppeople.com/jobs` or embedded in company career sites
- No clean public API — use `read_page` browser tool on the listings page.
- Common users: ANZ, Westpac, universities, state government agencies.

**Workday** (large Australian corporates):
- URL pattern: `{company}.wd{N}.myworkdayjobs.com/`
- Use `read_page` browser tool; Workday requires JS rendering.

**How to detect:**
1. Match the user's URL against the patterns above.
2. If it's a company's own domain (e.g. `careers.acme.com.au`), fetch the page and look for
   iframes or API calls to the known platforms.
3. Extract the `{company}` slug from the detected platform URL.

**If platform is unknown or undetectable:** use `read_page` browser tool on the careers page
directly and extract what's visible.

---

## Step 2: Fetch and Filter Job Listings

Once you have the source, fetch all available roles. Then apply hard filters:

**Hard filters (apply before any ranking):**

1. **Location** — if `preferences.au_only` is true, exclude roles with no Australian location or
   remote eligibility. Roles listed as "Remote" are fine if the user's location qualifies.
   Australian locations to recognise: NSW, VIC, QLD, SA, WA, ACT, TAS, NT and their capital
   cities (Sydney, Melbourne, Brisbane, Adelaide, Perth, Canberra, Darwin, Hobart).
   If a role lists multiple locations, keep it if any is Australian.

2. **Exclusion keywords** — remove roles whose title or description contains any term from
   `preferences.exclude_keywords` as a hard requirement. If it appears only as "preferred",
   include the role but flag it.

**Narrow to a workable set (~8–15 roles):**

Use `skills.domains` and `preferences.primary_specialty` to filter:
- **Tier 1 — direct title matches** to primary specialty (always include)
- **Tier 2 — transferable-skill matches** — roles in adjacent areas from `skills.domains`; read
  actual descriptions before deciding (titles are often misleading)
- **Tier 3 — only if nothing better** — further from core strengths

**Deduplicate:** same role at multiple Australian cities = one entry, list all locations.
Different seniority levels = pick the level closest to `preferences.level`, note others exist.

---

## Step 3: Extract Job Details

For each candidate role, extract:
- Title and seniority level
- Location(s) / remote policy / work arrangement (on-site, hybrid, remote)
- Salary / compensation (note AUD and whether it includes superannuation)
- Key responsibilities (summarised, 4–6 items)
- Must-have requirements
- Nice-to-have requirements
- Tech stack mentioned
- Any explicit work rights requirements

**Platform-specific fields:**
- Lever: `descriptionPlain`, `lists` (requirements, responsibilities)
- Greenhouse: `content` HTML field — extract text
- Ashby: `descriptionPlain` or `descriptionHtml`, `compensationTierSummary`
- SEEK / LinkedIn: extract from rendered page text and JSON-LD if present

---

## Step 4: Score and Rank

For each role compute a **Match Score (0–100)**:

| Factor | Weight | How to assess |
|--------|--------|---------------|
| **Core skill overlap** | 35% | % of must-have requirements covered by resume or `skills.domains` |
| **Experience level fit** | 20% | Exact level match → 100%; one level off → 60%; two+ → 20% |
| **Domain relevance** | 20% | Same domain → 100%; adjacent → 70%; unrelated → 30% |
| **Requirement gap severity** | 15% | No must-have gaps → 100%; 1 minor → 80%; 1 major → 50%; 2+ major → 20% |
| **Growth opportunity** | 10% | New domain + familiar skills → 100%; same + same → 50%; all new → 30% |

Select top 3 by score. Be transparent — a 45/100 is still useful information.

---

## Step 5: Present Results

For each of the top 3:

**[Rank]. [Job Title] — [Location / Work Arrangement] — Match: [X/100]**
- **Link:** posting URL
- **Match breakdown:** one line per factor with sub-score and reason
- **Why it fits:** 2–3 sentences connecting role requirements to specific resume experience
- **Key gaps:** specific and honest
- **Resume to use:** which DOCX file from `baseresume/` and why
- **Compensation:** if available — show as `Base: AUD $X pa | Package: AUD $Y pa (incl. super)`
  If only a package figure is listed, note it includes super. If super is not mentioned, flag it.
- **Work rights:** flag if role requires AU citizenship or PR; note sponsorship signals if any

After top 3, include a brief **"Also considered"** list — other close roles with one-line reasons
and match scores.

If no roles are a good fit, say so: "Nothing strong at this company right now — best I found is X
at 38/100" is a valid and useful output.

---

## Important Notes

- **Read descriptions before judging.** Titles are misleading — a "Platform Engineer" role may
  be a perfect backend match. Always read the actual JD for Tier 2 candidates.
- **SEEK and LinkedIn first** for Australian companies — many post only on these boards and not
  on their own ATS, especially smaller companies.
- **ATS APIs when available** — faster and more complete than scraping.
- **Salary includes super.** When quoting Australian salaries, always clarify whether the figure
  is base (excluding super) or total package (including super). The difference is ~11.5%.
- **Deduplicate aggressively.** Same role, different locations = one entry.
- **Honesty.** If this company has nothing good right now, say so clearly.
- **Speed over perfection.** Filter first, deep-read only the top 5–8 finalists.
