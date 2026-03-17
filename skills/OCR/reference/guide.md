# Step-by-Step Guide

Complete tutorial for using OCRskill.com with practical examples.

> All examples use curl (works on Windows, macOS, Linux via Git Bash).

## Quick Start

### 1. Get Your API Key

```bash
export OCRSKILL_API_KEY=$(curl -s https://api.ocrskill.com/get-key.json | node -e "process.stdin.resume();let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>console.log(JSON.parse(d).key))")
```

### 2. OCR an Image in One Go

```bash
IMG_B64=$(base64 -i "photo.png" | tr -d '\n')

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

That's it -- the extracted markdown text prints to stdout.

## Detailed Walkthrough

### Base64 Encoding the Image

The API expects images as base64-encoded data URIs. Encoding depends on the platform:

**macOS:**

```bash
IMG_B64=$(base64 -i "image.png" | tr -d '\n')
```

**Linux (GNU coreutils):**

```bash
IMG_B64=$(base64 -w 0 "image.png")
```

**Windows (Git Bash):**

```bash
IMG_B64=$(base64 -w 0 "image.png")
```

### Choosing the Right MIME Type

Match the MIME type to the file extension in the data URI:

| Extension | Data URI prefix |
|-----------|----------------|
| `.png` | `data:image/png;base64,` |
| `.jpg` / `.jpeg` | `data:image/jpeg;base64,` |
| `.webp` | `data:image/webp;base64,` |
| `.gif` | `data:image/gif;base64,` |

### Building the Request JSON

Write the JSON payload to a temp file. This avoids shell argument-length limits when images are large.

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

### Adding a System Prompt

Use a system message to guide the extraction (e.g., focus on specific content):

```bash
cat > "$TMPFILE" <<ENDJSON
{
  "model": "LightOnOCR-2-1B",
  "messages": [
    {"role": "system", "content": "Extract all text from this image as clean markdown. Preserve headings and lists."},
    {"role": "user", "content": [
      {"type": "image_url", "image_url": {
        "url": "data:image/jpeg;base64,${IMG_B64}"
      }}
    ]}
  ]
}
ENDJSON
```

### Calling the API

```bash
RESPONSE=$(curl -s https://api.ocrskill.com/v1/chat/completions \
  -H "Authorization: Bearer $OCRSKILL_API_KEY" \
  -H "Content-Type: application/json" \
  -d @"$TMPFILE")
```

### Handling the JSON Response

Extract the markdown content using node (do not use jq -- it is not available on all systems):

```bash
echo "$RESPONSE" | node -e "
  process.stdin.resume();
  let d='';
  process.stdin.on('data', c => d += c);
  process.stdin.on('end', () => {
    const r = JSON.parse(d);
    if (r.choices && r.choices[0]) {
      console.log(r.choices[0].message.content);
    } else {
      console.error('Error:', JSON.stringify(r));
      process.exit(1);
    }
  });
"
```

### Writing Output to a File

Redirect the parsed content to a markdown file:

```bash
echo "$RESPONSE" | node -e "process.stdin.resume();let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{const r=JSON.parse(d);process.stdout.write(r.choices[0].message.content)})" > extracted.md
```

## Tips

- **Large images**: Base64 encoding roughly increases file size by 33%. Very large images work fine because the request body is sent from a file (`-d @file`), not as a shell argument.
- **Batch processing**: Loop over multiple images and write each result to a separate `.md` file.
- **Expired keys**: If you get a 401 error, your key may have expired. Request a new one with `curl https://api.ocrskill.com/get-key.json`.
- **Cleanup**: Remove the temp request file after use: `rm -f "$TMPFILE"`.
