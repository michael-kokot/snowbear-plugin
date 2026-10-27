# Sequences: endpoint reference

Examples use fictional people and companies; identifiers are illustrative.

Tool: `api_get`

# List sequences

`GET /sequences` (needs the read scope)

Every sequence you can attach to a campaign, with its steps: yours, ones shared with your team, and SnowBear's templates. ownership says which, and editable says whether you can change it. A step's message is the text with {placeholders}; ai_instructions means SnowBear writes that message for each person. Fallback values and fallback_issues identify copy that needs manual correction. Newest first, and paged like the other lists.

## Parameters
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

- When the user chooses campaign setup, reuse the targeting already discussed and ask only for missing details. Check saved campaigns and reuse a suitable existing campaign rather than creating a duplicate. For a new campaign, propose three new contacts per weekday with settings.approval_mode:review_all so initial outreach waits for approval. Create paused unless the user's request also authorizes starting it; a saved paused campaign is not running automatic sourcing. Reuse or select a suitable outreach sequence under the existing sequence rules. Optional start/end dates are not prerequisites; use existing defaults when the user has not specified them. Offer explicit sequence_id:null for ongoing sourcing-only research without drafts, enrollments or sends when that matches the user's request. Preserve an existing campaign's settings unless the user authorizes changing them. Campaign sourcing uses SnowBear's weekday schedule; it does not require a separate agent reminder or recurring check-in.

- GET /sequences: sequences I can attach to a campaign, newest first, paged, each with steps [{position, action, label, wait_hours, subject, message, ai_instructions, fallback_message, fallback_subject, fallback_issues}] ownership (yours, shared_with_team, teammate, snowbear_template), editable (whether I can change it), and updated_at. Existing fallback_issues are for manual correction; reads never rewrite or pause outreach.

## MCP example: api_get
```json
{
  "path": "/sequences"
}
```

## Example response (200 OK)
```json
{
  "data": [
    {
      "id": "00000000-0000-4000-8000-000000000010",
      "name": "Fintech intro",
      "description": "Connect, then a personal follow-up",
      "ownership": "yours",
      "editable": true,
      "channels": {
        "linkedin": true,
        "email": true
      },
      "steps": [
        {
          "position": 1,
          "action": "send_connection",
          "label": "Send connection request",
          "wait_hours": null,
          "subject": null,
          "message": "Hi {first_name}, would love to connect.",
          "ai_instructions": null,
          "fallback_message": null,
          "fallback_subject": null,
          "fallback_issues": [],
          "pause": {
            "enabled": false,
            "auto_continue_hours": null
          }
        },
        {
          "position": 2,
          "action": "wait",
          "label": "Wait",
          "wait_hours": 48,
          "subject": null,
          "message": null,
          "ai_instructions": null,
          "fallback_message": null,
          "fallback_subject": null,
          "fallback_issues": [],
          "pause": {
            "enabled": false,
            "auto_continue_hours": null
          }
        },
        {
          "position": 3,
          "action": "send_message",
          "label": "Send message",
          "wait_hours": null,
          "subject": null,
          "message": null,
          "ai_instructions": "Thank them for connecting and mention one thing from their profile.",
          "fallback_message": "Hi, thanks for connecting. Would you be open to a quick chat?",
          "fallback_subject": null,
          "fallback_issues": [],
          "pause": {
            "enabled": false,
            "auto_continue_hours": null
          }
        },
        {
          "position": 4,
          "action": "wait",
          "label": "Wait",
          "wait_hours": 72,
          "subject": null,
          "message": null,
          "ai_instructions": null,
          "fallback_message": null,
          "fallback_subject": null,
          "fallback_issues": [],
          "pause": {
            "enabled": false,
            "auto_continue_hours": null
          }
        },
        {
          "position": 5,
          "action": "send_email",
          "label": "Send email",
          "wait_hours": null,
          "subject": "Quick idea for {company}",
          "message": "Hi {first_name}, ...",
          "ai_instructions": null,
          "fallback_message": "Hi, I wanted to share an idea. Would you be open to a quick chat?",
          "fallback_subject": "Quick idea",
          "fallback_issues": [],
          "pause": {
            "enabled": false,
            "auto_continue_hours": null
          }
        }
      ],
      "created_at": "2026-10-05T23:10:00.000+00:00",
      "updated_at": "2026-10-05T23:10:00.000+00:00"
    }
  ],
  "has_more": false,
  "next_cursor": null
}
```

Tool: `api_change`

# Create a sequence

`POST /sequences` (needs the write scope)

Creates a sequence you can attach to campaigns with sequence_id. Steps run in order. A written message can use placeholders such as {first_name} and {company}, which SnowBear fills for each person; the text around them is sent exactly as written. Or give ai_instructions and SnowBear writes the message per person. A sequence can have one connection request (its note is at most 300 characters, LinkedIn's limit, or leave it out to connect without a note), waits of up to 720 hours, and up to 20 steps. Message steps require fallback_message; email steps also require fallback_subject. Connection-request fallbacks are optional, including when a note is supplied. Fallback copy is prepared in case personalization cannot produce usable copy and, when used, sends exactly as written under existing approval rules. It must contain no { or }.

## Parameters
- `name` (string, body, required): Up to 120 characters.
  ```json
  {
    "type": "string",
    "minLength": 1,
    "maxLength": 120,
    "description": "Up to 120 characters."
  }
  ```
- `description` (string, body, optional): A short note for your team. Text is trimmed and truncated to 1000 characters. Null or omission means no description.
  ```json
  {
    "type": [
      "string",
      "null"
    ],
    "description": "A short note for your team. Text is trimmed and truncated to 1000 characters. Null or omission means no description."
  }
  ```
- `steps` (array, body, required): Each step: action (send_connection, send_message, send_email, view_profile, like_post, wait), plus message (with {placeholders}) or ai_instructions (plain guidance) for messages, subject for emails with a written message, and wait_hours for waits. Any step can also take pause, e.g. {"enabled": true, "auto_continue_hours": 48}: when someone reaches that step their outreach stops there until they're resumed, or continues by itself after auto_continue_hours (0.05 to 720; null waits for a person). Message steps require fallback_message; email steps also require fallback_subject. Connection-request fallbacks are optional, including when a note is supplied. Fallback copy is prepared in case personalization cannot produce usable copy and, when used, sends exactly as written under existing approval rules. It must contain no { or }. Include at least one non-wait action. At most one send_connection step is allowed; the native parser checks this across the full sequence.
  ```json
  {
    "type": "array",
    "minItems": 1,
    "maxItems": 20,
    "items": {
      "type": "object",
      "properties": {
        "action": {
          "type": "string",
          "enum": [
            "send_connection",
            "send_message",
            "send_email",
            "view_profile",
            "like_post",
            "wait"
          ]
        },
        "message": {
          "type": [
            "string",
            "null"
          ],
          "maxLength": 8000
        },
        "attachment_id": {
          "type": [
            "string",
            "null"
          ],
          "format": "uuid",
          "description": "One uploaded file on send_message; null removes it."
        },
        "subject": {
          "type": [
            "string",
            "null"
          ],
          "maxLength": 200
        },
        "ai_instructions": {
          "type": [
            "string",
            "null"
          ],
          "maxLength": 2000
        },
        "fallback_message": {
          "type": [
            "string",
            "null"
          ],
          "maxLength": 8000,
          "pattern": "^[^{}]*$"
        },
        "fallback_subject": {
          "type": [
            "string",
            "null"
          ],
          "maxLength": 200,
          "pattern": "^[^{}]*$"
        },
        "wait_hours": {
          "type": [
            "number",
            "null"
          ],
          "exclusiveMinimum": 0,
          "maximum": 720
        },
        "pause": {
          "type": [
            "object",
            "null"
          ],
          "properties": {
            "enabled": {
              "type": "boolean"
            },
            "auto_continue_hours": {
              "description": "Ignored when disabled; when enabled, 0.05–720 hours or null for manual resume."
            }
          },
          "required": [
            "enabled"
          ],
          "additionalProperties": false,
          "allOf": [
            {
              "if": {
                "properties": {
                  "enabled": {
                    "enum": [
                      true
                    ]
                  }
                },
                "required": [
                  "enabled"
                ]
              },
              "then": {
                "properties": {
                  "auto_continue_hours": {
                    "type": [
                      "number",
                      "null"
                    ],
                    "minimum": 0.05,
                    "maximum": 720
                  }
                }
              }
            }
          ]
        },
        "position": {
          "description": "GET-only field; ignored when sent back."
        },
        "label": {
          "description": "GET-only field; ignored when sent back."
        },
        "fallback_issues": {
          "description": "GET-only field; ignored when sent back."
        }
      },
      "required": [
        "action"
      ],
      "additionalProperties": false,
      "allOf": [
        {
          "not": {
            "properties": {
              "message": {
                "type": "string",
                "minLength": 1,
                "pattern": "\\S"
              },
              "ai_instructions": {
                "type": "string",
                "minLength": 1,
                "pattern": "\\S"
              }
            },
            "required": [
              "message",
              "ai_instructions"
            ]
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "send_message"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "wait_hours": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "send_email"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "wait_hours": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "send_connection"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "wait_hours": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "view_profile"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "wait_hours": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "like_post"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "wait_hours": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "wait"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "wait_hours": {
                "type": "number",
                "exclusiveMinimum": 0,
                "maximum": 720
              }
            },
            "required": [
              "wait_hours"
            ]
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "send_message"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "anyOf": [
              {
                "properties": {
                  "message": {
                    "type": "string",
                    "minLength": 1,
                    "pattern": "\\S"
                  }
                },
                "required": [
                  "message"
                ]
              },
              {
                "properties": {
                  "ai_instructions": {
                    "type": "string",
                    "minLength": 1,
                    "pattern": "\\S"
                  }
                },
                "required": [
                  "ai_instructions"
                ]
              }
            ],
            "properties": {
              "fallback_message": {
                "type": "string",
                "minLength": 1,
                "maxLength": 8000,
                "pattern": "\\S"
              }
            },
            "required": [
              "fallback_message"
            ]
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "send_email"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "anyOf": [
              {
                "properties": {
                  "message": {
                    "type": "string",
                    "minLength": 1,
                    "pattern": "\\S"
                  }
                },
                "required": [
                  "message"
                ]
              },
              {
                "properties": {
                  "ai_instructions": {
                    "type": "string",
                    "minLength": 1,
                    "pattern": "\\S"
                  }
                },
                "required": [
                  "ai_instructions"
                ]
              }
            ],
            "properties": {
              "fallback_message": {
                "type": "string",
                "minLength": 1,
                "maxLength": 8000,
                "pattern": "\\S"
              }
            },
            "required": [
              "fallback_message"
            ]
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "send_email"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "fallback_subject": {
                "type": "string",
                "minLength": 1,
                "maxLength": 200,
                "pattern": "\\S"
              }
            },
            "required": [
              "fallback_subject"
            ],
            "allOf": [
              {
                "if": {
                  "properties": {
                    "message": {
                      "type": "string",
                      "minLength": 1,
                      "pattern": "\\S"
                    }
                  },
                  "required": [
                    "message"
                  ]
                },
                "then": {
                  "properties": {
                    "subject": {
                      "type": "string",
                      "minLength": 1,
                      "maxLength": 200,
                      "pattern": "\\S"
                    }
                  },
                  "required": [
                    "subject"
                  ]
                }
              }
            ]
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "send_connection"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "subject": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "fallback_subject": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "send_message"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "subject": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "fallback_subject": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "view_profile"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "subject": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "fallback_subject": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "like_post"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "subject": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "fallback_subject": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "wait"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "subject": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "fallback_subject": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "view_profile"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "message": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "ai_instructions": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "fallback_message": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "like_post"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "message": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "ai_instructions": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "fallback_message": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "wait"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "message": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "ai_instructions": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "fallback_message": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "send_connection"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "message": {
                "type": [
                  "string",
                  "null"
                ],
                "maxLength": 300
              },
              "fallback_message": {
                "type": [
                  "string",
                  "null"
                ],
                "maxLength": 300
              }
            }
          }
        }
      ]
    },
    "contains": {
      "properties": {
        "action": {
          "not": {
            "enum": [
              "wait"
            ]
          }
        }
      },
      "required": [
        "action"
      ]
    },
    "description": "Each step: action (send_connection, send_message, send_email, view_profile, like_post, wait), plus message (with {placeholders}) or ai_instructions (plain guidance) for messages, subject for emails with a written message, and wait_hours for waits. Any step can also take pause, e.g. {\"enabled\": true, \"auto_continue_hours\": 48}: when someone reaches that step their outreach stops there until they're resumed, or continues by itself after auto_continue_hours (0.05 to 720; null waits for a person). Message steps require fallback_message; email steps also require fallback_subject. Connection-request fallbacks are optional, including when a note is supplied. Fallback copy is prepared in case personalization cannot produce usable copy and, when used, sends exactly as written under existing approval rules. It must contain no { or }. Include at least one non-wait action. At most one send_connection step is allowed; the native parser checks this across the full sequence."
  }
  ```

## How to use it

- POST /sequences: create a sequence.
  - Body: {"name", "description", "steps": [{"action": "send_connection" | "send_message" | "send_email" | "view_profile" | "like_post" | "wait", "message", "subject", "ai_instructions", "fallback_message", "fallback_subject", "wait_hours", "attachment_id"}]}.
  - Message steps require fallback_message; email steps also require fallback_subject. Connection-request fallbacks are optional, including when a note is supplied. Fallback copy is prepared in case personalization cannot produce usable copy and, when used, sends exactly as written under existing approval rules. It must contain no { or }.
  - A message can use placeholders like {first_name} and {company}; SnowBear fills them per person and keeps the rest of the text exactly. Or give ai_instructions instead of message: plain-language guidance (no placeholders needed) that SnowBear uses to write the message for each person.
  - Rules: at most one connection request (note up to 300 characters, or no note), emails with a written message need a subject, waits up to 720 hours, up to 20 steps.

## Example request body
```json
{
  "name": "Fintech intro",
  "description": "Connect, then a personal follow-up",
  "steps": [
    {
      "action": "send_connection",
      "message": "Hi {first_name}, would love to connect."
    },
    {
      "action": "wait",
      "wait_hours": 48
    },
    {
      "action": "send_message",
      "ai_instructions": "Thank them for connecting and mention one thing from their profile.",
      "fallback_message": "Hi, thanks for connecting. Would you be open to a quick chat?"
    },
    {
      "action": "wait",
      "wait_hours": 72
    },
    {
      "action": "send_email",
      "subject": "Quick idea for {company}",
      "message": "Hi {first_name}, ...",
      "fallback_subject": "Quick idea",
      "fallback_message": "Hi, I wanted to share an idea. Would you be open to a quick chat?"
    }
  ]
}
```

## MCP example: api_change
```json
{
  "method": "POST",
  "path": "/sequences",
  "body": {
    "name": "Fintech intro",
    "description": "Connect, then a personal follow-up",
    "steps": [
      {
        "action": "send_connection",
        "message": "Hi {first_name}, would love to connect."
      },
      {
        "action": "wait",
        "wait_hours": 48
      },
      {
        "action": "send_message",
        "ai_instructions": "Thank them for connecting and mention one thing from their profile.",
        "fallback_message": "Hi, thanks for connecting. Would you be open to a quick chat?"
      },
      {
        "action": "wait",
        "wait_hours": 72
      },
      {
        "action": "send_email",
        "subject": "Quick idea for {company}",
        "message": "Hi {first_name}, ...",
        "fallback_subject": "Quick idea",
        "fallback_message": "Hi, I wanted to share an idea. Would you be open to a quick chat?"
      }
    ]
  }
}
```

## Example response (201 Created)
```json
{
  "data": {
    "id": "00000000-0000-4000-8000-000000000010",
    "name": "Fintech intro",
    "description": "Connect, then a personal follow-up",
    "ownership": "yours",
    "editable": true,
    "channels": {
      "linkedin": true,
      "email": true
    },
    "steps": [
      {
        "position": 1,
        "action": "send_connection",
        "label": "Send connection request",
        "wait_hours": null,
        "subject": null,
        "message": "Hi {first_name}, would love to connect.",
        "ai_instructions": null,
        "fallback_message": null,
        "fallback_subject": null,
        "fallback_issues": [],
        "pause": {
          "enabled": false,
          "auto_continue_hours": null
        }
      },
      {
        "position": 2,
        "action": "wait",
        "label": "Wait",
        "wait_hours": 48,
        "subject": null,
        "message": null,
        "ai_instructions": null,
        "fallback_message": null,
        "fallback_subject": null,
        "fallback_issues": [],
        "pause": {
          "enabled": false,
          "auto_continue_hours": null
        }
      },
      {
        "position": 3,
        "action": "send_message",
        "label": "Send message",
        "wait_hours": null,
        "subject": null,
        "message": null,
        "ai_instructions": "Thank them for connecting and mention one thing from their profile.",
        "fallback_message": "Hi, thanks for connecting. Would you be open to a quick chat?",
        "fallback_subject": null,
        "fallback_issues": [],
        "pause": {
          "enabled": false,
          "auto_continue_hours": null
        }
      },
      {
        "position": 4,
        "action": "wait",
        "label": "Wait",
        "wait_hours": 72,
        "subject": null,
        "message": null,
        "ai_instructions": null,
        "fallback_message": null,
        "fallback_subject": null,
        "fallback_issues": [],
        "pause": {
          "enabled": false,
          "auto_continue_hours": null
        }
      },
      {
        "position": 5,
        "action": "send_email",
        "label": "Send email",
        "wait_hours": null,
        "subject": "Quick idea for {company}",
        "message": "Hi {first_name}, ...",
        "ai_instructions": null,
        "fallback_message": "Hi, I wanted to share an idea. Would you be open to a quick chat?",
        "fallback_subject": "Quick idea",
        "fallback_issues": [],
        "pause": {
          "enabled": false,
          "auto_continue_hours": null
        }
      }
    ],
    "created_at": "2026-10-05T23:10:00.000+00:00",
    "updated_at": "2026-10-05T23:10:00.000+00:00"
  }
}
```

Tool: `api_get`

# Get a sequence

`GET /sequences/{sequence_id}` (needs the read scope)

One sequence and its steps.

## Parameters
- `sequence_id` (string, path, required): The sequence's id, from GET /sequences.
  ```json
  {
    "type": "string",
    "description": "The sequence's id, from GET /sequences.",
    "format": "uuid"
  }
  ```

## How to use it

- GET /sequences/{sequence_id}: one sequence.

## MCP example: api_get
```json
{
  "path": "/sequences/00000000-0000-4000-8000-000000000001"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "id": "00000000-0000-4000-8000-000000000010",
    "name": "Fintech intro",
    "description": "Connect, then a personal follow-up",
    "ownership": "yours",
    "editable": true,
    "channels": {
      "linkedin": true,
      "email": true
    },
    "steps": [
      {
        "position": 1,
        "action": "send_connection",
        "label": "Send connection request",
        "wait_hours": null,
        "subject": null,
        "message": "Hi {first_name}, would love to connect.",
        "ai_instructions": null,
        "fallback_message": null,
        "fallback_subject": null,
        "fallback_issues": [],
        "pause": {
          "enabled": false,
          "auto_continue_hours": null
        }
      },
      {
        "position": 2,
        "action": "wait",
        "label": "Wait",
        "wait_hours": 48,
        "subject": null,
        "message": null,
        "ai_instructions": null,
        "fallback_message": null,
        "fallback_subject": null,
        "fallback_issues": [],
        "pause": {
          "enabled": false,
          "auto_continue_hours": null
        }
      },
      {
        "position": 3,
        "action": "send_message",
        "label": "Send message",
        "wait_hours": null,
        "subject": null,
        "message": null,
        "ai_instructions": "Thank them for connecting and mention one thing from their profile.",
        "fallback_message": "Hi, thanks for connecting. Would you be open to a quick chat?",
        "fallback_subject": null,
        "fallback_issues": [],
        "pause": {
          "enabled": false,
          "auto_continue_hours": null
        }
      },
      {
        "position": 4,
        "action": "wait",
        "label": "Wait",
        "wait_hours": 72,
        "subject": null,
        "message": null,
        "ai_instructions": null,
        "fallback_message": null,
        "fallback_subject": null,
        "fallback_issues": [],
        "pause": {
          "enabled": false,
          "auto_continue_hours": null
        }
      },
      {
        "position": 5,
        "action": "send_email",
        "label": "Send email",
        "wait_hours": null,
        "subject": "Quick idea for {company}",
        "message": "Hi {first_name}, ...",
        "ai_instructions": null,
        "fallback_message": "Hi, I wanted to share an idea. Would you be open to a quick chat?",
        "fallback_subject": "Quick idea",
        "fallback_issues": [],
        "pause": {
          "enabled": false,
          "auto_continue_hours": null
        }
      }
    ],
    "created_at": "2026-10-05T23:10:00.000+00:00",
    "updated_at": "2026-10-05T23:10:00.000+00:00"
  }
}
```

Tool: `api_change`

# Update a sequence

`PATCH /sequences/{sequence_id}` (needs the write scope)

Changes a sequence's name, description or steps. steps is the complete new list, in the same shape as create, and replaces the old one; you can send the steps back exactly as GET returned them, with your changes. A step that keeps its position and action also keeps the settings set in the app, such as a pause after it. The change applies wherever the sequence is used: people who haven't started get the new version (unsent drafts written from it are rewritten too, up to 25 per edit, unless someone edited one by hand), and anyone part-way through keeps the version they started. Approval isn't asked for again. changes in the response counts who was affected. You can edit your own sequences, and as a team admin your team's; SnowBear's templates can't be edited. Send expected_updated_at to make the edit fail with version_conflict if the sequence changed since you read it.

## Parameters
- `sequence_id` (string, path, required): The sequence's id, from GET /sequences.
  ```json
  {
    "type": "string",
    "description": "The sequence's id, from GET /sequences.",
    "format": "uuid"
  }
  ```
- `name` (string, body, optional): Up to 120 characters.
  ```json
  {
    "type": "string",
    "minLength": 1,
    "maxLength": 120,
    "description": "Up to 120 characters."
  }
  ```
- `description` (string, body, optional): A short note for your team. Text is trimmed and truncated to 1000 characters. Null clears it; omission leaves it unchanged.
  ```json
  {
    "type": [
      "string",
      "null"
    ],
    "description": "A short note for your team. Text is trimmed and truncated to 1000 characters. Null clears it; omission leaves it unchanged."
  }
  ```
- `steps` (array, body, optional): The complete new step list, as in Create a sequence, including pause. A step you leave pause out of keeps the pause it has if it keeps its position and action. Message steps require fallback_message; email steps also require fallback_subject. Connection-request fallbacks are optional, including when a note is supplied. Fallback copy is prepared in case personalization cannot produce usable copy and, when used, sends exactly as written under existing approval rules. It must contain no { or }. Include at least one non-wait action. At most one send_connection step is allowed; the native parser checks this across the full sequence.
  ```json
  {
    "type": "array",
    "minItems": 1,
    "maxItems": 20,
    "items": {
      "type": "object",
      "properties": {
        "action": {
          "type": "string",
          "enum": [
            "send_connection",
            "send_message",
            "send_email",
            "view_profile",
            "like_post",
            "wait"
          ]
        },
        "message": {
          "type": [
            "string",
            "null"
          ],
          "maxLength": 8000
        },
        "attachment_id": {
          "type": [
            "string",
            "null"
          ],
          "format": "uuid",
          "description": "One uploaded file on send_message; null removes it."
        },
        "subject": {
          "type": [
            "string",
            "null"
          ],
          "maxLength": 200
        },
        "ai_instructions": {
          "type": [
            "string",
            "null"
          ],
          "maxLength": 2000
        },
        "fallback_message": {
          "type": [
            "string",
            "null"
          ],
          "maxLength": 8000,
          "pattern": "^[^{}]*$"
        },
        "fallback_subject": {
          "type": [
            "string",
            "null"
          ],
          "maxLength": 200,
          "pattern": "^[^{}]*$"
        },
        "wait_hours": {
          "type": [
            "number",
            "null"
          ],
          "exclusiveMinimum": 0,
          "maximum": 720
        },
        "pause": {
          "type": [
            "object",
            "null"
          ],
          "properties": {
            "enabled": {
              "type": "boolean"
            },
            "auto_continue_hours": {
              "description": "Ignored when disabled; when enabled, 0.05–720 hours or null for manual resume."
            }
          },
          "required": [
            "enabled"
          ],
          "additionalProperties": false,
          "allOf": [
            {
              "if": {
                "properties": {
                  "enabled": {
                    "enum": [
                      true
                    ]
                  }
                },
                "required": [
                  "enabled"
                ]
              },
              "then": {
                "properties": {
                  "auto_continue_hours": {
                    "type": [
                      "number",
                      "null"
                    ],
                    "minimum": 0.05,
                    "maximum": 720
                  }
                }
              }
            }
          ]
        },
        "position": {
          "description": "GET-only field; ignored when sent back."
        },
        "label": {
          "description": "GET-only field; ignored when sent back."
        },
        "fallback_issues": {
          "description": "GET-only field; ignored when sent back."
        }
      },
      "required": [
        "action"
      ],
      "additionalProperties": false,
      "allOf": [
        {
          "not": {
            "properties": {
              "message": {
                "type": "string",
                "minLength": 1,
                "pattern": "\\S"
              },
              "ai_instructions": {
                "type": "string",
                "minLength": 1,
                "pattern": "\\S"
              }
            },
            "required": [
              "message",
              "ai_instructions"
            ]
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "send_message"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "wait_hours": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "send_email"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "wait_hours": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "send_connection"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "wait_hours": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "view_profile"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "wait_hours": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "like_post"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "wait_hours": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "wait"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "wait_hours": {
                "type": "number",
                "exclusiveMinimum": 0,
                "maximum": 720
              }
            },
            "required": [
              "wait_hours"
            ]
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "send_message"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "anyOf": [
              {
                "properties": {
                  "message": {
                    "type": "string",
                    "minLength": 1,
                    "pattern": "\\S"
                  }
                },
                "required": [
                  "message"
                ]
              },
              {
                "properties": {
                  "ai_instructions": {
                    "type": "string",
                    "minLength": 1,
                    "pattern": "\\S"
                  }
                },
                "required": [
                  "ai_instructions"
                ]
              }
            ],
            "properties": {
              "fallback_message": {
                "type": "string",
                "minLength": 1,
                "maxLength": 8000,
                "pattern": "\\S"
              }
            },
            "required": [
              "fallback_message"
            ]
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "send_email"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "anyOf": [
              {
                "properties": {
                  "message": {
                    "type": "string",
                    "minLength": 1,
                    "pattern": "\\S"
                  }
                },
                "required": [
                  "message"
                ]
              },
              {
                "properties": {
                  "ai_instructions": {
                    "type": "string",
                    "minLength": 1,
                    "pattern": "\\S"
                  }
                },
                "required": [
                  "ai_instructions"
                ]
              }
            ],
            "properties": {
              "fallback_message": {
                "type": "string",
                "minLength": 1,
                "maxLength": 8000,
                "pattern": "\\S"
              }
            },
            "required": [
              "fallback_message"
            ]
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "send_email"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "fallback_subject": {
                "type": "string",
                "minLength": 1,
                "maxLength": 200,
                "pattern": "\\S"
              }
            },
            "required": [
              "fallback_subject"
            ],
            "allOf": [
              {
                "if": {
                  "properties": {
                    "message": {
                      "type": "string",
                      "minLength": 1,
                      "pattern": "\\S"
                    }
                  },
                  "required": [
                    "message"
                  ]
                },
                "then": {
                  "properties": {
                    "subject": {
                      "type": "string",
                      "minLength": 1,
                      "maxLength": 200,
                      "pattern": "\\S"
                    }
                  },
                  "required": [
                    "subject"
                  ]
                }
              }
            ]
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "send_connection"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "subject": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "fallback_subject": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "send_message"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "subject": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "fallback_subject": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "view_profile"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "subject": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "fallback_subject": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "like_post"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "subject": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "fallback_subject": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "wait"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "subject": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "fallback_subject": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "view_profile"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "message": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "ai_instructions": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "fallback_message": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "like_post"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "message": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "ai_instructions": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "fallback_message": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "wait"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "message": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "ai_instructions": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^\\s*$"
              },
              "fallback_message": {
                "type": "null"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "action": {
                "enum": [
                  "send_connection"
                ]
              }
            },
            "required": [
              "action"
            ]
          },
          "then": {
            "properties": {
              "message": {
                "type": [
                  "string",
                  "null"
                ],
                "maxLength": 300
              },
              "fallback_message": {
                "type": [
                  "string",
                  "null"
                ],
                "maxLength": 300
              }
            }
          }
        }
      ]
    },
    "contains": {
      "properties": {
        "action": {
          "not": {
            "enum": [
              "wait"
            ]
          }
        }
      },
      "required": [
        "action"
      ]
    },
    "description": "The complete new step list, as in Create a sequence, including pause. A step you leave pause out of keeps the pause it has if it keeps its position and action. Message steps require fallback_message; email steps also require fallback_subject. Connection-request fallbacks are optional, including when a note is supplied. Fallback copy is prepared in case personalization cannot produce usable copy and, when used, sends exactly as written under existing approval rules. It must contain no { or }. Include at least one non-wait action. At most one send_connection step is allowed; the native parser checks this across the full sequence."
  }
  ```
- `expected_updated_at` (string, body, optional): The updated_at you read. The edit fails with 409 version_conflict if it has changed since. Null or omission skips the optional optimistic concurrency timestamp check.
  ```json
  {
    "type": [
      "string",
      "null"
    ],
    "description": "The updated_at you read. The edit fails with 409 version_conflict if it has changed since. Null or omission skips the optional optimistic concurrency timestamp check."
  }
  ```

## Preconditions and effects
Shared sequence edits reach people who have not started across every campaign using it, without a new approval hold. Read the sequence and identify affected campaigns before deciding the scope.

## How to use it

- PATCH /sequences/{sequence_id}: change a sequence's name, description or steps.
  - Body: any of {"name", "description", "steps"}, plus "expected_updated_at" (the updated_at you read). steps is the complete new list in the create shape and replaces the old one, so send every step, not only the changed ones.
  - Message steps require fallback_message; email steps also require fallback_subject. Connection-request fallbacks are optional, including when a note is supplied. Fallback copy is prepared in case personalization cannot produce usable copy and, when used, sends exactly as written under existing approval rules. It must contain no { or }. Full step lists validate every step. Name and description edits can leave unrelated legacy copy alone.
  - It applies everywhere the sequence is used: people who haven't started get the new version (their unsent drafts are rewritten too, up to 25 per edit), and anyone part-way through keeps the version they started.
  - Changing a shared attachment default keeps all existing contacts on their assigned version, including those who have not started. New contacts receive the new default. Contact edits stay independent.
  - The response's changes {contacts_on_new_version, drafts_refreshing, contacts_kept_on_previous_version, contacts_pause_updated, contacts_kept_own_pause, campaigns_using} says who it affected. contacts_kept_own_pause counts people with their own version of the sequence whose pause setting a pause change didn't touch.
  - Only sequences with editable true can be changed. SnowBear templates can't; create a copy with POST /sequences instead.
  - On 409 version_conflict, GET the sequence again, reapply the edit, and send the new updated_at.

## Example request body
```json
{
  "steps": [
    {
      "action": "send_connection"
    },
    {
      "action": "wait",
      "wait_hours": 48
    },
    {
      "action": "send_message",
      "message": "Hi {first_name}, thanks for connecting. Curious how {company} handles payments today?",
      "fallback_message": "Hi, thanks for connecting. Curious how your team handles payments today?"
    }
  ]
}
```

## MCP example: api_change
```json
{
  "method": "PATCH",
  "path": "/sequences/00000000-0000-4000-8000-000000000001",
  "body": {
    "steps": [
      {
        "action": "send_connection"
      },
      {
        "action": "wait",
        "wait_hours": 48
      },
      {
        "action": "send_message",
        "message": "Hi {first_name}, thanks for connecting. Curious how {company} handles payments today?",
        "fallback_message": "Hi, thanks for connecting. Curious how your team handles payments today?"
      }
    ]
  }
}
```

## Input requirements
```json
{
  "allOf": [
    {
      "anyOf": [
        {
          "required": [
            "name"
          ]
        },
        {
          "required": [
            "description"
          ]
        },
        {
          "required": [
            "steps"
          ]
        }
      ]
    }
  ]
}
```

## Example response (200 OK)
```json
{
  "data": {
    "id": "00000000-0000-4000-8000-000000000010",
    "name": "Fintech intro",
    "description": "Connect, then a personal follow-up",
    "ownership": "yours",
    "editable": true,
    "channels": {
      "linkedin": true,
      "email": true
    },
    "steps": [
      {
        "position": 1,
        "action": "send_connection",
        "label": "Send connection request",
        "wait_hours": null,
        "subject": null,
        "message": null,
        "ai_instructions": null,
        "fallback_message": null,
        "fallback_subject": null,
        "fallback_issues": [],
        "pause": {
          "enabled": false,
          "auto_continue_hours": null
        }
      },
      {
        "position": 2,
        "action": "wait",
        "label": "Wait",
        "wait_hours": 48,
        "subject": null,
        "message": null,
        "ai_instructions": null,
        "fallback_message": null,
        "fallback_subject": null,
        "fallback_issues": [],
        "pause": {
          "enabled": false,
          "auto_continue_hours": null
        }
      },
      {
        "position": 3,
        "action": "send_message",
        "label": "Send message",
        "wait_hours": null,
        "subject": null,
        "message": "Hi {first_name}, thanks for connecting. Curious how {company} handles payments today?",
        "ai_instructions": null,
        "fallback_message": "Hi, thanks for connecting. Curious how your team handles payments today?",
        "fallback_subject": null,
        "fallback_issues": [],
        "pause": {
          "enabled": false,
          "auto_continue_hours": null
        }
      }
    ],
    "created_at": "2026-10-05T23:10:00.000+00:00",
    "updated_at": "2026-10-06T09:12:00.000+00:00"
  },
  "changes": {
    "contacts_on_new_version": 12,
    "drafts_refreshing": 4,
    "contacts_kept_on_previous_version": 31,
    "contacts_pause_updated": 0,
    "contacts_kept_own_pause": 0,
    "campaigns_using": 2
  }
}
```
