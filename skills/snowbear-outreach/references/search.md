# Search: endpoint reference

Examples use fictional people and companies; identifiers are illustrative.

Tool: `api_get`

# Search companies

`GET /search/companies` (needs the read scope)

Companies matching a plain-language description (3 to 20 words, no quotes, OR or site:), up to 10, each with its homepage and a short description. Your own company and companies you rejected are left out. Pass exclude_domains (comma-separated, up to 100) to get new ones when you search again.

## Parameters
- `q` (string, query, required): What the companies do and who they serve, the way their homepage would say it.
- `exclude_domains` (string, query, optional): Comma-separated domains to leave out (ones you already have).

## How to use it

- Offer help finding companies, roles or hires that meet the user's needs. For job seekers, first establish and validate the opening and its actual employer; for sales, first establish the company and buyer role. Find suitable opportunities with SnowBear or your own research tools when those targets are missing. Once a target exists, use GET /companies?domain= or q= to look up its saved ID without provider work (or reuse a known saved ID), then show GET /companies/{company_id}/warm-connections immediately, explaining each saved path and why it may help. Then use existing warm searches for further discovery. Shared schools/employers work without LinkedIn authentication; shared background does not establish acquaintance. Preserve the stated relationship owner. Optional LinkedIn network searches use existing allowances and need a covering request. General onboarding and hiring do not automatically trigger warm searches. In later job-search conversations, use SnowBear to fetch/validate job URLs, confirm employers, find warm paths, and preview relevant outreach.

- GET /search/companies?q=...: describe the companies the way their own homepage would: what they sell or do and who they serve, 6 to 14 words, e.g. "companies selling payroll software to mid-market businesses". Never describe the need or pain my product solves (that returns vendors of it), never write a negation, and run 2 or 3 searches for different kinds of buyer. Start with my home metro for at most two searches, then widen, passing what you already have as exclude_domains. Then judge each result's fit yourself: not my own company or a competitor, and its core business must match, not a side offering.

## MCP example: api_get
```json
{
  "path": "/search/companies",
  "query": {
    "q": "companies selling payroll software to mid-market businesses"
  }
}
```
Search text uses q; query and search are aliases. A generic api_get query string is accepted as shorthand. Conflicting values are refused.

## Example response (200 OK)
```json
{
  "data": [
    {
      "name": "Northwind",
      "website_url": "https://northwind.example",
      "domain": "northwind.example",
      "source_url": "https://northwind.example/",
      "page_kind": "homepage",
      "description": "Northwind builds payroll and benefits software for growing companies..."
    }
  ],
  "message": "1 company. Judge each one's fit before using it: not the user's own company or a competitor, and its core business must match."
}
```

Tool: `api_get`

# Search people at a company

`GET /search/people` (needs the read scope)

People currently at a company in the roles you name (one role family per search works best, up to 4 titles), from LinkedIn profiles on the web. With the company's domain or LinkedIn page, people at other companies with the same name are left out. Each person has current_company, current_title (from their work history, not the headline), linkedin_url, employer_match (id: matched the company's own records; name: matched by name only) and ended_role (true when that job has ended).

## Parameters
- `company` (string, query, required): The company's name.
- `domain` (string, query, optional): The company's website domain. Strongly recommended.
- `linkedin_url` (string, query, optional): The company's LinkedIn page.
- `description` (string, query, optional): A short description of what the company does (helps for big companies).
- `roles` (string, query, required): Comma-separated titles, e.g. VP of Engineering,Head of Engineering.
  ```json
  {
    "type": "string",
    "minLength": 1,
    "pattern": "^(?:\\s*,)*[^,]+(?:,(?:\\s*,)*[^,]+){0,3}(?:,\\s*)*$",
    "description": "Comma-separated titles, e.g. VP of Engineering,Head of Engineering."
  }
  ```
- `location` (string, query, optional): Where the people are (a country works better than a small city).

## How to use it

- GET /search/people?company=...&domain=...&roles=...: pass the company's domain whenever you know it, so people at a same-name company are left out. One role family per search; for a person's location use a country rather than a city I'm only starting with. The role you searched for is only how someone was found, never proof of their job: read current_title. Similar names, agencies, clients and subsidiaries aren't a match without evidence; when it matters, enrich them.

## MCP example: api_get
```json
{
  "path": "/search/people",
  "query": {
    "company": "Northwind",
    "domain": "northwind.example",
    "roles": "VP of Engineering",
    "location": "United States"
  }
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "name": "Jane Doe",
      "current_company": "Northwind",
      "current_title": "VP of Engineering",
      "headline": "Engineering at Northwind",
      "linkedin_url": "https://www.linkedin.com/in/example-jane-doe",
      "location": "New York, New York, United States",
      "excerpt": "Leads platform engineering at Northwind...",
      "employer_match": "id",
      "ended_role": false
    }
  ],
  "message": "1 person currently at Northwind. The role you searched for is only how they were found, not proof of their job: check current_title."
}
```

Tool: `api_get`

# Search job postings

`GET /search/jobs` (needs the read scope)

Job postings matching a plain-language description (8 to 25 words: one role family, the kind of employer, one location; no quotes, OR or site:), up to 10. By default only employers' own job boards (Ashby, Greenhouse, Lever, Workday and similar) posted in the last 30 days; ats_only=false widens to career sites and job boards, and posted_within_days goes up to 90. posted_date is set only when it's the posting's real date; otherwise indexed_date is when the page was seen. Check a posting with Verify a job posting before calling it open.

## Parameters
- `q` (string, query, required): One role family, the kind of employer, and one location.
- `posted_within_days` (integer, query, optional): 1 to 90 (default 30). Values above 90 are clamped to 90.
  ```json
  {
    "type": "integer",
    "minimum": 1,
    "description": "1 to 90 (default 30). Values above 90 are clamped to 90."
  }
  ```
- `ats_only` (boolean, query, optional): true (default): only employers' job boards. false: also career sites and job boards.

## How to use it

- Offer help finding companies, roles or hires that meet the user's needs. For job seekers, first establish and validate the opening and its actual employer; for sales, first establish the company and buyer role. Find suitable opportunities with SnowBear or your own research tools when those targets are missing. Once a target exists, use GET /companies?domain= or q= to look up its saved ID without provider work (or reuse a known saved ID), then show GET /companies/{company_id}/warm-connections immediately, explaining each saved path and why it may help. Then use existing warm searches for further discovery. Shared schools/employers work without LinkedIn authentication; shared background does not establish acquaintance. Preserve the stated relationship owner. Optional LinkedIn network searches use existing allowances and need a covering request. General onboarding and hiring do not automatically trigger warm searches. In later job-search conversations, use SnowBear to fetch/validate job URLs, confirm employers, find warm paths, and preview relevant outreach.

- GET /search/jobs?q=...: one role family, the kind of employer in your own words, and one location per search (12 to 25 words works best; two locations means two searches). No employee counts, dates, excluded employers or my biography in the query. It starts with employers' own job boards from the last 30 days; if results are thin or repetitive, search again with ats_only=false.

## MCP example: api_get
```json
{
  "path": "/search/jobs",
  "query": {
    "q": "senior product manager roles at fintech startups building payments infrastructure in New York"
  }
}
```
Search text uses q; query and search are aliases. A generic api_get query string is accepted as shorthand. Conflicting values are refused.

## Example response (200 OK)
```json
{
  "data": [
    {
      "title": "Senior Product Manager, Payments - Northwind",
      "url": "https://jobs.northwind.example/1234",
      "link_kind": "employer_ats",
      "posted_date": "2026-09-30",
      "indexed_date": null,
      "snippet": "Northwind is hiring a Senior Product Manager to lead payments..."
    }
  ],
  "message": "1 postings. Check one with GET /jobs/verify before calling it open; it reads the posting and gives the employer and title."
}
```
