# Account and team: endpoint reference

Examples use fictional people and companies; identifiers are illustrative.

Tool: `get_me`

# Check your account and setup status

`GET /me` (needs the read scope)

Use when setting up SnowBear, returning from authentication, or checking account readiness. Returns the authenticated user and scopes, is_onboarded, LinkedIn identity candidates or confirmation, selection versions, enrichment status, saved professional profile, timezone, LinkedIn connection and sending eligibility. This stored read starts no enrichment, discovery or outreach. Identity confirmation is separate from a live LinkedIn sending connection. Reading docs alone does not check these account states. The agent performs this read with its connected tools; it is not a manual API step for the user.

## How to use it

- Use GET /me for the authenticated user's identity, selection version, enrichment status, saved professional background, timezone, LinkedIn connection and sending eligibility. Use that background to personalize the conversation and suggest relevant options; goals remain the user's choice. For an already onboarded account, reuse its context without restarting identity confirmation. When identity is confirmed but enrichment is pending, explain the pending state without repeating confirmation or claiming completion. For an unconfirmed identity, present the returned candidates or ask for the missing personal LinkedIn URL. Only after the user chooses, use POST /me/linkedin with the exact option_number and presentation_version, or their supplied URL. Rejecting candidates also uses the current presentation version. Confirmation requires write access; identity completion and LinkedIn reconnection do not block unrelated permitted reads. Successful matching enrichment completes external identity onboarding silently. Preserve existing conversation context; ask only for missing information. Identity completion does not require choosing a goal, list, campaign or alerts.

- During onboarding, or when the user asks for next-step recommendations after an initial prospect search, use the established goal and target audience to offer relevant options: ongoing weekday discovery through a campaign, or conversation review, drafting and requested outreach to selected people. Present relevant choices simply, personalized to the conversation, with one short sentence per option. Research alone does not authorize drafting, sending, campaign changes or increased sending volume. Introduce campaign settings and approval requirements only after the user chooses campaign setup. Until that choice, do not list daily counts, approval modes, sequence choices or activation settings, or add a starter campaign proposal. Keep lists, enrichment and warm-path work inside the selected path when useful. Continue an already chosen or authorized task instead of asking the user to choose again. Honor research-only requests, connection-only requests and prior declines; identity onboarding does not require choosing either path.

- When account identity is needed for my request, GET /me confirms it and its scopes. A read-only connection cannot change outreach; reconnect with write scope when a requested change requires it.

get_me (GET /me) is the authenticated account setup check, including after sign-in. It returns account state without starting enrichment, discovery or outreach; read_docs returns documentation, not the user's setup state. No installed skill is needed for this operation.

is_onboarded means identity onboarding is complete, including previously completed native accounts; it does not mean LinkedIn sending is connected. linkedin.confirmed and linkedin.url describe identity. Unconfirmed candidates are under linkedin.candidates with option_number, name, linkedin_url and available role/company or headline evidence; candidate decisions use the returned option_number and linkedin.presentation_version with confirm_my_linkedin (POST /me/linkedin), after the user's choice. A supplied personal LinkedIn URL uses action=confirm_url. Confirmation needs write access and can start durable identity enrichment, but sends no outreach. A stale selection requires a fresh /me read, not a guessed version.

enrichment.status is ready for matching saved enrichment, ready_for_promotion for a prepared candidate, or the current preparation state such as processing or not_started. An already confirmed identity with pending enrichment needs no repeat confirmation or tight polling. profile contains saved professional background; goals remain the user's choice. timezone may be null. linkedin_connection reports the separate live sending connection; sending_eligibility reports its observed eligibility and may contain unknown values. A read does not reserve capacity or guarantee a later send. Identity completion and LinkedIn reconnection are not prerequisites for unrelated permitted saved-data reads.

A failed /me read leaves account readiness unverified even if documentation loaded. A successful account check and a failed campaign read are separate results. Missing account fields are unknown, not proof of completion or a reason to restart onboarding. Account and identity details are private authenticated data, never part of public setup metadata.

## MCP example: get_me
```json
{}
```

## Example response (200 OK)
```json
{
  "data": {
    "user_id": "00000000-0000-4000-8000-000000000024",
    "name": "Jordan Lee",
    "email": "jordan@acme.example",
    "token_id": "00000000-0000-4000-8000-000000000003",
    "scopes": [
      "read"
    ],
    "auth_type": "oauth",
    "client_name": "Example assistant",
    "timezone": "America/Los_Angeles",
    "is_onboarded": true,
    "linkedin": {
      "status": "confirmed",
      "url": "https://www.linkedin.com/in/example-jordan-lee",
      "confirmed": true,
      "candidates": [],
      "presentation_version": 2,
      "resolution_version": 1
    },
    "enrichment": {
      "status": "ready",
      "updated_at": "2026-10-01T12:00:00Z"
    },
    "profile": {
      "user_name": "Jordan Lee",
      "current_role": {
        "title": "Engineer",
        "company": "Example Company"
      }
    },
    "linkedin_connection": {
      "status": "not_connected",
      "account_name": null,
      "connected_at": null,
      "products": []
    },
    "sending_eligibility": {
      "status": "available",
      "eligible": false,
      "linkedin_connected": false,
      "paid_features_allowed": false
    }
  }
}
```

Tool: `api_get`

# List your teams

`GET /teams` (needs the read scope)

Your teams, newest first, with their confirmed members. Use a member's user_id (or email) to add them as a sender on a team campaign.

## How to use it

- GET /teams: my teams and their confirmed members (user_id, name, email, role). Use these to pick campaign senders.

## MCP example: api_get
```json
{
  "path": "/teams"
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "id": "00000000-0000-4000-8000-000000000007",
      "name": "Acme Sales",
      "your_role": "admin",
      "members": [
        {
          "user_id": "00000000-0000-4000-8000-000000000024",
          "name": "Jordan Lee",
          "email": "jordan@acme.example",
          "role": "admin"
        },
        {
          "user_id": "00000000-0000-4000-8000-000000000030",
          "name": "Sam Rivera",
          "email": "sam@acme.example",
          "role": "member"
        }
      ]
    }
  ]
}
```

Tool: `api_get`

# List a team's members

`GET /teams/{team_id}/members` (needs the read scope)

Everyone on the team with their role and LinkedIn status (connected, verifying, reconnect_required or not_connected). Team admins also see pending invitations (status invited, with who invited them and when). Any member can read their team; anyone else gets 404.

## Parameters
- `team_id` (string, path, required): The team's id, from GET /teams.
  ```json
  {
    "type": "string",
    "description": "The team's id, from GET /teams.",
    "format": "uuid"
  }
  ```

## How to use it

- GET /teams/{team_id}/members: everyone on a team with role, status (member, or invited for pending invitations, which only admins see) and linkedin {status: connected | verifying | reconnect_required | not_connected, account_name, products}. Use it to see who can send and who needs to connect LinkedIn.

## MCP example: api_get
```json
{
  "path": "/teams/00000000-0000-4000-8000-000000000001/members"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "team": {
      "id": "00000000-0000-4000-8000-000000000007",
      "name": "Acme Sales",
      "your_role": "admin"
    },
    "members": [
      {
        "user_id": "00000000-0000-4000-8000-000000000024",
        "name": "Jordan Lee",
        "email": "jordan@acme.example",
        "role": "admin",
        "status": "member",
        "is_you": true,
        "invited_at": null,
        "invited_by": null,
        "joined_at": "2026-08-11T22:42:13.676+00:00",
        "linkedin": {
          "status": "connected",
          "account_name": "Jordan Lee",
          "connected_at": "2026-07-23T05:23:54.939+00:00",
          "products": [
            "classic"
          ]
        }
      },
      {
        "user_id": "00000000-0000-4000-8000-000000000004",
        "name": "Riley Chen",
        "email": "riley@acme.example",
        "role": "member",
        "status": "invited",
        "is_you": false,
        "invited_at": "2026-10-06T23:14:02.000Z",
        "invited_by": {
          "user_id": "00000000-0000-4000-8000-000000000024",
          "name": "Jordan Lee"
        },
        "joined_at": null,
        "linkedin": {
          "status": "not_connected",
          "account_name": null,
          "connected_at": null,
          "products": []
        }
      }
    ]
  }
}
```

Tool: `api_change`

# Invite people to a team

`POST /teams/{team_id}/members` (needs the write scope)

Team admins only. Invites up to 10 people by email. Connecting LinkedIn to SnowBear counts as accepting: it confirms every pending team invitation the person has. So someone who already has LinkedIn connected joins right away (outcome added). Everyone else gets a pending invitation and, unless send_invitation is false, a short message from SnowBear (by their usual channel) asking them to reply accept or reject; they join when they accept, or as soon as they connect LinkedIn (for example through the link from POST /teams/{team_id}/members/{user_id}/linkedin-link). Someone without a SnowBear account gets one, set up for the team (team accounts are billed through your team's agreement). Outcomes: invited, added, already_invited (send resend: true to send it again), reinvited, already_member, invalid, error.

## Parameters
- `team_id` (string, path, required): The team's id, from GET /teams.
  ```json
  {
    "type": "string",
    "description": "The team's id, from GET /teams.",
    "format": "uuid"
  }
  ```
- `members` (array, body, required): Up to 10 of { email, name, role }. Only email is required: name is made from the email when it's missing, and role is member (default) or admin.
  ```json
  {
    "type": "array",
    "minItems": 1,
    "maxItems": 10,
    "items": {
      "type": "object",
      "properties": {
        "email": {
          "type": "string",
          "format": "email"
        },
        "name": {
          "type": "string"
        },
        "role": {
          "type": "string",
          "enum": [
            "member",
            "admin"
          ]
        }
      },
      "required": [
        "email"
      ],
      "additionalProperties": false
    },
    "description": "Up to 10 of { email, name, role }. Only email is required: name is made from the email when it's missing, and role is member (default) or admin."
  }
  ```
- `message` (string, body, optional): An optional personal note added to the invitation, up to 1,000 characters. The invitation is sent with or without it. Null or omission means no optional personal note.
  ```json
  {
    "type": [
      "string",
      "null"
    ],
    "maxLength": 1000,
    "description": "An optional personal note added to the invitation, up to 1,000 characters. The invitation is sent with or without it. Null or omission means no optional personal note."
  }
  ```
- `send_invitation` (boolean, body, optional): false saves the invitation without messaging them, for when you'll tell them yourself; there's no place in the app to accept an invitation, so they won't know about it otherwise. Defaults to true.
- `resend` (boolean, body, optional): true re-sends an invitation that's still pending.

## Preconditions and effects
Sends invitations unless send_invitation is false. New recipients get accounts; connecting LinkedIn can accept all pending team invitations. Membership can affect team billing and account access. At most 10 recipients. Connection links are sensitive credentials.

## How to use it

- POST /teams/{team_id}/members (team admins): {"members": [{"email", "name", "role"}], "message", "send_invitation", "resend"}, up to 10. Only email is required.
  - name is optional (SnowBear makes one from the email when it's missing); role is member (default) or admin.
  - message is an optional personal note added to SnowBear's invitation. The invitation itself is sent either way, by the person's usual channel, asking them to reply accept or reject. Don't ask me for a note or a name if I didn't give one.
  - How they join: connecting LinkedIn to SnowBear counts as accepting. It confirms every pending team invitation they have, on any team. So someone who already has LinkedIn connected joins right away (outcome added), and anyone else joins when they reply accept to SnowBear or connect LinkedIn, for example through the teammate LinkedIn link below. There's no place in the app to accept an invitation.
  - send_invitation: false saves the invitation without contacting them; use it only when I say I'll tell them myself (for example by sending them their LinkedIn link).
  - Someone new to SnowBear gets an account. resend: true sends a pending invitation again. Outcomes: invited, added, already_invited, reinvited, already_member, invalid, error.

## Example request body
```json
{
  "members": [
    {
      "email": "riley@acme.example",
      "name": "Riley Chen"
    }
  ],
  "message": "Joining so you can send for the Q4 fintech campaign."
}
```

## MCP example: api_change
```json
{
  "method": "POST",
  "path": "/teams/00000000-0000-4000-8000-000000000001/members",
  "body": {
    "members": [
      {
        "email": "riley@acme.example",
        "name": "Riley Chen"
      }
    ],
    "message": "Joining so you can send for the Q4 fintech campaign."
  }
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "index": 0,
      "outcome": "invited",
      "email": "riley@acme.example",
      "user_id": "00000000-0000-4000-8000-000000000004"
    }
  ]
}
```

Tool: `api_change`

# Cancel invitations

`POST /teams/{team_id}/members/cancel` (needs the write scope)

Team admins only. Withdraws pending invitations, by user_id or email. People who already joined are never removed here (already_member). Outcomes: cancelled, not_invited, already_member, invalid, error.

## Parameters
- `team_id` (string, path, required): The team's id, from GET /teams.
  ```json
  {
    "type": "string",
    "description": "The team's id, from GET /teams.",
    "format": "uuid"
  }
  ```
- `members` (array, body, required): Up to 25 of { user_id } or { email }.
  ```json
  {
    "type": "array",
    "minItems": 1,
    "maxItems": 25,
    "items": {
      "type": "object",
      "properties": {
        "email": {
          "type": "string"
        },
        "user_id": {
          "type": "string",
          "format": "uuid"
        }
      },
      "additionalProperties": false,
      "allOf": [
        {
          "anyOf": [
            {
              "required": [
                "email"
              ]
            },
            {
              "required": [
                "user_id"
              ]
            }
          ]
        }
      ]
    },
    "description": "Up to 25 of { user_id } or { email }."
  }
  ```

## How to use it

- POST /teams/{team_id}/members/cancel (team admins): {"members": [{"user_id"} or {"email"}]}: withdraws pending invitations. It never removes someone who joined.

## Example request body
```json
{
  "members": [
    {
      "email": "riley@acme.example"
    }
  ]
}
```

## MCP example: api_change
```json
{
  "method": "POST",
  "path": "/teams/00000000-0000-4000-8000-000000000001/members/cancel",
  "body": {
    "members": [
      {
        "email": "riley@acme.example"
      }
    ]
  }
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "index": 0,
      "outcome": "cancelled",
      "user_id": "00000000-0000-4000-8000-000000000004"
    }
  ]
}
```

Tool: `api_change`

# Create a LinkedIn link for a teammate

`POST /teams/{team_id}/members/{user_id}/linkedin-link` (needs the write scope)

Team admins only. A secure link a teammate (or someone with a pending invitation) opens to connect their own LinkedIn account to SnowBear, so they can send for team campaigns. For someone invited, connecting also accepts their invitation. Send it to them; it works once and expires (8 hours by default). Their account needs a paid plan or a team agreement (402 paid_plan_required otherwise).

## Parameters
- `team_id` (string, path, required): The team's id, from GET /teams.
  ```json
  {
    "type": "string",
    "description": "The team's id, from GET /teams.",
    "format": "uuid"
  }
  ```
- `user_id` (string, path, required): The teammate's user_id, from GET /teams/{team_id}/members.
  ```json
  {
    "type": "string",
    "description": "The teammate's user_id, from GET /teams/{team_id}/members.",
    "format": "uuid"
  }
  ```
- `products` (array, body, optional): LinkedIn products to connect: classic (always), plus sales_navigator or recruiter if they have one.
  ```json
  {
    "type": "array",
    "minItems": 1,
    "maxItems": 2,
    "items": {
      "type": "string",
      "enum": [
        "classic",
        "sales_navigator",
        "recruiter"
      ]
    },
    "not": {
      "allOf": [
        {
          "contains": {
            "enum": [
              "sales_navigator"
            ]
          }
        },
        {
          "contains": {
            "enum": [
              "recruiter"
            ]
          }
        }
      ]
    },
    "description": "LinkedIn products to connect: classic (always), plus sales_navigator or recruiter if they have one."
  }
  ```
- `expires_in_minutes` (integer, body, optional): 30 to 480, in steps of 30. Defaults to 480.
  ```json
  {
    "type": "integer",
    "minimum": 30,
    "maximum": 480,
    "multipleOf": 30,
    "description": "30 to 480, in steps of 30. Defaults to 480."
  }
  ```

## How to use it

- POST /teams/{team_id}/members/{user_id}/linkedin-link (team admins): {"products", "expires_in_minutes"}: a link a teammate or invitee opens to connect their own LinkedIn. For an invitee, connecting through it also accepts their invitation, so it's the quickest way to get someone joined and ready to send. Give it only to that person.
  - products: leave it out for regular LinkedIn, or send ["sales_navigator"] or ["recruiter"] when they have that product (Classic is always included; at most one of the two). After they connect, their linkedin.products shows what their account really has, so asking for Sales Navigator without it ends Classic-only.

## Example request body
```json
{
  "products": [
    "sales_navigator"
  ]
}
```

## MCP example: api_change
```json
{
  "method": "POST",
  "path": "/teams/00000000-0000-4000-8000-000000000001/members/00000000-0000-4000-8000-000000000001/linkedin-link",
  "body": {
    "products": [
      "sales_navigator"
    ]
  }
}
```

## Example response (201 Created)
```json
{
  "data": {
    "url": "https://auth.snowbear.ai/…",
    "type": "connect",
    "expires_at": "2026-10-07T08:28:05.822Z",
    "products": [
      "classic",
      "sales_navigator"
    ],
    "for_user_id": "00000000-0000-4000-8000-000000000004"
  }
}
```

Tool: `api_get`

# Your LinkedIn connection

`GET /linkedin` (needs the read scope)

Whether your LinkedIn account is connected (connected, verifying, reconnect_required or not_connected), which account, and which LinkedIn products. Outreach from you waits while it's reconnect_required.

## How to use it

- GET /linkedin: my own LinkedIn status. POST /linkedin/connect-link: {"products", "reconnect"}: a link to connect or reconnect my LinkedIn (type says which; products as for teammate links). Links expire after 8 hours (30 to 480 minutes for teammate links); 402 paid_plan_required means the account needs a paid plan first.

## MCP example: api_get
```json
{
  "path": "/linkedin"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "status": "connected",
    "account_name": "Jordan Lee",
    "connected_at": "2026-07-23T05:23:54.939+00:00",
    "products": [
      "classic"
    ]
  }
}
```

Tool: `api_change`

# Create your LinkedIn connection link

`POST /linkedin/connect-link` (needs the write scope)

A secure link to connect your LinkedIn account to SnowBear, or to reconnect it (type says which; SnowBear reconnects an earlier connection when there is one). Open it in a browser; it expires after 8 hours. Needs a paid plan (402 paid_plan_required otherwise). reconnect: true with nothing to reconnect is 409 nothing_to_reconnect.

## Parameters
- `products` (array, body, optional): classic (always), plus sales_navigator or recruiter if you have one.
  ```json
  {
    "type": "array",
    "minItems": 1,
    "maxItems": 2,
    "items": {
      "type": "string",
      "enum": [
        "classic",
        "sales_navigator",
        "recruiter"
      ]
    },
    "not": {
      "allOf": [
        {
          "contains": {
            "enum": [
              "sales_navigator"
            ]
          }
        },
        {
          "contains": {
            "enum": [
              "recruiter"
            ]
          }
        }
      ]
    },
    "description": "classic (always), plus sales_navigator or recruiter if you have one."
  }
  ```
- `reconnect` (boolean, body, optional): true to require reconnecting your existing connection.

## How to use it

- When the user requests conversation review, drafting or direct outreach, reuse the selected recipients and relevant saved context, including any saved contact_id already supplied. Summarize incoming messages and draft replies only when requested; a request to review or draft does not authorize saving, approval or delivery. Declining a campaign does not decline SnowBear's standalone drafting tools. When drafting is requested, call the available canonical draft_linkedin_message, draft_email or reply tool rather than writing a preview yourself. Standalone drafting needs the saved contact and intent or drafting directions; campaign_id and enrollment_id are optional, so do not require or create a campaign for a standalone preview. Previews work without a connected LinkedIn account. One-off LinkedIn messages and connection requests use send_contact_linkedin_action (POST /contacts/{contact_id}/linkedin-actions), without creating a campaign, after authorization of the authenticated sender, exact recipient, action and final text. Existing clear authorization is sufficient. If LinkedIn is disconnected or needs reconnection, explain that linking it enables SnowBear to send from the user's account. When the user has chosen LinkedIn sending, call POST /linkedin/connect-link using the exposed linkedin_connect_link tool, or the documented REST operation for an HTTP client, and show the secure connection link as the next setup step; A request to send also covers generating this necessary connection link, so do not ask another question just to generate it. For an unconnected account the existing request is POST /linkedin/connect-link with body {}; for reconnect_required, use body {reconnect:true}. Keep the chosen recipient and draft context when the user returns, verify the connection, and continue the authorized task; linking alone never approves a send or enables campaign outreach. Respect paid-plan, connection, quality, hold and allowance checks, reuse the same idempotency_key and exact body on retry, and distinguish queued from confirmed sent. Email previews are available, but email delivery uses existing campaign/sequence operations, not a standalone email-send endpoint.

- GET /linkedin: my own LinkedIn status. POST /linkedin/connect-link: {"products", "reconnect"}: a link to connect or reconnect my LinkedIn (type says which; products as for teammate links). Links expire after 8 hours (30 to 480 minutes for teammate links); 402 paid_plan_required means the account needs a paid plan first.

## Example request body
```json
{}
```

## MCP example: api_change
```json
{
  "method": "POST",
  "path": "/linkedin/connect-link",
  "body": {}
}
```

## Example response (201 Created)
```json
{
  "data": {
    "url": "https://auth.snowbear.ai/…",
    "type": "reconnect",
    "expires_at": "2026-10-07T08:28:05.822Z",
    "products": [
      "classic"
    ]
  }
}
```
