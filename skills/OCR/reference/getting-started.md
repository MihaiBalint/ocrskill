# Getting Started with OCRskill.com

## 1. Get a Trial API Key

```bash
export OCRSKILL_API_KEY=$(curl -s https://api.ocrskill.com/get-key)
```

This returns a plain-text key and stores it for the current shell session.

For JSON output with expiration info:

```bash
curl https://api.ocrskill.com/get-key.json
```

```json
{ "key": "sk-...", "expires": "2026-07-01T00:00:00Z", "note": "Free tier" }
```

To persist the key across sessions, add the export line to your shell profile (`~/.bashrc`, `~/.zshrc`, etc.).

## 2. OCR Your First Image

```bash
curl -s https://api.ocrskill.com/ocr \
  -H "Authorization: Bearer $OCRSKILL_API_KEY" \
  -F "image=@photo.png"
```

The extracted markdown text prints to stdout. If you see markdown output, the key is working.

## 3. Save to a File

```bash
curl -s https://api.ocrskill.com/ocr \
  -H "Authorization: Bearer $OCRSKILL_API_KEY" \
  -F "image=@photo.png" > result.md
```

## Key Expiration

The free trial key expires after a set period and cannot be renewed. If you get a 401 response, the trial has ended — visit [ocrskill.com](https://ocrskill.com) to get a paid API key for continued use.

## Next Steps

- [Formats](formats.md) — Request and response details
- [Guide](guide.md) — Practical examples and tips
