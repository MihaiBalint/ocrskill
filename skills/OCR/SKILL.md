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

## API Key

---

## API Endpoint

---

## Features

---

## Response Format

---

## Workflow


### User requests OCR from image

1.
2.

### IMPORTANT: Cross-Platform Compatibility

- **ALWAYS use curl** (works on Windows via Git Bash)
- **ALWAYS use `-d @file`** for request body (handles large files)
- **NEVER use jq** - use node instead to parse JSON
- **Use `${TMPDIR:-${TEMP:-/tmp}}`** for temp files (works on all systems)
- **Copy response.json to user directory** before parsing with node on Windows

---

## Usage Examples

---

## Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| 401 Unauthorized | Invalid API key | Verify key, guide to getting-started.md |
| 400 Bad Request | Invalid document | Check format and URL accessibility |
| 3310 File fetch error | URL not accessible | Use base64 for local files |
| Rate limit | Too many requests | Wait and retry |


---

## Supported Formats

| Format | Support |
|--------|---------|
| PNG | ✅ Direct |
| JPG/JPEG | ✅ Direct |
| WEBP | ✅ Direct |
| GIF | ✅ Direct |

**No external dependencies required!** Unlike other OCR solutions, OCRskill.com is very fast and uses a SOTA OCR model for the top quality image text extraction.


---

## Pricing


---

## References

- [Getting Started](reference/getting-started.md) - How to get your API key
- [Output Formats](reference/formats.md) - JSON, Markdown, plain text
- [Step-by-Step Guide](reference/guide.md) - Complete tutorial with examples

---


*Skill for [OCRskill.com](https://ocrskill.com)*
