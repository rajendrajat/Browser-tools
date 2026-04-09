# browser-tools

A growing collection of lightweight, browser-based utility tools for IT support, diagnostics, and text processing.

No installation. No backend. No data leaves your browser. Every tool is a single HTML file — download and open, or host on any static server.

---

## Tools

| Tool | Description | File |
|---|---|---|
| Endpoint Reachability Checker | Test if service/API endpoints are reachable from your network without opening them in the address bar | `Endpoint_Reachability_Checker.html` |
| Plain Text Converter | Strip bullets, markdown, and formatting from emails and documents to get clean plain text | `plain_text_converter.html` |

---

## Endpoint Reachability Checker

### Why this exists

Many services — Microsoft Edge Sync, Azure RMS, enterprise SSO — rely on backend API endpoints that aren't websites. They don't render anything in a browser tab, but if they're blocked by a corporate firewall or proxy, the dependent service silently fails.

When customers say *"we've already allowed those URLs"*, this tool lets you verify from their actual machine and network — not from a server-side ping that bypasses their proxy entirely.

### Features

- Add endpoints one at a time or paste a bulk list (one URL per line)
- Optional label for each entry
- Runs all checks in parallel
- Shows **Reachable**, **Blocked**, **Redirected**, and latency in ms
- Prints a blocked-URL summary after the run — ready to share with network teams
- No dependencies, no build step, works offline
- Respects dark mode

### How to use

**Option 1 — Local file**
Download `Endpoint_Reachability_Checker.html` and open it in any modern browser. No server needed.

**Option 2 — GitHub Pages (recommended for sharing with customers)**
1. Fork this repo and enable Pages under **Settings → Pages → main branch**
2. Send the customer the GitHub Pages URL
3. They open it on the affected machine, run checks, and screenshot the results

**Option 3 — Any static host**
Drop the HTML file on any web server or CDN. No server-side logic required.

### Understanding the results

| Status | Meaning |
|---|---|
| **Reachable** | Network path is open. `fetch()` completed (opaque `no-cors` responses with status 0 are also counted as reachable — normal for non-CORS APIs). |
| **Blocked** | Connection failed at the network level — firewall, proxy, or DNS dropped the request. This is the result to escalate to the network team. |
| **Redirected** | Server responded with a 3xx or a CORS intercept. Network path is likely open; authentication or routing may need attention. |
| **Not tested** | Check hasn't been run yet. |

### How it works technically

```javascript
fetch(url, { method: 'HEAD', mode: 'no-cors', cache: 'no-store' })
```

- `mode: 'no-cors'` — allows cross-origin requests without requiring CORS headers from the server. Most API/service endpoints don't serve CORS headers because they're not designed to be called from browser pages. The response will be opaque (status 0, no body), but receiving any response confirms the network path is open.
- `method: 'HEAD'` — avoids downloading response bodies; only connectivity is tested.
- 8-second timeout — requests that don't complete within 8 seconds are marked as blocked.

> **Note:** Tests run through the same network path the browser uses — including system proxies. This is intentional for enterprise network diagnostics.

### Limitations

| Limitation | Details |
|---|---|
| Browser sandbox | Cannot simulate a different application's network path — only the browser's. |
| No raw TCP/UDP | Only HTTP/HTTPS endpoints can be tested. |
| Opaque responses | Response headers and status codes are not inspectable for cross-origin requests. |
| HTTPS only | Mixed-content restrictions prevent fetching `http://` URLs from an `https://` page. Use the local file version for plain HTTP endpoints. |

### Common Microsoft / Edge endpoints to test

Paste these into the bulk input field:

```
https://edge.microsoft.com/
https://config.edge.skype.com/
https://browser.pipe.aria.microsoft.com/
https://api.aadrm.com/
https://api.aadrm.de/
https://login.microsoftonline.com/
https://login.live.com/
https://device.login.microsoftonline.com/
https://enterpriseregistration.windows.net/
https://storage.live.com/
https://api.onedrive.com/
```

---

## Plain Text Converter

### What it does

Paste any formatted content — forwarded emails, markdown documents, AI-generated text, copied web content — into the left panel and get clean, plain lines on the right. No bullets, no headers, no symbols. Updates live as you type.

### Features

- Strips bullet points and list markers (`-`, `*`, `•`, `►`, `▪`, numbered lists, lettered lists)
- Removes markdown bold, italic, headings, and hyperlinks (keeps the link label text)
- Converts HTML entities and special Unicode (`&nbsp;`, non-breaking spaces, zero-width spaces)
- Normalises smart quotes and curly apostrophes to straight
- Toggleable options — choose exactly what gets stripped
- Copy to clipboard or download as `.txt`
- Works offline, no data sent anywhere

### How to use

1. Open `plain_text_converter.html` in any modern browser
2. Paste formatted content into the **Input** box on the left
3. The **Output** box updates live
4. Toggle checkboxes to control which elements are stripped
5. Click **Copy output** or **Download .txt**

### Options

| Option | What it removes |
|---|---|
| Remove bullets & list markers | `-`, `*`, `•`, numbered and lettered lists |
| Remove blank lines | Collapses all empty lines |
| Trim leading/trailing spaces | Strips whitespace from start and end of each line |
| Strip markdown headers | Lines starting with `#`, `##`, `###`, etc. |
| Strip bold/italic | `**bold**`, `*italic*`, `__bold__`, `_italic_` |
| Strip markdown links | `[text](url)` → keeps `text` only |
| Replace &nbsp; / special spaces | Non-breaking spaces, zero-width characters, smart quotes |

### Use cases

- Cleaning up forwarded emails before pasting into a ticket or report
- Stripping markdown from AI-generated content for use in plain editors
- Preparing text for SMS, WhatsApp, or systems that don't support rich text
- Sanitising content before importing into a database or CRM

### Limitations

- Does not strip HTML tags (`<b>`, `<ul>`, `<li>`) — paste plain or markdown text, not raw HTML source
- Clipboard paste button requires browser permission (works on HTTPS or localhost)

---

## Contributing

PRs welcome. Every tool is a single self-contained HTML file with no build process — edit directly and submit.

If you have a tool idea that fits the same philosophy (single file, no backend, runs in the browser), feel free to open an issue.

---

## License

MIT
