# Jobs: endpoint reference

Examples use fictional people and companies; identifiers are illustrative.

Tool: `list_campaign_jobs`

# List accepted campaign jobs

`GET /campaigns/{campaign_id}/jobs` (needs the read scope)

Jobs saved and accepted for this campaign, newest saved first, whose last saved verification is open. Excludes rejected, pending, unclear, closed and unverified jobs. Returns the posting, company, location, salary, dates and saved fit reasons, ratings, confidence and caveats when present. Includes previously reported jobs and jobs without contacts, including on paused campaigns. Uses the same owner or authorized team-admin access as the campaign; LinkedIn and Gmail connections are unnecessary. This is a read of stored decisions: it does not search, verify availability again, review, use credits, mark jobs delivered or start outreach. Follow next_cursor while has_more is true. No posting-age cutoff.

## Parameters
- `campaign_id` (string, path, required): The campaign's id, from GET /campaigns.
  ```json
  {
    "type": "string",
    "description": "The campaign's id, from GET /campaigns.",
    "format": "uuid"
  }
  ```
- `limit` (integer, query, optional): Items per page, 1 to 200. Defaults to 100.
  ```json
  {
    "type": "integer",
    "minimum": 1,
    "maximum": 200,
    "description": "Items per page, 1 to 200. Defaults to 100."
  }
  ```
- `cursor` (string, query, optional): The next_cursor from the previous page.

## How to use it

- GET /campaigns/{id}/jobs: jobs already saved and accepted for this campaign, newest saved first, whose last saved verification is open. Read them here when I ask what my job seeker campaign found; GET /search/jobs starts a new web search instead. Includes jobs shared before and jobs without contacts, on paused campaigns too. Each job has its exact job_url, company, location, salary, date_posted, saved_at, last_verified_at and saved fit reasons, ratings, confidence and caveats. Missing legacy reasons stay empty. verification_basis is saved: tell me when it was last checked, never claim this read checked that it is still open. Pending, unclear, rejected, closed and unverified jobs are excluded; posting age alone is not an exclusion. Follow next_cursor while has_more is true. Reading changes no jobs, delivery receipts, credits or outreach.

## MCP example: list_campaign_jobs
```json
{
  "campaign_id": "00000000-0000-4000-8000-000000000001"
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "id": "00000000-0000-4000-8000-000000000025",
      "title": "Senior Product Manager, Payments",
      "job_url": "https://jobs.northwind.example/1234",
      "company": {
        "id": "00000000-0000-4000-8000-000000000009",
        "name": "Northwind",
        "domain": "northwind.example",
        "website_url": "https://northwind.example",
        "linkedin_url": "https://www.linkedin.com/company/example-northwind"
      },
      "location": "New York, NY",
      "country_code": "US",
      "remote": false,
      "salary": "$180,000 - $220,000",
      "date_posted": "2026-09-30",
      "saved_at": "2026-10-12T17:15:00.000Z",
      "verification_status": "open",
      "last_verified_at": "2026-10-12T17:10:00.000Z",
      "verification_basis": "saved",
      "fit": {
        "reason": "Accepted for payments product experience.",
        "why_chosen": "Payments product work matches your background.",
        "role_rating": "High",
        "company_rating": "Medium",
        "role_rationale": "Relevant payments product experience.",
        "company_rationale": "Fintech matches your target sector.",
        "confidence": "medium",
        "caveats": [
          "Office attendance requirements need confirmation."
        ],
        "preference_gaps": []
      }
    }
  ],
  "has_more": false,
  "next_cursor": null,
  "message": "Accepted campaign jobs recorded as open at their last saved verification. This read performs no fresh availability check."
}
```

Tool: `verify_job`

# Verify a job posting

`GET /jobs/verify` (needs the read scope)

Reads the posting and says whether the job is open, closed or unverified, with the posting's own title, company, location, salary and dates. When the link is a job board and you pass company and title, SnowBear first looks for the employer's own posting and reads that. The employer's job board (Ashby, Greenhouse, Lever, BambooHR) decides when it lists the job or has removed it; an expired posting is closed; otherwise a model reads the page. Job boards that only copy postings can't confirm a job, so those come back unverified. decided_by says which rule decided. Takes 5 to 40 seconds. Reading the posting uses no credits; looking for the employer's own posting uses Exa's cost of that search x 1.2 (about 2 credits). SnowBear API reference: https://www.snowbear.ai/api-reference#jobs.

## Parameters
- `url` (string, query, required): The posting's link.
  ```json
  {
    "type": "string",
    "pattern": "^https?://",
    "description": "The posting's link."
  }
  ```
- `company` (string, query, optional): The employer's name; helps find the employer's own posting.
- `title` (string, query, optional): The job title you expect; title_matches says whether the posting agrees. With company, it lets SnowBear find the employer's own posting.
- `domain` (string, query, optional): The employer's website domain, so its own careers pages are recognised.

## How to use it

- Offer help finding companies, roles or hires that meet the user's needs. For job seekers, first establish and validate the opening and its actual employer; for sales, first establish the company and buyer role. Find suitable opportunities with SnowBear or your own research tools when those targets are missing. Once a target exists, use GET /companies?domain= or q= to look up its saved ID without provider work (or reuse a known saved ID), then show GET /companies/{company_id}/warm-connections immediately, explaining each saved path and why it may help. Then use existing warm searches for further discovery. Shared schools/employers work without LinkedIn authentication; shared background does not establish acquaintance. Preserve the stated relationship owner. Optional LinkedIn network searches use existing allowances and need a covering request. General onboarding and hiring do not automatically trigger warm searches. In later job-search conversations, use SnowBear to fetch/validate job URLs, confirm employers, find warm paths, and preview relevant outreach.

- GET /jobs/verify?url=...&company=...&title=...: before you tell me a job is open, check it. status is open, closed or unverified. Use posting (title, company, location, salary) from the result rather than what the search page said, and link to employer_posting_url when there is one. unverified means SnowBear couldn't confirm it (often a job board that only copies postings), not that it's closed.

## MCP example: verify_job
```json
{
  "url": "https://jobs.northwind.example/1234",
  "company": "Northwind",
  "title": "Senior Product Manager"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "status": "open",
    "reason": "The employer's job board still lists this job.",
    "decided_by": "employer_board",
    "url": "https://jobs.northwind.example/1234",
    "employer_posting_url": "https://jobs.northwind.example/1234",
    "checked_urls": [
      {
        "url": "https://jobs.northwind.example/1234",
        "kind": "employer_ats",
        "loaded": true
      }
    ],
    "posting": {
      "title": "Senior Product Manager, Payments",
      "company": "Northwind",
      "location": "New York, NY",
      "remote": false,
      "employment_type": "FULL_TIME",
      "date_posted": "2026-09-30",
      "valid_through": null,
      "salary": "$180,000 - $220,000",
      "excerpt": "Northwind is hiring a Senior Product Manager to lead payments..."
    },
    "title_matches": true,
    "company_matches": true,
    "checked_at": "2026-10-12T17:10:00.000Z"
  },
  "message": "The job is open (Senior Product Manager, Payments at Northwind). The employer's job board still lists this job."
}
```
