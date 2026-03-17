# Output Formats

Request and response JSON schemas for the OCRskill.com API.

## Request Format

`POST https://api.ocrskill.com/v1/chat/completions`

Headers:

```
Authorization: Bearer <API_KEY>
Content-Type: application/json
```

### Basic request (image only)

```json
{
  "model": "LightOnOCR-2-1B",
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "image_url",
          "image_url": {
            "url": "data:<MIME_TYPE>;base64,<BASE64_DATA>"
          }
        }
      ]
    }
  ]
}
```

### Request with system prompt

```json
{
  "model": "LightOnOCR-2-1B",
  "messages": [
    {
      "role": "system",
      "content": "Extract all text from this image as clean markdown."
    },
    {
      "role": "user",
      "content": [
        {
          "type": "image_url",
          "image_url": {
            "url": "data:<MIME_TYPE>;base64,<BASE64_DATA>"
          }
        }
      ]
    }
  ]
}
```

### Field reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `model` | string | Yes | Always `LightOnOCR-2-1B` |
| `messages` | array | Yes | Array of message objects |
| `messages[].role` | string | Yes | `system` or `user` |
| `messages[].content` | string or array | Yes | Text string (for system) or content array (for user with image) |
| `image_url.url` | string | Yes | Data URI: `data:<mime>;base64,<data>` |

## Response Format

Standard OpenAI chat completion response:

```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "created": 1700000000,
  "model": "LightOnOCR-2-1B",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "# Heading\n\nExtracted text in markdown format..."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 100,
    "completion_tokens": 50,
    "total_tokens": 150
  }
}
```

### Key fields

| Field | Description |
|-------|-------------|
| `choices[0].message.content` | The extracted markdown text |
| `choices[0].finish_reason` | `stop` on success |
| `usage.total_tokens` | Total tokens consumed by the request |

### Error response

```json
{
  "error": {
    "message": "Description of what went wrong",
    "type": "error_type",
    "code": 401
  }
}
```

## Supported Image MIME Types

| Format | MIME Type | Data URI Prefix |
|--------|-----------|-----------------|
| PNG | `image/png` | `data:image/png;base64,` |
| JPG/JPEG | `image/jpeg` | `data:image/jpeg;base64,` |
| WEBP | `image/webp` | `data:image/webp;base64,` |
| GIF | `image/gif` | `data:image/gif;base64,` |
