### Warm searches
A warm search finds people at a company who went to the same school or worked at the same company as me (or as backgrounds you name, for example a candidate's), so we have something real in common to open with. Use it whenever I ask who I know (or who someone I'm helping knows) at a company, or for a warm way in. It runs in the background: usually 1 to 5 minutes; a step that hits a temporary error retries within a couple of minutes, and status_summary says so.
- POST /warm-searches: {"companies": [...], "target"?: {...}, "backgrounds"?: [...], "locations"?: [...], "include_linkedin_network"?: false}.
  - Companies and their LinkedIn pages: for each company, send its LinkedIn page as linkedin_url whenever you can. Fictional example: {"name": "Northwind", "domain": "northwind.example", "linkedin_url": "https://www.linkedin.com/company/example-northwind"}. It's the page SnowBear searches, so the right page is what makes the search find people. Before sending it, check it's the company's own page: open or search for it, and confirm the website listed on the page is the company's domain. Many companies have look-alike or old pages (example-northwind-old vs example-northwind). When the website on the page is a short link (short.example/...), follow it to the real site. Don't guess a page from the company's name. If you can't find or confirm the page, send name and domain only; SnowBear checks its saved page itself.
  - Each company in the response shows the page searched and linkedin_url_source: yours (the page you sent), verified (SnowBear confirmed it), saved_unverified (SnowBear couldn't confirm it), or saved in the first answer (SnowBear's saved page, checked once the search runs). A page that found people is the right one. Only when a company found nobody and comes back with linkedin_url_warning: unverified or note page_unverified, tell me, find and confirm the right page, and start the search again with it. no_people_found means the page was fine and nobody there shares these backgrounds.
  - Caller-supplied linkedin_url is authoritative: SnowBear uses that page even if its website disagrees or verification is unavailable; it never substitutes a saved namesake. With only a domain, SnowBear preserves the requested domain, checks website links, then uses exact-domain research and the existing LinkedIn name-search and AI-resolver fallbacks. Indirect results such as founder posts can supply the company link. Automatically discovered pages must verify against that domain or observed redirects. requested_domain shows the original input. If a company is unresolved/not_searchable, research that domain and start a new search with its linkedin_url; do not report it as searched with zero matches. Partial results must distinguish searched companies from unsearched companies.
  - target {"type": "company", "persona": "..."} (the default) rates people by how useful they are for reaching that kind of person; without a persona, by how well they can open doors at the company in general. {"type": "role", "title": "...", "job_url": "..."} rates them for a job I'm pursuing there (its team, likely hiring manager, same function, recruiters). One target per search: for a different posting at each company, run one search per posting, or give the title without a job_url.
  - Leave backgrounds out to compare against my own schools and employers. Pass them to compare against someone else's (a candidate's); those matches stay with this search, and you should always pass a target that describes that person's goal. Name each school the way its LinkedIn page does ("UCL", not "University College London"); a background that matches nobody at any company comes back in backgrounds with note nobody_found.
  - locations: where the people should be, by country. Leave it out to search the United States. Each entry is one of: a country, by name, ISO code or common alias ("United Kingdom", "UK", "GB", "England", "Brazil", "BR", "USA"); "Europe", which covers every country in geographic Europe; or "anywhere", which means no location filter (send it alone). Cities, states and regions aren't accepted: map them to their country yourself ("London" → "United Kingdom", "Texas" → "United States"). An entry SnowBear doesn't recognize is refused with a 400 that names it, so nothing is dropped silently. When my goal is outside the US (a role in London, a team in Brazil), pass those countries; pass ["anywhere"] only when I want people anywhere. The response's locations shows exactly what was applied (normalized names); tell me when it limited the search. People whose location LinkedIn doesn't show are kept.
  - include_linkedin_network adds my 1st- and 2nd-degree connections there (mutual connections). It uses my LinkedIn search allowance and 2.5 credits per profile, and can take until the next business day, so only turn it on when I ask for my connections or mutual connections.
  - When I ask for mutual connections, who I know, or a warm way into a company, start the search right away: shared schools and employers need no LinkedIn. If my LinkedIn isn't connected (GET /linkedin), still start it; the answer carries warnings: linkedin_not_connected and the network part is skipped. Then tell me what it found and offer a link to reconnect LinkedIn (POST /linkedin/connect-link) for my mutual connections. If it answers nothing_to_run, ask me for my schools and past employers and pass them as backgrounds.
  - It answers right away with search_id, and already_known: people I already have warm paths to there. Tell me about those first. planned counts the searches: one per company for schools and one for employers (two companies with both is 4). Sending the same request again the same day returns that search with its status (still running, or finished: read its people with GET /warm-searches/{search_id}/people); one that failed, was cancelled, had nothing to run, or where a company's search failed starts fresh.
- GET /warm-searches/{search_id}?cursor=...: check every next_check_after_seconds (20 while it runs) and pass next_cursor back each time, so you get only the people found since your last check. Checking moves the search forward, so keep checking while I'm waiting. Share people with me as they arrive rather than waiting for the end. A person comes back again when they gain another path or their review settles; update them by contact_id rather than listing them twice. Stop when next_check_after_seconds is null; if only the LinkedIn network search is left, check back in an hour. A check can take up to half a minute while the search is busy, so allow 60 seconds before timing out.
  - companies says what happened at each company (people_found, and note: no_people_found, page_unverified, not_searchable or search_failed, with note_detail); backgrounds lists each school and employer searched with people_found (note nobody_found when it matched nobody anywhere). status is queued, running, completed, partial (a company couldn't be searched, or its search failed; companies says which), failed, cancelled or nothing_to_run (no schools or employers to compare against; reason says why).
- GET /warm-searches/{search_id}/people: everyone found, one entry per person, best first; page with next_cursor while has_more is true. Use it for the final list: group people by company, show the best few at each (around five) with their title, how they're warm and why, and offer the rest.
- GET /warm-searches lists my searches; POST /warm-searches/{search_id}/cancel stops one (a finished search is left as it is, and the answer says so). At most 5 can run at once (409 too_many_running names them).
- Each person has paths, one per reason they're warm: shared_school or shared_employer (with the background), connected (my 1st-degree connection) or mutual (2nd degree, with mutual_connections). A shared school or employer is a talking point, not evidence we know each other, so never write as if we do.
  - relation says how they relate to the company: works_here, or board, investor, advisor or other_role (an ambassador, mentor, campus leader or similar program role) for people who don't work there. They rank lower than employees with similar paths, but can still make an introduction; say what their role is. title is their title at the company, and also_at lists their other current jobs (someone matched at two of the companies is listed at the one their headline names).
  - fit.reviewed says whether SnowBear's review rated them; when it did, networking_value (0 to 1) and why say how useful they are for the target. Unreviewed people (reviewed: false) are shown as found, without a rating.
  - warm_score (0 to 1) is the ranking: networking_value, how close the shared background is, and their relation to the company together. A path's overlap says when your time at a school or employer overlapped (only known for my own backgrounds, from my profile); null means unknown, which is common.

# Warm searches: endpoint reference

Examples use fictional people and companies; identifiers are illustrative.

Tool: `api_get`

# List warm searches

`GET /warm-searches` (needs the read scope)

Your warm searches started through the API, newest first, with their status and companies.

## Parameters
- `limit` (integer, query, optional): How many to return, 1 to 100 (default 100).
  ```json
  {
    "type": "integer",
    "minimum": 1,
    "maximum": 100,
    "description": "How many to return, 1 to 100 (default 100)."
  }
  ```
- `cursor` (string, query, optional): next_cursor from the previous page.

## How to use it

- GET /warm-searches lists my searches; POST /warm-searches/{search_id}/cancel stops one (a finished search is left as it is, and the answer says so). At most 5 can run at once (409 too_many_running names them).

## MCP example: api_get
```json
{
  "path": "/warm-searches"
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "search_id": "00000000-0000-4000-8000-000000000020",
      "status": "completed",
      "target": {
        "type": "role",
        "title": "Senior Product Manager"
      },
      "locations": [
        "United States"
      ],
      "companies": [
        {
          "company_id": "00000000-0000-4000-8000-000000000006",
          "name": "Northwind"
        }
      ],
      "created_at": "2026-10-12T17:00:02.110+00:00",
      "completed_at": "2026-10-12T17:05:40.002+00:00"
    }
  ],
  "has_more": false,
  "next_cursor": null
}
```

Tool: `start_warm_search`

# Start a warm search

`POST /warm-searches` (needs the write scope)

Starts a search for people at up to 10 companies who share a school or past employer with you. For each company, send its LinkedIn page as linkedin_url whenever you can: it's the page SnowBear searches, so the right page is what makes the search find people. Before sending it, check it's the company's own page (the website listed on the page is the company's domain); many companies have look-alike or old pages (ramp-hq vs ramp), so don't guess one from the name. Without a page SnowBear checks its saved one itself. Each company comes back with the page searched and linkedin_url_source (yours, verified, saved_unverified, or saved before the search has run); a page that found people is the right one. Its domain is the one saved in SnowBear; requested_domain retains the domain you supplied. Caller-supplied linkedin_url is authoritative: SnowBear uses that page even if its website disagrees or verification is unavailable; it never substitutes a saved namesake. With only a domain, SnowBear preserves the requested domain, checks website links, then uses exact-domain research and the existing LinkedIn name-search and AI-resolver fallbacks. Indirect results such as founder posts can supply the company link. Automatically discovered pages must verify against that domain or observed redirects. requested_domain shows the original input. If a company is unresolved/not_searchable, research that domain and start a new search with its linkedin_url; do not report it as searched with zero matches. Partial results must distinguish searched companies from unsearched companies. Without backgrounds it compares against every school and employer on your SnowBear profile; with backgrounds it compares against those instead (for example a candidate's), and those matches stay with this search only. target (default company) decides what the people are rated against: a kind of person at the company, or a job you're pursuing there (one target per search: one search per posting). locations (default the United States) limits where the people are, by country, "Europe" or "anywhere"; the response's locations shows what was applied. include_linkedin_network also searches your 1st- and 2nd-degree LinkedIn connections there (uses your LinkedIn search allowance and 2.5 credits per profile, and is paced by LinkedIn, so it can take until the next business day). The same request on the same day returns that search with its status; one that failed, was cancelled, had nothing to run, or where a company's search failed starts fresh. Answers 202 with search_id (200 for a search that already finished today or has nothing to run), and already_known (people you already have warm paths to, when you didn't pass backgrounds). Companies and people found are saved to your SnowBear account. Free for now.

## Parameters
- `companies` (array, body, required): 1 to 10 companies.
  ```json
  {
    "type": "array",
    "minItems": 1,
    "maxItems": 10,
    "items": {
      "type": [
        "string",
        "object"
      ],
      "description": "A company name, or {name, domain, linkedin_url}, or {company_id, linkedin_url} for a company already in SnowBear. Send linkedin_url whenever you can, checked first: it's the page SnowBear searches.",
      "properties": {
        "company_id": {
          "type": "string",
          "format": "uuid"
        },
        "name": {
          "type": "string"
        },
        "domain": {
          "type": "string",
          "description": "The requested company website. Preserved throughout resolution; a same-name company with another domain cannot replace it."
        },
        "linkedin_url": {
          "type": [
            "string",
            "null"
          ],
          "description": "The company's own LinkedIn page (linkedin.com/company/...). Confirm the website listed on the page is the company's domain before sending it; many companies have look-alike or old pages. It's searched even when SnowBear has another page saved."
        }
      }
    },
    "description": "1 to 10 companies."
  }
  ```
- `target` (object, body, optional): What the people are for (default: people at the company in general). Pass one whenever the backgrounds are someone else's, describing that person's goal. Null or omission uses the existing company target default. An omitted type defaults to company.
  ```json
  {
    "type": [
      "object",
      "null"
    ],
    "description": "What the people are for (default: people at the company in general). Pass one whenever the backgrounds are someone else's, describing that person's goal. Null or omission uses the existing company target default. An omitted type defaults to company. {\"type\": \"company\", \"persona\": \"VP of Engineering\"} (the default) or {\"type\": \"role\", \"title\": \"Senior Product Manager\", \"job_url\": \"...\"}.",
    "properties": {
      "type": {
        "type": "string",
        "enum": [
          "company",
          "role"
        ]
      },
      "persona": {
        "type": "string",
        "description": "Company target: who you want to reach there."
      },
      "title": {
        "type": "string",
        "description": "Role target: the job you're pursuing."
      },
      "job_url": {
        "type": "string",
        "description": "Role target: the posting's link."
      },
      "description": {
        "type": "string",
        "description": "Role target: a short description of the job."
      }
    },
    "allOf": [
      {
        "if": {
          "properties": {
            "type": {
              "enum": [
                "role"
              ]
            }
          },
          "required": [
            "type"
          ]
        },
        "then": {
          "properties": {
            "title": {
              "type": "string",
              "minLength": 1
            }
          },
          "required": [
            "title"
          ]
        }
      }
    ]
  }
  ```
- `backgrounds` (array, body, optional): Up to 10 schools or companies to compare against instead of your own background. Name each school the way its LinkedIn page does ("UCL", not "University College London"); one that matches nobody anywhere comes back in backgrounds with note nobody_found. Null, an empty array or omission uses your saved background.
  ```json
  {
    "type": [
      "array",
      "null"
    ],
    "maxItems": 10,
    "items": {
      "type": "object",
      "properties": {
        "type": {
          "type": "string",
          "enum": [
            "school",
            "company"
          ]
        },
        "name": {
          "type": "string"
        },
        "linkedin_url": {
          "type": "string"
        },
        "domain": {
          "type": "string"
        }
      },
      "required": [
        "type",
        "name"
      ]
    },
    "description": "Up to 10 schools or companies to compare against instead of your own background. Name each school the way its LinkedIn page does (\"UCL\", not \"University College London\"); one that matches nobody anywhere comes back in backgrounds with note nobody_found. Null, an empty array or omission uses your saved background."
  }
  ```
- `locations` (array, body, optional): Where the people should be (default the United States).
  ```json
  {
    "type": "array",
    "maxItems": 10,
    "description": "Where the people should be (default the United States). Where the people should be. Each entry: a country by name, ISO code or common alias (\"United Kingdom\", \"UK\", \"GB\", \"England\", \"Brazil\", \"BR\", \"USA\"); \"Europe\" (every country in geographic Europe); or \"anywhere\" (no filter, sent alone). Cities, states and regions are refused: map them to their country (\"London\" → \"United Kingdom\"). Left out: the United States.",
    "items": {
      "type": "string"
    }
  }
  ```
- `include_linkedin_network` (boolean, body, optional): Also search your LinkedIn network at these companies (default false).

## Preconditions and effects
Saves the companies and people it finds; sends nothing. The shared-background search uses no credits. Searches the United States unless locations names countries, "Europe" or "anywhere". include_linkedin_network (the user's 1st- and 2nd-degree, mutual, connections) uses the user's LinkedIn search allowance and credits for each profile, and can take until the next business day: turn it on only when asked. Asked for mutual connections or who they know, start it right away even when their LinkedIn isn't connected: shared schools and employers need no LinkedIn, and the answer warns linkedin_not_connected. Check it with GET /warm-searches/{search_id} (check_warm_search) every next_check_after_seconds, passing next_cursor. The same request on the same day returns that search with its status.

## How to use it

- Offer help finding companies, roles or hires that meet the user's needs. For job seekers, first establish and validate the opening and its actual employer; for sales, first establish the company and buyer role. Find suitable opportunities with SnowBear or your own research tools when those targets are missing. Once a target exists, use GET /companies?domain= or q= to look up its saved ID without provider work (or reuse a known saved ID), then show GET /companies/{company_id}/warm-connections immediately, explaining each saved path and why it may help. Then use existing warm searches for further discovery. Shared schools/employers work without LinkedIn authentication; shared background does not establish acquaintance. Preserve the stated relationship owner. Optional LinkedIn network searches use existing allowances and need a covering request. General onboarding and hiring do not automatically trigger warm searches. In later job-search conversations, use SnowBear to fetch/validate job URLs, confirm employers, find warm paths, and preview relevant outreach.

- POST /warm-searches: {"companies": [...], "target"?: {...}, "backgrounds"?: [...], "locations"?: [...], "include_linkedin_network"?: false}.
  - Companies and their LinkedIn pages: for each company, send its LinkedIn page as linkedin_url whenever you can. Fictional example: {"name": "Northwind", "domain": "northwind.example", "linkedin_url": "https://www.linkedin.com/company/example-northwind"}. It's the page SnowBear searches, so the right page is what makes the search find people. Before sending it, check it's the company's own page: open or search for it, and confirm the website listed on the page is the company's domain. Many companies have look-alike or old pages (example-northwind-old vs example-northwind). When the website on the page is a short link (short.example/...), follow it to the real site. Don't guess a page from the company's name. If you can't find or confirm the page, send name and domain only; SnowBear checks its saved page itself.
  - Each company in the response shows the page searched and linkedin_url_source: yours (the page you sent), verified (SnowBear confirmed it), saved_unverified (SnowBear couldn't confirm it), or saved in the first answer (SnowBear's saved page, checked once the search runs). A page that found people is the right one. Only when a company found nobody and comes back with linkedin_url_warning: unverified or note page_unverified, tell me, find and confirm the right page, and start the search again with it. no_people_found means the page was fine and nobody there shares these backgrounds.
  - Caller-supplied linkedin_url is authoritative: SnowBear uses that page even if its website disagrees or verification is unavailable; it never substitutes a saved namesake. With only a domain, SnowBear preserves the requested domain, checks website links, then uses exact-domain research and the existing LinkedIn name-search and AI-resolver fallbacks. Indirect results such as founder posts can supply the company link. Automatically discovered pages must verify against that domain or observed redirects. requested_domain shows the original input. If a company is unresolved/not_searchable, research that domain and start a new search with its linkedin_url; do not report it as searched with zero matches. Partial results must distinguish searched companies from unsearched companies.
  - target {"type": "company", "persona": "..."} (the default) rates people by how useful they are for reaching that kind of person; without a persona, by how well they can open doors at the company in general. {"type": "role", "title": "...", "job_url": "..."} rates them for a job I'm pursuing there (its team, likely hiring manager, same function, recruiters). One target per search: for a different posting at each company, run one search per posting, or give the title without a job_url.
  - Leave backgrounds out to compare against my own schools and employers. Pass them to compare against someone else's (a candidate's); those matches stay with this search, and you should always pass a target that describes that person's goal. Name each school the way its LinkedIn page does ("UCL", not "University College London"); a background that matches nobody at any company comes back in backgrounds with note nobody_found.
  - locations: where the people should be, by country. Leave it out to search the United States. Each entry is one of: a country, by name, ISO code or common alias ("United Kingdom", "UK", "GB", "England", "Brazil", "BR", "USA"); "Europe", which covers every country in geographic Europe; or "anywhere", which means no location filter (send it alone). Cities, states and regions aren't accepted: map them to their country yourself ("London" → "United Kingdom", "Texas" → "United States"). An entry SnowBear doesn't recognize is refused with a 400 that names it, so nothing is dropped silently. When my goal is outside the US (a role in London, a team in Brazil), pass those countries; pass ["anywhere"] only when I want people anywhere. The response's locations shows exactly what was applied (normalized names); tell me when it limited the search. People whose location LinkedIn doesn't show are kept.
  - include_linkedin_network adds my 1st- and 2nd-degree connections there (mutual connections). It uses my LinkedIn search allowance and 2.5 credits per profile, and can take until the next business day, so only turn it on when I ask for my connections or mutual connections.
  - When I ask for mutual connections, who I know, or a warm way into a company, start the search right away: shared schools and employers need no LinkedIn. If my LinkedIn isn't connected (GET /linkedin), still start it; the answer carries warnings: linkedin_not_connected and the network part is skipped. Then tell me what it found and offer a link to reconnect LinkedIn (POST /linkedin/connect-link) for my mutual connections. If it answers nothing_to_run, ask me for my schools and past employers and pass them as backgrounds.
  - It answers right away with search_id, and already_known: people I already have warm paths to there. Tell me about those first. planned counts the searches: one per company for schools and one for employers (two companies with both is 4). Sending the same request again the same day returns that search with its status (still running, or finished: read its people with GET /warm-searches/{search_id}/people); one that failed, was cancelled, had nothing to run, or where a company's search failed starts fresh.

## Example request body
```json
{
  "companies": [
    {
      "name": "Northwind",
      "domain": "northwind.example",
      "linkedin_url": "https://www.linkedin.com/company/example-northwind"
    }
  ],
  "target": {
    "type": "role",
    "title": "Senior Product Manager",
    "job_url": "https://jobs.northwind.example/1234"
  },
  "backgrounds": [
    {
      "type": "school",
      "name": "Stanford University"
    }
  ],
  "locations": [
    "United States",
    "Canada"
  ]
}
```

## MCP example: start_warm_search
```json
{
  "companies": [
    {
      "name": "Northwind",
      "domain": "northwind.example",
      "linkedin_url": "https://www.linkedin.com/company/example-northwind"
    }
  ],
  "target": {
    "type": "role",
    "title": "Senior Product Manager",
    "job_url": "https://jobs.northwind.example/1234"
  },
  "backgrounds": [
    {
      "type": "school",
      "name": "Stanford University"
    }
  ],
  "locations": [
    "United States",
    "Canada"
  ]
}
```

## Example response (202 Accepted)
```json
{
  "data": {
    "search_id": "00000000-0000-4000-8000-000000000020",
    "status": "running",
    "reused": false,
    "companies": [
      {
        "company_id": "00000000-0000-4000-8000-000000000006",
        "name": "Northwind",
        "domain": "northwind.example",
        "linkedin_url": "https://www.linkedin.com/company/example-northwind",
        "linkedin_url_source": "yours"
      }
    ],
    "locations": [
      "United States",
      "Canada"
    ],
    "planned": {
      "shared_background_searches": 1,
      "linkedin_network_searches": 0
    },
    "already_known": [],
    "next_check_after_seconds": 20
  },
  "message": "Warm search started. It looks for people in United States, Canada (people whose location LinkedIn doesn't show are kept). Check it with GET /warm-searches/00000000-0000-4000-8000-000000000020 every 20 seconds and pass next_cursor back as cursor to get only the people found since your last check."
}
```

Tool: `check_warm_search`

# Check a warm search

`GET /warm-searches/{search_id}` (needs the read scope)

The search's status and progress, what happened at each company, and the people found since cursor (pass next_cursor from your last check to get only new ones; a cursor from another search is refused). Checking a running search also moves it forward, so check every next_check_after_seconds (20 while the shared-background search runs; null once it's done). status is one of: queued, running, completed, partial (a company couldn't be searched or its search failed; companies says which), failed, cancelled, nothing_to_run (no schools or employers to compare against; reason says why). companies gives each company's page searched (linkedin_url, linkedin_url_source), people_found and note: no_people_found, page_unverified (nobody found and the page couldn't be confirmed: find the right page and search again with linkedin_url), not_searchable or search_failed, with note_detail. backgrounds gives each school and employer searched with people_found and note nobody_found when it matched nobody anywhere (often a school LinkedIn names differently). When a step retries after a temporary error, status_summary says when, and next_check_after_seconds waits for it. Each person is one entry: paths (shared_school, shared_employer, connected or mutual, with the background), relation (works_here, board, investor, advisor or other_role), title at the company, also_at (their other jobs), fit (reviewed, and when reviewed networking_value 0-1 and why) and warm_score (the ranking: networking_value, how close the shared background is and the relation together). A person comes back again when they gain another path or their review settles; update them by contact_id.

## Parameters
- `search_id` (string, path, required): The warm search's id, from Start a warm search or List warm searches.
  ```json
  {
    "type": "string",
    "description": "The warm search's id, from Start a warm search or List warm searches.",
    "format": "uuid"
  }
  ```
- `cursor` (string, query, optional): next_cursor from your previous check.

## How to use it

- GET /warm-searches/{search_id}?cursor=...: check every next_check_after_seconds (20 while it runs) and pass next_cursor back each time, so you get only the people found since your last check. Checking moves the search forward, so keep checking while I'm waiting. Share people with me as they arrive rather than waiting for the end. A person comes back again when they gain another path or their review settles; update them by contact_id rather than listing them twice. Stop when next_check_after_seconds is null; if only the LinkedIn network search is left, check back in an hour. A check can take up to half a minute while the search is busy, so allow 60 seconds before timing out.
  - companies says what happened at each company (people_found, and note: no_people_found, page_unverified, not_searchable or search_failed, with note_detail); backgrounds lists each school and employer searched with people_found (note nobody_found when it matched nobody anywhere). status is queued, running, completed, partial (a company couldn't be searched, or its search failed; companies says which), failed, cancelled or nothing_to_run (no schools or employers to compare against; reason says why).

## MCP example: check_warm_search
```json
{
  "search_id": "00000000-0000-4000-8000-000000000001",
  "cursor": "WyIyMDI2LTEwLTEyVDE3OjA0OjExLjUxMiswMDowMCIsIjlkNGI2ZjBlLTFjMmEtNGIzZC04ZTVmLTZhN2I4YzlkMGUxZiJd"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "search_id": "00000000-0000-4000-8000-000000000020",
    "status": "running",
    "target": {
      "type": "role",
      "title": "Senior Product Manager"
    },
    "locations": [
      "United States"
    ],
    "created_at": "2026-10-12T17:00:02.110+00:00",
    "completed_at": null,
    "phases": {
      "shared_background": {
        "state": "running",
        "searches": 2,
        "searches_done": 1
      },
      "linkedin_network": {
        "state": "not_requested",
        "searches": 0,
        "searches_done": 0
      }
    },
    "companies": [
      {
        "company_id": "00000000-0000-4000-8000-000000000006",
        "name": "Northwind",
        "linkedin_url": "https://www.linkedin.com/company/example-northwind",
        "linkedin_url_source": "verified",
        "people_found": 1,
        "note": null
      }
    ],
    "backgrounds": [
      {
        "type": "school",
        "name": "Stanford University",
        "people_found": 1,
        "note": null
      }
    ],
    "people_found": 1,
    "people": [
      {
        "contact_id": "00000000-0000-4000-8000-000000000015",
        "name": "Jane Doe",
        "title": "Director of Product",
        "headline": "Product at Northwind | Payments",
        "company": "Northwind",
        "company_id": "00000000-0000-4000-8000-000000000006",
        "relation": "works_here",
        "also_at": [],
        "linkedin_url": "https://www.linkedin.com/in/example-jane-doe",
        "location": "San Francisco, California, United States",
        "paths": [
          {
            "type": "shared_school",
            "background": "Stanford University",
            "overlap": null,
            "mutual_connections": null
          },
          {
            "type": "shared_employer",
            "background": "Stripe",
            "overlap": null,
            "mutual_connections": null
          }
        ],
        "fit": {
          "networking_value": 0.82,
          "why": "Leads the product team the role reports into.",
          "reviewed": true
        },
        "warm_score": 0.71,
        "found_at": "2026-10-12T17:04:11.512+00:00"
      }
    ],
    "more_people_waiting": false,
    "next_check_after_seconds": 20,
    "status_summary": "Searching for shared schools and employers (1 of 2 searches finished). 1 person found so far. Check again in 20 seconds with cursor set to next_cursor."
  },
  "next_cursor": "WyIyMDI2LTEwLTEyVDE3OjA0OjExLjUxMiswMDowMCIsIjlkNGI2ZjBlLTFjMmEtNGIzZC04ZTVmLTZhN2I4YzlkMGUxZiJd"
}
```

Tool: `api_get`

# List a warm search's people

`GET /warm-searches/{search_id}/people` (needs the read scope)

Everyone the search found, one entry per person (with all their paths), best first by warm_score (board members, investors and other non-employees rank lower than employees with similar paths), including while it's still running. Page with next_cursor while has_more is true; a cursor only continues the search it came from.

## Parameters
- `search_id` (string, path, required): The warm search's id, from Start a warm search or List warm searches.
  ```json
  {
    "type": "string",
    "description": "The warm search's id, from Start a warm search or List warm searches.",
    "format": "uuid"
  }
  ```
- `limit` (integer, query, optional): How many to return, 1 to 100 (default 100).
  ```json
  {
    "type": "integer",
    "minimum": 1,
    "maximum": 100,
    "description": "How many to return, 1 to 100 (default 100)."
  }
  ```
- `cursor` (string, query, optional): next_cursor from the previous page.

## How to use it

- GET /warm-searches/{search_id}/people: everyone found, one entry per person, best first; page with next_cursor while has_more is true. Use it for the final list: group people by company, show the best few at each (around five) with their title, how they're warm and why, and offer the rest.

## MCP example: api_get
```json
{
  "path": "/warm-searches/00000000-0000-4000-8000-000000000001/people"
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "contact_id": "00000000-0000-4000-8000-000000000015",
      "name": "Jane Doe",
      "title": "Director of Product",
      "headline": "Product at Northwind | Payments",
      "company": "Northwind",
      "company_id": "00000000-0000-4000-8000-000000000006",
      "relation": "works_here",
      "also_at": [],
      "linkedin_url": "https://www.linkedin.com/in/example-jane-doe",
      "location": "San Francisco, California, United States",
      "paths": [
        {
          "type": "shared_school",
          "background": "Stanford University",
          "overlap": null,
          "mutual_connections": null
        },
        {
          "type": "shared_employer",
          "background": "Stripe",
          "overlap": null,
          "mutual_connections": null
        }
      ],
      "fit": {
        "networking_value": 0.82,
        "why": "Leads the product team the role reports into.",
        "reviewed": true
      },
      "warm_score": 0.71,
      "found_at": "2026-10-12T17:04:11.512+00:00"
    }
  ],
  "has_more": false,
  "next_cursor": null
}
```

Tool: `api_change`

# Cancel a warm search

`POST /warm-searches/{search_id}/cancel` (needs the write scope)

Stops a running search right away, including its LinkedIn scrapes. People it already found stay in its people list. A search that already finished is left as it is: the answer gives its status and says there was nothing to cancel.

## Parameters
- `search_id` (string, path, required): The warm search's id, from Start a warm search or List warm searches.
  ```json
  {
    "type": "string",
    "description": "The warm search's id, from Start a warm search or List warm searches.",
    "format": "uuid"
  }
  ```

## How to use it

- GET /warm-searches lists my searches; POST /warm-searches/{search_id}/cancel stops one (a finished search is left as it is, and the answer says so). At most 5 can run at once (409 too_many_running names them).

## Example request body
```json
{}
```

## MCP example: api_change
```json
{
  "method": "POST",
  "path": "/warm-searches/00000000-0000-4000-8000-000000000001/cancel",
  "body": {}
}
```

## Example response (200 OK)
```json
{
  "data": {
    "search_id": "00000000-0000-4000-8000-000000000020",
    "status": "cancelled"
  },
  "message": "Search cancelled; it stops searching now. People it already found stay available with GET /warm-searches/00000000-0000-4000-8000-000000000020/people."
}
```
