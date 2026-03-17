# Getting Started with OCRskill.com

This guide walks you through obtaining an API key and setting it up for use with this skill.

## Step 1: Get a Free API Key

Run this command in your terminal:

```bash
curl https://api.ocrskill.com/get-key.json
```

You'll receive a JSON response:

```json
{ "key": "sk-abc123...", "expires": "2025-07-01T00:00:00Z", "note": "Free tier" }
```

For plain-text output (just the key, no JSON wrapper):

```bash
curl https://api.ocrskill.com/get-key
```

## Step 2: Store Your API Key

Export it as an environment variable so the skill can use it:

```bash
export OCRSKILL_API_KEY="sk-abc123..."
```

Or fetch and store it in one step:

```bash
export OCRSKILL_API_KEY=$(curl -s https://api.ocrskill.com/get-key.json | node -e "process.stdin.resume();let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>console.log(JSON.parse(d).key))")
```

To persist across sessions, add the export line to your shell profile (`~/.bashrc`, `~/.zshrc`, etc.).

## Step 3: Verify It Works

Run a quick test to confirm the key is valid:

```bash
curl -s https://api.ocrskill.com/v1/chat/completions \
  -H "Authorization: Bearer $OCRSKILL_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"LightOnOCR-2-1B","messages":[{"role":"user","content":"hello"}]}' \
  | node -e "process.stdin.resume();let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{const r=JSON.parse(d);console.log(r.choices?'Key is valid':'Error: '+JSON.stringify(r))})"
```

If you see "Key is valid", you're ready to go.

## Key Expiration

Free keys have an expiration date (check the `expires` field in the JSON response). When a key expires, request a new one with the same curl command. For longer-lived keys, visit [ocrskill.com](https://ocrskill.com).

## Next Steps

- [Step-by-Step Guide](guide.md) -- Full tutorial with practical examples
- [Output Formats](formats.md) -- Request and response JSON schemas
