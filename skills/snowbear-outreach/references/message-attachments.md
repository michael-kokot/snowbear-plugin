# LinkedIn message files: endpoint reference

Examples use fictional people and companies; identifiers are illustrative.

Tool: `prepare_message_attachment`

# Prepare a message file upload

`POST /message-attachments` (needs the write scope)

Creates a private immutable upload destination. Upload the exact file bytes with uploadToSignedUrl using the returned bucket, path and token, or PUT to signed_url. The upload credential lasts two hours. Call complete_message_attachment after uploading. One file per LinkedIn message, at most 20,000,000 bytes. Does not send, approve, schedule or pause anything.

## Parameters
- `sequence_id` (string, body, optional): Editable shared sequence whose default file this will be; omit for a personal file.
  ```json
  {
    "type": "string",
    "format": "uuid",
    "description": "Editable shared sequence whose default file this will be; omit for a personal file."
  }
  ```
- `enrollment_id` (string, body, optional): Exact contact outreach context when uploading for a sender you are authorized to manage.
  ```json
  {
    "type": "string",
    "format": "uuid",
    "description": "Exact contact outreach context when uploading for a sender you are authorized to manage."
  }
  ```
- `contact_id` (string, body, optional): Exact contact outreach context when uploading for a sender you are authorized to manage.
  ```json
  {
    "type": "string",
    "format": "uuid",
    "description": "Exact contact outreach context when uploading for a sender you are authorized to manage."
  }
  ```
- `campaign_id` (string, body, optional): Exact contact outreach context when uploading for a sender you are authorized to manage.
  ```json
  {
    "type": "string",
    "format": "uuid",
    "description": "Exact contact outreach context when uploading for a sender you are authorized to manage."
  }
  ```
- `filename` (string, body, required): Original file name.
- `mime_type` (string, body, required): MIME type, e.g. video/mp4.
- `byte_size` (integer, body, required): Exact file size in bytes.
  ```json
  {
    "type": "integer",
    "minimum": 1,
    "maximum": 20000000,
    "description": "Exact file size in bytes."
  }
  ```

## How to use it

- POST /message-attachments: prepare a private signed upload with {filename, mime_type, byte_size}, optionally sequence_id or exact contact/enrollment context. Upload one file up to 20,000,000 bytes to the returned signed destination, then POST /message-attachments/{attachment_id}/complete to verify its immutable checksum. GET /message-attachments/{attachment_id} gives a temporary preview URL. Select attachment_id on a LinkedIn Send message step or send request; omission preserves a saved file, null removes it, and a new ID replaces it. A deliberate file-only message is allowed. Connection invitations and email do not support these files. Uploading and saving never approve or send. Existing contacts keep their assigned files when a shared sequence default changes.

## Example request body
```json
{
  "filename": "introduction.mp4",
  "mime_type": "video/mp4",
  "byte_size": 1000000
}
```

## MCP example: prepare_message_attachment
```json
{
  "filename": "introduction.mp4",
  "mime_type": "video/mp4",
  "byte_size": 1000000
}
```

## Example response (201 Created)
```json
{
  "data": {
    "attachment": {
      "id": "00000000-0000-4000-8000-000000000001",
      "status": "uploading"
    },
    "upload": {
      "bucket": "linkedin-message-attachments",
      "path": "owner/file/file",
      "token": "signed-upload-token",
      "signed_url": "https://example.supabase.co/storage/v1/object/upload/sign/..."
    }
  }
}
```

Tool: `complete_message_attachment`

# Complete a message file upload

`POST /message-attachments/{attachment_id}/complete` (needs the write scope)

Verifies the uploaded file size, type and checksum, then marks it ready. Pass its attachment_id on a Send message step or authorized LinkedIn action. Repeating completion is safe. No message is sent.

## Parameters
- `attachment_id` (string, path, required): A saved message file ID.
  ```json
  {
    "type": "string",
    "format": "uuid",
    "description": "A saved message file ID."
  }
  ```
- `sequence_id` (string, body, optional): Editable shared sequence whose default file this will be; omit for a personal file.
  ```json
  {
    "type": "string",
    "format": "uuid",
    "description": "Editable shared sequence whose default file this will be; omit for a personal file."
  }
  ```
- `enrollment_id` (string, body, optional): Exact contact outreach context when uploading for a sender you are authorized to manage.
  ```json
  {
    "type": "string",
    "format": "uuid",
    "description": "Exact contact outreach context when uploading for a sender you are authorized to manage."
  }
  ```
- `contact_id` (string, body, optional): Exact contact outreach context when uploading for a sender you are authorized to manage.
  ```json
  {
    "type": "string",
    "format": "uuid",
    "description": "Exact contact outreach context when uploading for a sender you are authorized to manage."
  }
  ```
- `campaign_id` (string, body, optional): Exact contact outreach context when uploading for a sender you are authorized to manage.
  ```json
  {
    "type": "string",
    "format": "uuid",
    "description": "Exact contact outreach context when uploading for a sender you are authorized to manage."
  }
  ```

## How to use it

- POST /message-attachments: prepare a private signed upload with {filename, mime_type, byte_size}, optionally sequence_id or exact contact/enrollment context. Upload one file up to 20,000,000 bytes to the returned signed destination, then POST /message-attachments/{attachment_id}/complete to verify its immutable checksum. GET /message-attachments/{attachment_id} gives a temporary preview URL. Select attachment_id on a LinkedIn Send message step or send request; omission preserves a saved file, null removes it, and a new ID replaces it. A deliberate file-only message is allowed. Connection invitations and email do not support these files. Uploading and saving never approve or send. Existing contacts keep their assigned files when a shared sequence default changes.

## Example request body
```json
{}
```

## MCP example: complete_message_attachment
```json
{
  "attachment_id": "00000000-0000-4000-8000-000000000001"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "attachment": {
      "id": "00000000-0000-4000-8000-000000000001",
      "status": "ready",
      "filename": "introduction.mp4",
      "mime_type": "video/mp4",
      "byte_size": 1000000,
      "checksum_sha256": "sha256"
    }
  }
}
```

Tool: `get_message_attachment`

# Preview a message file

`GET /message-attachments/{attachment_id}` (needs the read scope)

Returns authorized file metadata and a private preview link valid for fifteen minutes. Request again to renew it. The file itself is immutable; replacing it requires a new upload.

## Parameters
- `attachment_id` (string, path, required): A saved message file ID.
  ```json
  {
    "type": "string",
    "format": "uuid",
    "description": "A saved message file ID."
  }
  ```
- `sequence_id` (string, query, optional): Editable shared sequence whose default file this will be; omit for a personal file.
  ```json
  {
    "type": "string",
    "format": "uuid",
    "description": "Editable shared sequence whose default file this will be; omit for a personal file."
  }
  ```
- `enrollment_id` (string, query, optional): Exact contact outreach context when uploading for a sender you are authorized to manage.
  ```json
  {
    "type": "string",
    "format": "uuid",
    "description": "Exact contact outreach context when uploading for a sender you are authorized to manage."
  }
  ```
- `contact_id` (string, query, optional): Exact contact outreach context when uploading for a sender you are authorized to manage.
  ```json
  {
    "type": "string",
    "format": "uuid",
    "description": "Exact contact outreach context when uploading for a sender you are authorized to manage."
  }
  ```
- `campaign_id` (string, query, optional): Exact contact outreach context when uploading for a sender you are authorized to manage.
  ```json
  {
    "type": "string",
    "format": "uuid",
    "description": "Exact contact outreach context when uploading for a sender you are authorized to manage."
  }
  ```

## How to use it

- POST /message-attachments: prepare a private signed upload with {filename, mime_type, byte_size}, optionally sequence_id or exact contact/enrollment context. Upload one file up to 20,000,000 bytes to the returned signed destination, then POST /message-attachments/{attachment_id}/complete to verify its immutable checksum. GET /message-attachments/{attachment_id} gives a temporary preview URL. Select attachment_id on a LinkedIn Send message step or send request; omission preserves a saved file, null removes it, and a new ID replaces it. A deliberate file-only message is allowed. Connection invitations and email do not support these files. Uploading and saving never approve or send. Existing contacts keep their assigned files when a shared sequence default changes.

## MCP example: get_message_attachment
```json
{
  "attachment_id": "00000000-0000-4000-8000-000000000001"
}
```

## Example response (200 OK)
```json
{
  "data": {
    "attachment": {
      "id": "00000000-0000-4000-8000-000000000001",
      "filename": "introduction.mp4",
      "status": "ready",
      "preview_url": "https://example.supabase.co/storage/v1/object/sign/..."
    }
  }
}
```
