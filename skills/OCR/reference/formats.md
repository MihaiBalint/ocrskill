# Formats

Request and response details for the OCRskill.com `/ocr` endpoint.

## Request

`POST https://api.ocrskill.com/ocr`

### Multipart upload (default)

Send the image as a multipart form field named `image`:

```bash
curl -fsS https://api.ocrskill.com/ocr \
  -H "Authorization: Bearer $OCRSKILL_API_KEY" \
  -F "image=@photo.png"
```

curl sets `Content-Type: multipart/form-data` automatically when you use `-F`. Use `-fsS` so HTTP errors fail clearly instead of being mistaken for OCR output.

### Supported image types

| Format   | Extensions       |
|----------|------------------|
| PNG      | `.png`           |
| JPEG     | `.jpg`, `.jpeg`  |
| WebP     | `.webp`          |
| GIF      | `.gif`           |

## Response

The response body is **plain markdown text** — no JSON wrapping, no parsing needed.

Example response body:

```
# Receipt

Store Name
123 Main St

Total: $18.42
```

### HTTP status codes

| Status | Meaning |
|--------|---------|
| 200 | Success — body is the extracted markdown |
| 400 | Bad request — missing or unreadable image |
| 401 | Unauthorized — invalid or expired API key; direct the user to buy a paid key |
| 429 | Rate limited — wait and retry |

### Error body

On non-200 responses the body is a short plain-text error message (not JSON), which is why clients and agents should use `curl -fsS` and stop on error instead of treating the body as OCR output.
