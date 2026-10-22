## Downloadable contact exports
For a downloadable contact export, use start_contact_export and keep its export_id. Use paged contact reads for browsing or analysis, not to pull an entire audience through the model just to build a file. Reuse the selected list/campaign, filters and requested columns. Defaults are Contact ID, Name, Role/title, Company name, LinkedIn URL, City, State/region and Country; work email and company ID are not defaults. Check status once immediately, then honor next_check_after_seconds (30 while pending), with at most three checks per active turn. If the client cannot wait, or work remains pending, report its status and export_id so it can be checked later. Do not promise automatic notifications or create monitoring. Return the completed download link and exact row count; read the file contents only when requested for analysis. Never restart an accepted export to check progress or refresh a link. An ambiguous start failure is an uncertain outcome, not permission to start another job. File generation does not authorize enrichment, provider refresh, discovery or outreach changes. Cancel only when requested with POST /contact-exports/{export_id}/cancel.

# Contact exports: endpoint reference

Examples use fictional people and companies; identifiers are illustrative.

Tool: `start_contact_export`

# Start a contact CSV export

`POST /contact-exports` (needs the write scope)

Creates a private CSV document using stored data from exactly one accessible list or campaign. Defaults to Contact ID, Name, Role/title, Company name, LinkedIn URL, City, State/region and Country. Work email and company ID are not defaults. Select other stored fields by stable catalog ID. Uses SnowBear's existing asynchronous export worker. No enrichment, provider refresh, discovery or outreach changes. For a downloadable contact export, use start_contact_export and keep its export_id. Use paged contact reads for browsing or analysis, not to pull an entire audience through the model just to build a file. Reuse the selected list/campaign, filters and requested columns. Defaults are Contact ID, Name, Role/title, Company name, LinkedIn URL, City, State/region and Country; work email and company ID are not defaults. Check status once immediately, then honor next_check_after_seconds (30 while pending), with at most three checks per active turn. If the client cannot wait, or work remains pending, report its status and export_id so it can be checked later. Do not promise automatic notifications or create monitoring. Return the completed download link and exact row count; read the file contents only when requested for analysis. Never restart an accepted export to check progress or refresh a link. An ambiguous start failure is an uncertain outcome, not permission to start another job. File generation does not authorize enrichment, provider refresh, discovery or outreach changes.

## Parameters
- `list_id` (string, body, optional): An accessible list; choose exactly one list_id or campaign_id.
  ```json
  {
    "type": "string",
    "format": "uuid",
    "description": "An accessible list; choose exactly one list_id or campaign_id."
  }
  ```
- `campaign_id` (string, body, optional): An authorized campaign; choose exactly one list_id or campaign_id.
  ```json
  {
    "type": "string",
    "format": "uuid",
    "description": "An authorized campaign; choose exactly one list_id or campaign_id."
  }
  ```
- `columns` (array, body, optional): Ordered stored field IDs, including enrichment:<column_id> for custom/analysis columns. Omit for the eight core defaults; contact_id and linkedin_url are supported export fields. List-specific headers retain list identity. Conflicting contact copies are encoded as contact_id/value pairs in a cell.
  ```json
  {
    "type": "array",
    "minItems": 1,
    "maxItems": 50,
    "items": {
      "type": "string"
    },
    "description": "Ordered stored field IDs, including enrichment:<column_id> for custom/analysis columns. Omit for the eight core defaults; contact_id and linkedin_url are supported export fields. List-specific headers retain list identity. Conflicting contact copies are encoded as contact_id/value pairs in a cell."
  }
  ```
- `filters` (array, body, optional): List-only filters, identical to GET /lists/{list_id}/people.
  ```json
  {
    "type": [
      "array",
      "null"
    ],
    "maxItems": 30,
    "items": {
      "type": "object",
      "properties": {
        "field": {
          "type": "string",
          "anyOf": [
            {
              "enum": [
                "companies",
                "roles",
                "seniorities",
                "cities",
                "regions",
                "countries",
                "relationship_types",
                "relationship_strengths",
                "review_priorities",
                "manual_priorities",
                "enrichment_statuses",
                "linkedin_search_statuses",
                "linkedin_connection_degrees",
                "company_tags",
                "stages",
                "contact_owner_ids",
                "contact_type_roles"
              ]
            },
            {
              "pattern": "^enrichment:[0-9a-fA-F-]{36}$"
            }
          ]
        },
        "operator": {
          "type": "string",
          "enum": [
            "includes_any",
            "contains"
          ]
        },
        "values": {
          "type": "array",
          "minItems": 1,
          "maxItems": 100,
          "items": {
            "type": "string",
            "minLength": 1,
            "maxLength": 200
          }
        }
      },
      "required": [
        "field",
        "values"
      ],
      "additionalProperties": false,
      "allOf": [
        {
          "if": {
            "properties": {
              "operator": {
                "enum": [
                  "contains"
                ]
              }
            },
            "required": [
              "operator"
            ]
          },
          "then": {
            "properties": {
              "field": {
                "enum": [
                  "companies",
                  "roles",
                  "cities",
                  "regions",
                  "countries"
                ]
              }
            }
          }
        }
      ]
    },
    "description": "List-only filters, identical to GET /lists/{list_id}/people."
  }
  ```
- `responded` (boolean, body, optional): Campaign-only reply filter, identical to its contacts read.
- `accepted` (boolean, body, optional): Campaign-only connection acceptance filter.
- `changed_since` (timestamp, body, optional): Campaign-only ISO timestamp, identical to its contacts read. A date-only value such as 2026-10-19 is also accepted.
- `include_unenrolled` (boolean, body, optional): Campaign-only history control. Defaults false, as in the existing contacts read.

## How to use it

For a downloadable contact export, use start_contact_export and keep its export_id. Use paged contact reads for browsing or analysis, not to pull an entire audience through the model just to build a file. Reuse the selected list/campaign, filters and requested columns. Defaults are Contact ID, Name, Role/title, Company name, LinkedIn URL, City, State/region and Country; work email and company ID are not defaults. Check status once immediately, then honor next_check_after_seconds (30 while pending), with at most three checks per active turn. If the client cannot wait, or work remains pending, report its status and export_id so it can be checked later. Do not promise automatic notifications or create monitoring. Return the completed download link and exact row count; read the file contents only when requested for analysis. Never restart an accepted export to check progress or refresh a link. An ambiguous start failure is an uncertain outcome, not permission to start another job. File generation does not authorize enrichment, provider refresh, discovery or outreach changes. Cancel only when requested with POST /contact-exports/{export_id}/cancel.

## Example request body
```json
{
  "list_id": "00000000-0000-4000-8000-000000000003"
}
```

## MCP example: start_contact_export
```json
{
  "list_id": "00000000-0000-4000-8000-000000000003"
}
```

## Input requirements
```json
{
  "oneOf": [
    {
      "required": [
        "list_id"
      ],
      "not": {
        "required": [
          "campaign_id"
        ]
      }
    },
    {
      "required": [
        "campaign_id"
      ],
      "not": {
        "required": [
          "list_id"
        ]
      }
    }
  ]
}
```

## Example response (202 Accepted)
```json
{
  "data": {
    "export_id": "00000000-0000-4000-8000-000000000019",
    "status": "queued",
    "next_check_after_seconds": 30
  }
}
```

Tool: `get_contact_export`

# Check a contact CSV export

`GET /contact-exports/{export_id}` (needs the read scope)

A read-only status snapshot for your export. Returns queued, processing, completed, failed, cancel_requested or cancelled, progress and bounded errors. A completed export includes its exact row count, filename and a one-hour private download link. Reread to refresh an expired link without rebuilding the file. This read never starts, retries or advances a job. For a downloadable contact export, use start_contact_export and keep its export_id. Use paged contact reads for browsing or analysis, not to pull an entire audience through the model just to build a file. Reuse the selected list/campaign, filters and requested columns. Defaults are Contact ID, Name, Role/title, Company name, LinkedIn URL, City, State/region and Country; work email and company ID are not defaults. Check status once immediately, then honor next_check_after_seconds (30 while pending), with at most three checks per active turn. If the client cannot wait, or work remains pending, report its status and export_id so it can be checked later. Do not promise automatic notifications or create monitoring. Return the completed download link and exact row count; read the file contents only when requested for analysis. Never restart an accepted export to check progress or refresh a link. An ambiguous start failure is an uncertain outcome, not permission to start another job. File generation does not authorize enrichment, provider refresh, discovery or outreach changes.

## Parameters
- `export_id` (string, path, required): The exact export_id returned when the job was accepted.
  ```json
  {
    "type": "string",
    "format": "uuid",
    "description": "The exact export_id returned when the job was accepted."
  }
  ```

## How to use it

For a downloadable contact export, use start_contact_export and keep its export_id. Use paged contact reads for browsing or analysis, not to pull an entire audience through the model just to build a file. Reuse the selected list/campaign, filters and requested columns. Defaults are Contact ID, Name, Role/title, Company name, LinkedIn URL, City, State/region and Country; work email and company ID are not defaults. Check status once immediately, then honor next_check_after_seconds (30 while pending), with at most three checks per active turn. If the client cannot wait, or work remains pending, report its status and export_id so it can be checked later. Do not promise automatic notifications or create monitoring. Return the completed download link and exact row count; read the file contents only when requested for analysis. Never restart an accepted export to check progress or refresh a link. An ambiguous start failure is an uncertain outcome, not permission to start another job. File generation does not authorize enrichment, provider refresh, discovery or outreach changes. Cancel only when requested with POST /contact-exports/{export_id}/cancel.

## MCP example: get_contact_export
```json
{
  "export_id": "00000000-0000-4000-8000-000000000001"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "export_id": "00000000-0000-4000-8000-000000000019",
    "status": "completed",
    "processed_rows": 1005,
    "total_rows": 1005,
    "next_check_after_seconds": null,
    "filename": "contacts.csv",
    "download_url": "https://storage.example.invalid/private-export",
    "download_url_expires_in": 3600
  }
}
```

Tool: `cancel_contact_export`

# Cancel a contact CSV export

`POST /contact-exports/{export_id}/cancel` (needs the write scope)

Requests cancellation of your existing export using the native worker's cancellation and cleanup safeguards. Terminal jobs keep their terminal state. Creates no replacement export and changes no contacts or outreach.

## Parameters
- `export_id` (string, path, required): The export_id to cancel.
  ```json
  {
    "type": "string",
    "format": "uuid",
    "description": "The export_id to cancel."
  }
  ```

## How to use it

For a downloadable contact export, use start_contact_export and keep its export_id. Use paged contact reads for browsing or analysis, not to pull an entire audience through the model just to build a file. Reuse the selected list/campaign, filters and requested columns. Defaults are Contact ID, Name, Role/title, Company name, LinkedIn URL, City, State/region and Country; work email and company ID are not defaults. Check status once immediately, then honor next_check_after_seconds (30 while pending), with at most three checks per active turn. If the client cannot wait, or work remains pending, report its status and export_id so it can be checked later. Do not promise automatic notifications or create monitoring. Return the completed download link and exact row count; read the file contents only when requested for analysis. Never restart an accepted export to check progress or refresh a link. An ambiguous start failure is an uncertain outcome, not permission to start another job. File generation does not authorize enrichment, provider refresh, discovery or outreach changes. Cancel only when requested with POST /contact-exports/{export_id}/cancel.

## Example request body
```json
{}
```

## MCP example: cancel_contact_export
```json
{
  "export_id": "00000000-0000-4000-8000-000000000001"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "export_id": "00000000-0000-4000-8000-000000000019",
    "status": "cancel_requested",
    "cancellation_requested": true,
    "next_check_after_seconds": 30
  }
}
```
