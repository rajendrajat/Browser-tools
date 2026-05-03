# browser-tools

A growing collection of lightweight, browser-based utility tools for IT support, diagnostics, and text processing.

No installation. No backend. No data leaves your browser. Every tool is a single HTML file — download and open, or host on any static server.

---

## Tools

| Tool | Description | File |
|---|---|---|
| Endpoint Reachability Checker | Test if service/API endpoints are reachable from your network without opening them in the address bar | `Endpoint_Reachability_Checker.html` |
| Plain Text Converter | Strip bullets, markdown, and formatting from emails and documents to get clean plain text | `plain_text_converter.html` |
| NetLog Analyzer | Parse and triage Edge/Chrome `net-export.json` logs — surfaces blocked endpoints, proxy issues, cert errors, DNS failures, and HTTP/2/QUIC session details without manual event-by-event inspection | `netlog-analyzer.html` |
| SAZ Analyzer Pro | Parse and analyze Fiddler `.saz` trace files — auto-detects HTTP errors, auth failures, redirect loops, connection issues, and provides URL investigation with request history, success/fail comparison, and suggested fixes | `SAZ_Analyzer.html` |

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

## NetLog Analyzer

### Why this exists

When diagnosing Edge profile sync failures, Microsoft 365 sign-in issues, or enterprise network connectivity problems, support engineers collect `net-export.json` logs from `edge://net-export/` and review them manually in the [NetLog Viewer](https://netlog-viewer.appspot.com/). That viewer surfaces raw events one by one — useful but slow when all you need to know is *what failed and why*.

This tool parses the same log format and immediately surfaces what matters: blocked endpoints, proxy errors, cert failures, DNS issues, and whether Microsoft identity/sync endpoints were affected.

### Features

- **Events tab** — all URL requests grouped by source ID, sorted by severity (Blocked → Failed → Warning → OK), with inline tags for SYNC, PROXY, CERT, DNS
- **Proxy tab** — detects whether a proxy is in use or direct, lists proxy servers seen, surfaces all proxy-specific errors, and includes a reference table of common proxy error codes with causes and fixes
- **DNS tab** — lists unresolved hosts, DNS failure count, raw DNS event failures, and DNS config snapshot
- **Sockets tab** — all TCP/SSL socket connections with host, remote IP, proxy presence, SSL protocol, bytes transferred, and socket-level errors
- **HTTP/2 tab** — session list with host, negotiated protocol, stream count, and errors
- **QUIC tab** — detects QUIC usage or its absence (with a note if UDP 443 may be blocked), packet stats, per-session errors
- **Cache tab** — hit/miss counts and cache errors
- **Meta tab** — build version, OS, capture mode, command line, total event count
- **Insight bar** — auto-surfaces suspected root causes (blocked sync hosts, proxy error count, cert error count, DNS failure count) as clickable chips that filter the table instantly
- **Copy features** — copy a structured snippet for any single request, or copy a full error summary of all blocked/failed entries for pasting into a ticket
- All analysis runs locally — no data leaves the browser

### How to use

1. Reproduce the issue in Edge while a capture is running at `edge://net-export/`
2. Stop the capture and save the `net-export.json`
3. Open `netlog-analyzer.html` in any browser
4. Drop the JSON file onto the page or use **Choose File**
5. Review the **Insight bar** at the top for suspected root causes
6. Use the tab bar to drill into Proxy, DNS, Sockets, or other areas
7. Use **Copy All Errors** or the per-row **Copy** button to extract snippets for tickets

### Understanding severity levels

| Severity | Colour | Meaning |
|---|---|---|
| **Blocked** | Red | Request was actively denied — `ERR_BLOCKED_BY_ADMINISTRATOR`, cert errors, proxy auth failures, CSP blocks, org policy |
| **Failed** | Orange | Connection could not be established — refused, timed out, DNS failure, proxy unreachable |
| **Warning** | Yellow | Request completed but returned HTTP 4xx/5xx |
| **OK** | Green | Request completed successfully |

### Row-level tags

| Tag | Colour | Meaning |
|---|---|---|
| `SYNC` | Blue | URL matches a Microsoft identity or Edge sync endpoint |
| `PROXY` | Purple | A proxy server was observed on this request |
| `CERT` | Yellow | Error is certificate or TLS related |
| `DNS` | Teal | Error is a DNS resolution failure |

### Filters available

`All` · `Blocked` · `Failed` · `Warnings` · `Sync / Auth` · `Proxy Errors` · `Cert Issues` · `DNS Issues` · `OK` · free-text search across URL and error string

### Common errors detected

| Error | Category | Typical cause |
|---|---|---|
| `ERR_BLOCKED_BY_ADMINISTRATOR` | Blocked | Firewall or Intune policy blocking the endpoint |
| `ERR_PROXY_AUTH_REQUESTED` | Proxy | Proxy requires credentials not configured in Edge |
| `ERR_TUNNEL_CONNECTION_FAILED` | Proxy | Proxy blocking HTTPS CONNECT tunnels |
| `ERR_CERT_AUTHORITY_INVALID` | Cert | SSL inspection proxy with untrusted CA |
| `ERR_NAME_NOT_RESOLVED` | DNS | Host not resolvable — split DNS, DNS filtering, or no internet |
| `ERR_CONNECTION_REFUSED` | Failed | Service or port unreachable at the network layer |

### Limitations

| Limitation | Details |
|---|---|
| Local file security | Chromium restricts some APIs when opening HTML files via `file://`. If the tool misbehaves, serve it over localhost or a static host. |
| Large logs | Very large captures (100 MB+) may be slow to parse; the tool processes everything in-memory in the browser tab. |
| Capture mode | Some event types (cache, sockets) are only present if the log was captured in **All events** mode, not the default mode. |
| No timeline view | Raw event timing is visible in the expanded row detail, but there is no graphical timeline chart. |

---

## SAZ Analyzer Pro

### Why this exists

Support engineers routinely collect Fiddler traces (`.saz` files) to troubleshoot web application issues — authentication failures, broken pages, API errors, redirect loops. The typical workflow is to open the trace in Fiddler and manually scroll through hundreds or thousands of HTTP sessions, cross-referencing URLs, status codes, headers, and cookies to find what broke.

This process is time-consuming, inconsistent, and easy to get wrong. An engineer might spend 30–60 minutes on a single trace and still miss the root cause hiding in session #347.

This tool parses the same `.saz` file and immediately tells you what's broken, why, and what to do about it.

### Features

**Three main views:**

- **Issues View (default)** — auto-detected problems sorted by severity, each with a description, affected URL, and suggested fix. Click any issue to jump straight to the session detail.
- **Sessions View** — full session table with sorting, filtering, search, and a detail panel with request/response headers and body. The manual inspection mode for when you need to dig deeper.
- **URL Investigator** — enter a URL, domain, or keyword and see every matching session, a timeline of status codes, success/fail ratio, error details, and a side-by-side header comparison between successful and failed requests.

**Smart Summary Banner** — health score (`Good` / `Warning` / `Critical`), total sessions, issue counts by severity, and the top issue — all visible the moment the file loads.

**Issue Detection Engine** — automatically scans all sessions and detects patterns across the trace, not just individual errors. See the full detection table below.

**Export** — download all detected issues as JSON for attaching to tickets or sharing with the team.

### How to use

1. Open `SAZ_Analyzer.html` in any modern browser
2. Drag and drop the `.saz` file onto the page — or click **Browse Files**
3. Check the **Issues** tab — it shows what's broken, sorted by severity
4. Click any issue card to jump to the session detail with full headers and body
5. Switch to **URL Investigator** to search for a specific domain or endpoint and see its full request history
6. Use **Sessions** view for manual line-by-line inspection when needed
7. Click **Export Issues** to download findings as JSON

### Issue detection

| Category | Severity | What it detects |
|---|---|---|
| **Server Error** | 🔴 Critical | HTTP 5xx responses — 500, 502, 503, 504, etc. |
| **Authentication** | 🔴 Critical | HTTP 401 Unauthorized — missing, expired, or invalid credentials |
| **Authorization** | 🔴 Critical | HTTP 403 Forbidden — user authenticated but lacks permission |
| **Connection / SSL** | 🔴 Critical | CONNECT tunnel failures — proxy, firewall, or certificate blocking the SSL handshake |
| **Connection** | 🔴 Critical | Status 0 / No Response — connection refused, timed out, DNS failure, or request dropped |
| **Client Error** | 🟡 Warning | HTTP 404 Not Found — broken URLs, deleted resources |
| **Client Error** | 🟡 Warning | Other 4xx errors — 400, 405, 408, 409, 429, etc. |
| **Repeated Failures** | 🟡 Warning | Same host returning errors 3+ times — potential service outage or systematic misconfiguration |
| **Redirect** | 🟡 Warning | Redirect loops — same URL redirecting to the same destination multiple times |
| **Performance** | 🔵 Info | Large responses exceeding 5 MB — may impact page load time |
| **Security** | 🔵 Info | Mixed content — HTTP requests in a predominantly HTTPS trace |

Each detected issue includes:
- Severity badge and category label
- Clear description of what went wrong
- The affected URL
- **Suggested fix / next action** — actionable guidance, not just a status code
- Clickable session references — jump to full request/response detail in one click

### URL Investigator

The URL Investigator is designed for targeted troubleshooting. Enter a URL, domain, keyword, or even a status code and get:

| Section | What it shows |
|---|---|
| **Stats** | Total matching sessions, success count, error count, redirect count, total response size |
| **Request Timeline** | Every matching session in chronological order — method, status badge, URL, and size. See the full request flow at a glance (e.g., `302 → 200 → 302 → 401`). |
| **Error Details** | All error responses grouped with severity badges, status codes, and affected URLs |
| **Success vs. Failed Comparison** | Side-by-side table comparing a successful and a failed request — status, method, size, and key headers (`Authorization`, `Cookie`, `Content-Type`, `User-Agent`, `Origin`, `Referer`). Differences are highlighted so you can instantly spot what changed. |

**Common investigation scenarios:**

| Scenario | What to search | What you'll find |
|---|---|---|
| Auth failures | `login.microsoftonline.com` or `401` | Full token request flow — where auth breaks and which headers are missing |
| API errors | `/api/` or `graph.microsoft.com` | All API calls with success/fail ratio and error breakdown |
| Redirect issues | domain name or `302` | Full redirect chain — spot loops and misconfigured redirects |
| Slow page load | `contoso.com` | All requests to the domain with sizes — find the bottleneck |
| SSO problems | `adfs` or `federation` | Federation endpoint history with status flow |

### Sessions View

The manual inspection mode for deep-dive analysis.

- **Sortable columns**: `#`, `Method`, `URL`, `Status`, `Content-Type`, `Size`
- **Quick filters**: `All`, `4xx/5xx`, `Images`, `Scripts`, `XHR/API`, `HTML`, `3xx`
- **Free-text search** across URL, method, status code, and content type
- **Detail panel** with three tabs:
  - **Overview** — request method + URL, response status, content type, size, session flags/timings from Fiddler metadata
  - **Request** — full URL, all request headers (with copy button), request body
  - **Response** — status line, all response headers (with copy button), response body with auto-formatted JSON
- Color-coded **method badges** (GET = green, POST = blue, DELETE = red, CONNECT = yellow, etc.)
- Color-coded **status badges** (2xx = green, 3xx = orange, 4xx = red, 5xx = dark red)

### How it works technically

A `.saz` file is a ZIP archive. Fiddler stores each captured HTTP session as three files inside a `raw/` directory:

```
raw/
├── 001_c.txt    # Client request — method, URL, HTTP version, headers, body
├── 001_s.txt    # Server response — status line, headers, body (binary-safe)
├── 001_m.xml    # Metadata — session flags, timings, Fiddler annotations
├── 002_c.txt
├── 002_s.txt
├── 002_m.xml
├── ...
```

The tool:
1. Extracts the ZIP in-browser using **JSZip**
2. Parses each `_c.txt` — extracts method, URL, HTTP version, headers, and body
3. Parses each `_s.txt` — extracts status code, status text, headers, and body (binary-safe using `Uint8Array`)
4. Parses each `_m.xml` — extracts session flags and timing metadata via `DOMParser`
5. Runs the **Issue Detection Engine** across all parsed sessions — detects individual errors and cross-session patterns (repeated failures, redirect loops, mixed content)
6. Renders results across three views

**Dependency:**

| Library | Version | Purpose | Loaded via |
|---|---|---|---|
| [JSZip](https://stuk.github.io/jszip/) | 3.10.1 | Client-side ZIP/SAZ parsing | CDN |

No other external dependencies. All CSS and JavaScript is inline in a single HTML file.

### Limitations

| Limitation | Details |
|---|---|
| Response body decoding | Response bodies are decoded as UTF-8 text. Binary content (images, compressed payloads) will appear garbled in the viewer. Gzip/Brotli-compressed bodies are shown in their compressed form — Fiddler typically stores them decompressed, but if not, the raw bytes are displayed. |
| Very large traces | Traces with 5,000+ sessions may take a few seconds to parse. The tool processes everything in-memory in the browser tab. |
| Chunked encoding | Chunked transfer encoding markers are shown as-is in the response body. The tool does not reassemble chunked responses. |
| CONNECT sessions | CONNECT tunnel sessions (method = CONNECT) show the tunnel negotiation, not the encrypted payload inside. This is a Fiddler limitation — the actual HTTPS content is in separate sessions. |
| No timeline chart | Request timing is available in the metadata tab but there is no graphical waterfall view (planned for a future release). |
| JSZip CDN | The tool loads JSZip from a CDN on first open. After that, the browser caches it. For fully offline use, download JSZip and embed it in the HTML file. |

### Use cases

**For support engineers:**
- Load a customer's SAZ file → check Issues tab → immediately identify auth failures, server errors, or blocked connections
- Search `401` or `login.microsoftonline.com` in URL Investigator → trace the full authentication flow
- Export issues as JSON → attach to the case notes before escalation

**For technical advisors:**
- Quick-review an engineer's trace without opening Fiddler → summary banner shows the health score and top issue
- Compare successful vs. failed requests side-by-side → spot the missing `Authorization` header or changed cookie

**For escalation engineers:**
- Drill into specific sessions with full header and body inspection
- Identify redirect loops and CONNECT failures that point to proxy/firewall misconfiguration
- Use the repeated-failures detection to confirm a service-wide outage vs. a single-user issue

---

## Contributing

PRs welcome. Every tool is a single self-contained HTML file with no build process — edit directly and submit.

If you have a tool idea that fits the same philosophy (single file, no backend, runs in the browser), feel free to open an issue.

---

## License

MIT
