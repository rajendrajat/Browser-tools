# Plain Text Converter

A lightweight, browser-based tool that strips all formatting from emails, markdown documents, and rich text — giving you clean, plain lines with no bullets, headers, or symbols.

No installation required. No data leaves your browser. Works entirely offline once loaded.

---

## What It Does

Paste any formatted content into the left panel and get clean plain text on the right — instantly, as you type.

It removes:

- Bullet points and list markers (`-`, `*`, `•`, `·`, `►`, `▪`, etc.)
- Numbered and lettered lists (`1.`, `2)`, `a.`, `b)`)
- Markdown bold and italic (`**text**`, `*text*`, `__text__`, `_text_`)
- Markdown headings (`#`, `##`, `###`, etc.)
- Markdown hyperlinks (`[label](url)` → keeps the label text)
- HTML entities and special Unicode (`&nbsp;`, non-breaking spaces, zero-width spaces)
- Smart quotes and curly apostrophes (converted to straight)
- Extra blank lines and leading/trailing whitespace per line

---

## How to Use

1. Open the tool in any modern browser
2. Paste your formatted content into the **Input** box on the left
3. The **Output** box on the right updates live as you type
4. Use the checkboxes at the top to toggle which formatting elements get stripped
5. Click **Copy output** to copy the result to your clipboard
6. Click **Download .txt** to save the result as a plain text file

### Buttons

| Button | Action |
|---|---|
| Convert | Manually trigger conversion |
| Clear | Empty both input and output boxes |
| Paste from clipboard | Pull content directly from your clipboard (requires browser permission) |
| Copy output | Copy the cleaned text to your clipboard |
| Download .txt | Save the output as `plain-text.txt` |

---

## Options (Checkboxes)

Each option can be toggled independently:

| Option | What it removes |
|---|---|
| Remove bullets & list markers | `-`, `*`, `•`, numbered lists, lettered lists |
| Remove blank lines | Collapses all empty lines |
| Trim leading/trailing spaces | Strips whitespace from the start and end of each line |
| Strip markdown headers | Lines starting with `#`, `##`, `###`, etc. |
| Strip bold/italic | `**bold**`, `*italic*`, `__bold__`, `_italic_` |
| Strip markdown links | `[text](url)` → keeps `text` only |
| Replace &nbsp; / special spaces | Converts non-breaking spaces, zero-width characters, and smart quotes |

---

## Use Cases

- Cleaning up forwarded emails before pasting into a report or ticket
- Stripping markdown from AI-generated content for use in plain editors
- Preparing text for SMS, WhatsApp, or systems that do not support rich text
- Removing formatting from copied web content or documentation
- Sanitising content before importing into databases or CRMs

---

## Technical Details

- Built with plain HTML, CSS, and vanilla JavaScript — no frameworks or dependencies
- Runs entirely in the browser; no data is sent to any server
- All processing happens client-side using regular expressions
- Compatible with Chrome, Firefox, Edge, and Safari (any modern browser)
- Clipboard access uses the `navigator.clipboard` API (requires HTTPS or localhost)

---

## Limitations

- Does not parse or strip HTML tags (e.g. `<b>`, `<ul>`, `<li>`) — paste plain or markdown text, not raw HTML source
- Smart quote conversion is one-directional (curly → straight)
- The "Paste from clipboard" button requires the browser to grant clipboard read permission

---

## License

This tool is free to use, modify, and distribute for personal or commercial purposes.


===========================================================================================================================



# Endpoint Reachability Checker

A lightweight, single-file browser tool to test whether service endpoints (APIs, Microsoft/Azure URLs, etc.) are reachable from your current network — **without opening them in the browser address bar**.

---

## Why this exists

Many services like Microsoft Edge Sync, Azure RMS, or enterprise SSO rely on backend API endpoints that are not websites — they don't render anything if you visit them in a browser tab. However, if these endpoints are blocked by a corporate firewall or proxy, the dependent service will silently fail.

When customers report issues (e.g. Edge profile sync not working, RMS decryption failing), they often say *"we've already allowed those URLs"* — but it's hard to verify without a tool that tests from their actual network environment.

This tool runs `fetch()` calls directly from the user's browser, so the test happens **from their machine, through their proxy/firewall** — giving an accurate picture of what's actually reachable.

---

## Features

- Add endpoints one at a time or paste a bulk list
- Optional label for each endpoint
- Runs all checks in parallel
- Shows: **Reachable**, **Blocked**, **Redirected**, latency in ms
- Blocked summary printed after the run (can be shared with network teams)
- No dependencies, no backend, no build step
- Works offline (single HTML file)
- Respects dark mode

---

## How to use

### Option 1 — Open directly in a browser

Download `Endpoint_Reachability_Checker.html` and open it in any modern browser (Chrome, Edge, Firefox).

No server needed. Just double-click the file.

### Option 2 — GitHub Pages

1. Fork this repo
2. Go to **Settings → Pages**
3. Set source to `main` branch, root `/`
4. Access at `https://<your-username>.github.io/<repo-name>/endpoint-checker.html`

This is the recommended way to share the tool with customers — send them the GitHub Pages link and ask them to open it on the affected machine.

### Option 3 — Host on any static file server

Drop `Endpoint_Reachability_Checker.html` on any web server or CDN. No server-side logic required.

---

## Understanding the results

| Status | Meaning |
|---|---|
| **Reachable** | The network path is open. The `fetch()` completed successfully (including opaque `no-cors` responses, which are normal for non-CORS APIs). |
| **Blocked** | Connection failed at the network level — firewall, proxy, or DNS is dropping or rejecting the request. This is the actionable result to share with network teams. |
| **Redirected** | Server responded with a 3xx redirect or a CORS intercept. Usually means the network path is open but authentication or routing needs attention. |
| **Not tested** | Check hasn't been run yet. |

---

## How it works technically

The tool uses the browser's `fetch()` API with:

```javascript
fetch(url, { method: 'HEAD', mode: 'no-cors', cache: 'no-store' })
```

- `mode: 'no-cors'` — allows cross-origin requests without requiring the server to send CORS headers. This is intentional: most API/service endpoints don't serve CORS headers because they're not meant to be called from browser pages. The response will be "opaque" (status 0, no body), but the fact that a response was received at all confirms the network path is open.
- `method: 'HEAD'` — avoids downloading response bodies; we only care about connectivity.
- 8-second timeout — requests that don't complete within 8 seconds are treated as blocked.

### What a network-level block looks like

When a firewall or proxy rejects a request, the browser throws a `TypeError: Failed to fetch` (or similar). This is caught and reported as **Blocked**. A DNS failure produces the same result.

> **Note:** This tool tests from the browser's network context. If the browser itself uses a system proxy, tests will go through that proxy — which is usually the desired behaviour for enterprise network diagnostics.

---

## Limitations

| Limitation | Details |
|---|---|
| Browser sandbox | Tests run from the browser, so they are subject to the same network path the browser uses. They cannot simulate a different application's network path. |
| No raw TCP/UDP | Only HTTP/HTTPS endpoints can be tested. The tool cannot test non-HTTP ports. |
| Opaque responses | Due to `no-cors`, you cannot inspect response headers or status codes for cross-origin requests. Reachability is inferred from whether a response was received at all. |
| HTTPS only | Mixed-content restrictions in modern browsers prevent fetching plain `http://` URLs from an `https://` page. Use the local file version for testing plain HTTP endpoints. |

---

## Common Microsoft / Edge endpoints to test

These are examples — paste them into the bulk input field.

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

## Contributing

PRs welcome. The entire tool is a single HTML file with no build process — edit `endpoint-checker.html` directly.

---

## License

MIT
