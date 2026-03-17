---
name: OCR
description: Extract markdown text from images ocrskill.com API. Convert scanned documents to Markdown. No external dependencies required. Use for fast high quality OCR, extract markdown text from images.
user-invocable: true
allowed-tools: Bash(curl:*), Read, Write
metadata:
  version: "1.0.0"
---

# OCRskill.com Image text to markdown and OCR

## Requirements

- `curl` (pre-installed on macOS, Linux, and Windows Git Bash)
- `base64` CLI (pre-installed on macOS and Linux; on Windows use `certutil` or Git Bash)
- `node` (for JSON response parsing -- do not use `jq`)
- No pip, npm, or other package installs needed

## API Key

Get a free limited API key with a single curl call:

```bash
curl https://api.ocrskill.com/get-key.json
```

Response:

```json
{ "key": "sk-...", "expires": "2025-07-01T00:00:00Z", "note": "Free tier" }
```

Store the key for the session:

```bash
export OCRSKILL_API_KEY=$(curl -s https://api.ocrskill.com/get-key.json | node -e "process.stdin.resume();let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>console.log(JSON.parse(d).key))")
```

The key has an expiration date. Check the `expires` field and request a new key when needed. For plain-text output (key only, no JSON), use `curl https://api.ocrskill.com/get-key` instead.

See [Getting Started](reference/getting-started.md) for full setup instructions.

---

## API Endpoint

| Property | Value |
|----------|-------|
| URL | `https://api.ocrskill.com/v1/chat/completions` |
| Method | `POST` |
| Auth | `Authorization: Bearer <API_KEY>` |
| Model | `LightOnOCR-2-1B` |
| Compatibility | OpenAI Chat Completions API |

The API is fully OpenAI-compatible. Images are sent base64-encoded inside the `image_url` content type.

---

## Features

- Extracts text from images and returns clean Markdown
- Supports PNG, JPG/JPEG, WEBP, GIF
- State-of-the-art OCR model (LightOnOCR-2-1B) -- fast and accurate
- OpenAI SDK compatible (works with any OpenAI client library)
- Optional system prompts for guided extraction
- No external dependencies -- only `curl` and `base64`

---

## Response Format

The API returns a standard OpenAI chat completion response. The extracted markdown text is in `choices[0].message.content`.

```json
{
  "id": "chatcmpl-...",
  "object": "chat.completion",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "# Extracted Heading\n\nExtracted paragraph text from the image..."
      },
      "finish_reason": "stop"
    }
  ]
}
```

See [Output Formats](reference/formats.md) for the full request/response JSON schemas.

---

## Workflow

### User requests OCR from image

1. Locate the image file the user wants to OCR (read the path or URL from the user's request).
2. Check for an API key -- look for `$OCRSKILL_API_KEY` env var. If missing, obtain one:

```bash
export OCRSKILL_API_KEY=$(curl -s https://api.ocrskill.com/get-key.json | node -e "process.stdin.resume();let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>console.log(JSON.parse(d).key))")
```

3. Determine the MIME type from the file extension (`image/png`, `image/jpeg`, `image/webp`, `image/gif`).
4. Base64-encode the image:

```bash
IMG_B64=$(base64 -i "path/to/image.png" | tr -d '\n')
```

On Linux (GNU coreutils), use `base64 -w 0` instead of `base64 -i`.

5. Write the request JSON to a temp file (handles large payloads reliably):

```bash
TMPFILE="${TMPDIR:-${TEMP:-/tmp}}/ocr_request.json"
cat > "$TMPFILE" <<ENDJSON
{
  "model": "LightOnOCR-2-1B",
  "messages": [{
    "role": "user",
    "content": [
      {"type": "image_url", "image_url": {
        "url": "data:image/png;base64,${IMG_B64}"
      }}
    ]
  }]
}
ENDJSON
```

6. Call the API:

```bash
RESPONSE=$(curl -s https://api.ocrskill.com/v1/chat/completions \
  -H "Authorization: Bearer $OCRSKILL_API_KEY" \
  -H "Content-Type: application/json" \
  -d @"$TMPFILE")
```

7. Extract the markdown content from the response:

```bash
echo "$RESPONSE" | node -e "process.stdin.resume();let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{const r=JSON.parse(d);console.log(r.choices[0].message.content)})"
```

8. Write the result to a file or display it to the user as requested.

### IMPORTANT: Cross-Platform Compatibility

- **ALWAYS use curl** (works on Windows via Git Bash)
- **ALWAYS use `-d @file`** for request body (handles large base64 payloads without argument-length issues)
- **NEVER use jq** -- use `node -e` to parse JSON instead
- **Use `${TMPDIR:-${TEMP:-/tmp}}`** for temp files (works on macOS, Linux, and Windows)
- **Copy response.json to user directory** before parsing with node on Windows

---

## Usage Examples

### OCR a local image file

```bash
IMG_B64=$(base64 -i "screenshot.png" | tr -d '\n')

TMPFILE="${TMPDIR:-${TEMP:-/tmp}}/ocr_request.json"
cat > "$TMPFILE" <<ENDJSON
{
  "model": "LightOnOCR-2-1B",
  "messages": [{
    "role": "user",
    "content": [
      {"type": "image_url", "image_url": {
        "url": "data:image/png;base64,${IMG_B64}"
      }}
    ]
  }]
}
ENDJSON

curl -s https://api.ocrskill.com/v1/chat/completions \
  -H "Authorization: Bearer $OCRSKILL_API_KEY" \
  -H "Content-Type: application/json" \
  -d @"$TMPFILE" | node -e "process.stdin.resume();let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{const r=JSON.parse(d);console.log(r.choices[0].message.content)})"
```

### OCR with a system prompt for guided extraction

```bash
IMG_B64=$(base64 -i "document.jpg" | tr -d '\n')

TMPFILE="${TMPDIR:-${TEMP:-/tmp}}/ocr_request.json"
cat > "$TMPFILE" <<ENDJSON
{
  "model": "LightOnOCR-2-1B",
  "messages": [
    {"role": "system", "content": "Extract all text from this image as clean markdown."},
    {"role": "user", "content": [
      {"type": "image_url", "image_url": {
        "url": "data:image/jpeg;base64,${IMG_B64}"
      }}
    ]}
  ]
}
ENDJSON

curl -s https://api.ocrskill.com/v1/chat/completions \
  -H "Authorization: Bearer $OCRSKILL_API_KEY" \
  -H "Content-Type: application/json" \
  -d @"$TMPFILE" | node -e "process.stdin.resume();let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{const r=JSON.parse(d);console.log(r.choices[0].message.content)})"
```

### Save OCR result to a markdown file

```bash
curl -s https://api.ocrskill.com/v1/chat/completions \
  -H "Authorization: Bearer $OCRSKILL_API_KEY" \
  -H "Content-Type: application/json" \
  -d @"$TMPFILE" | node -e "process.stdin.resume();let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{const r=JSON.parse(d);process.stdout.write(r.choices[0].message.content)})" > output.md
```

---

## Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid or expired API key | Get a new key: `curl https://api.ocrskill.com/get-key.json` |
| 400 Bad Request | Invalid request body or image | Verify JSON structure and base64 encoding |
| 3310 File fetch error | URL not accessible | Use base64 for local files instead of URLs |
| Rate limit | Too many requests | Wait a few seconds and retry |

---

## Supported Formats

| Format | MIME Type | Support |
|--------|-----------|---------|
| PNG | `image/png` | ✅ Direct |
| JPG/JPEG | `image/jpeg` | ✅ Direct |
| WEBP | `image/webp` | ✅ Direct |
| GIF | `image/gif` | ✅ Direct |

No external dependencies required. OCRskill.com uses a state-of-the-art OCR model for fast, high-quality image text extraction.

---

## Pricing

- **Free tier**: Get a limited API key instantly via `curl https://api.ocrskill.com/get-key`. Keys expire after a set period.
- **Paid plans**: Visit [ocrskill.com](https://ocrskill.com) for higher rate limits and longer-lived keys.

---

## References

- [Getting Started](reference/getting-started.md) - How to get your API key
- [Output Formats](reference/formats.md) - JSON, Markdown, plain text
- [Step-by-Step Guide](reference/guide.md) - Complete tutorial with examples

---


*Skill for [OCRskill.com](https://ocrskill.com)*
