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
| MIPLog Analyzer | Parse and analyze `.miplog` files from Microsoft Edge — auto-detects MIP protection failures, profile sync issues, permission errors, certificate store problems, and generates copy-paste ready case notes | `MIPLog_Analyzer.html` |

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

# 🔍 SAZ Analyzer Pro

**Intelligent Fiddler Trace Analyzer — Identify Issues in Seconds, Not Hours**

![Version](https://img.shields.io/badge/version-1.0.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Platform](https://img.shields.io/badge/platform-Browser-orange)
![No Server](https://img.shields.io/badge/server-Not%20Required-brightgreen)
![Privacy](https://img.shields.io/badge/privacy-100%25%20Client--Side-purple)

---

## 📖 Overview

**SAZ Analyzer Pro** is a single-page web application that parses and analyzes Fiddler `.saz` trace files directly in your browser. It automatically detects issues, surfaces actionable insights, and helps support engineers resolve problems faster — without scrolling through hundreds of sessions manually.

> **One file. Zero installation. Zero data uploaded. Just drag, drop, and diagnose.**

---

## ❓ The Problem

Support engineers routinely collect Fiddler traces (`.saz` files) to troubleshoot web application issues. The typical workflow looks like this:

1. Open the `.saz` file in Fiddler
2. Manually scroll through **hundreds or thousands** of sessions
3. Look for red entries (errors), suspicious redirects, auth failures
4. Cross-reference URLs, headers, cookies, and status codes
5. Try to piece together what went wrong

**This process is time-consuming, error-prone, and inconsistent.** An engineer might spend 30–60 minutes analyzing a trace and still miss the root cause.

---

## ✅ The Solution

SAZ Analyzer Pro **automates the analysis**. It:

- **Parses** the entire `.saz` file in seconds
- **Detects** common issues automatically (auth failures, server errors, redirect loops, connection problems, etc.)
- **Surfaces** issues with severity, description, and **suggested actions**
- **Investigates** specific URLs showing full request history, error patterns, and header comparisons
- **Provides** manual inspection tools when you need to dig deeper

**Result: What used to take 30–60 minutes now takes 30–60 seconds.**

---

## 🚀 Key Features

### 1. 🛑 Issues View (Auto-Detection)

The **default view** when you load a SAZ file. It automatically scans every session and surfaces problems.

- Issues are categorized by **severity**: `Critical` | `Warning` | `Info`
- Each issue includes:
  - 🏷️ Severity badge and category
  - 📝 Clear description of what went wrong
  - 🔗 Affected URL
  - 🛠️ **Suggested fix / next steps**
  - 📌 Clickable link to jump to the session detail
- Filter issues by severity with one click
- Export all issues as JSON for case documentation

### 2. 📋 Sessions View (Manual Inspection)

The full session table for when you need to manually investigate.

- Sortable columns: `#`, `Method`, `URL`, `Status`, `Content-Type`, `Size`
- Quick filter buttons: `All`, `4xx/5xx`, `Images`, `Scripts`, `XHR/API`, `HTML`, `3xx`
- Free-text search across URL, method, status, and content type
- Click any session to open the **Detail Panel** with three tabs:
  - **Overview**: Request summary, response status, session flags/timings
  - **Request**: Full URL, all request headers, request body
  - **Response**: Status line, all response headers, response body (auto-formatted JSON)
- Copy headers to clipboard with one click

### 3. 🔎 URL Investigator

Enter a URL, domain, or keyword to see **everything** about it.

- **Stats Dashboard**: Total matches, success count, error count, redirects, total response size
- **Request Timeline**: Every request in chronological order with method, status, and size — see the full flow at a glance (e.g., `302 → 200 → 401`)
- **Error Details**: All errors for that URL with severity and descriptions
- **Success vs. Failed Comparison**: Side-by-side diff table comparing headers between a successful and a failed request — highlights differences in `Authorization`, `Cookie`, `User-Agent`, `Content-Type`, etc.

### 4. 📊 Smart Summary Banner

At the top of the page after loading:

- **Health Score**: `Good` ✅ | `Warning` ⚠️ | `Critical` 🔴
- Quick stats: Total sessions, critical issues, warnings, info items
- Top issue at a glance

---

## 🔎 Supported Issue Detection

| Category | Severity | What It Detects |
|---|---|---|
| **Server Error** | 🔴 Critical | HTTP 5xx responses (500, 502, 503, 504, etc.) |
| **Authentication** | 🔴 Critical | HTTP 401 Unauthorized — missing/expired tokens |
| **Authorization** | 🔴 Critical | HTTP 403 Forbidden — insufficient permissions |
| **Connection / SSL** | 🔴 Critical | CONNECT tunnel failures — proxy, firewall, or certificate issues |
| **Connection** | 🔴 Critical | Status 0 / No Response — connection refused, timeout, DNS failure |
| **Client Error** | 🟡 Warning | HTTP 404 Not Found — broken URLs, missing resources |
| **Client Error** | 🟡 Warning | Other 4xx errors (400, 405, 408, 409, 429, etc.) |
| **Repeated Failures** | 🟡 Warning | Same host failing 3+ times — potential service outage |
| **Redirect** | 🟡 Warning | Redirect loops — same redirect occurring multiple times |
| **Performance** | 🔵 Info | Large responses exceeding 5 MB |
| **Security** | 🔵 Info | Mixed content — HTTP requests in a predominantly HTTPS trace |

---

## 📦 How to Use

### Step 1: Open
Open `SAZ_Analyzer.html` in any modern browser (Edge, Chrome, Firefox, etc.).

> **No installation, no server, no dependencies to install.**

### Step 2: Load
Drag and drop your `.saz` file onto the page — or click **Browse Files**.

### Step 3: Analyze
The tool parses the file and immediately shows:
- **Health Score** in the summary banner
- **Auto-detected issues** in the Issues tab (sorted by severity)

### Step 4: Investigate
- Click any issue to **jump to the session detail**
- Switch to **URL Investigator** to search a specific domain/URL
- Use **Sessions View** for manual deep-dive when needed

### Step 5: Report
- Click **Export Issues** to download the findings as JSON
- Copy headers or session details for your case notes

---

## 🔬 URL Investigator — Detailed Capabilities

The URL Investigator is designed for scenarios like:

| Scenario | What to Search | What You'll See |
|---|---|---|
| Auth failures | `login.microsoftonline.com` | Timeline showing token requests, 302 redirects, and where auth breaks |
| API errors | `/api/` or `graph.microsoft.com` | All API calls with success/fail ratio and error details |
| Specific error | `401` or `403` | Every session with that status, grouped with fix suggestions |
| Slow page load | `contoso.com` | All requests to the domain with sizes, helping find bottlenecks |
| Redirect issues | `redirect` or specific URL | Full redirect chain visualization |

### Comparison Feature

When both successful and failed requests exist for the same search query, the tool automatically generates a **side-by-side comparison table** showing:

- Status codes
- Request methods
- Response sizes
- Key headers: `Authorization`, `Cookie`, `Content-Type`, `User-Agent`, `Origin`, `Referer`, etc.
- **Differences are highlighted** — making it easy to spot what changed between the working and broken request

---

## 📤 Export & Reporting

| Export Type | Format | Contents |
|---|---|---|
| **Issues Report** | JSON | All detected issues with severity, category, title, description, affected sessions, and suggested actions |

The exported JSON can be:
- Attached to support tickets / case notes
- Shared with team members for review
- Used as input for automated reporting pipelines

---

## 🔧 Technical Details

### Architecture
- **100% client-side** — runs entirely in the browser
- **No server required** — just open the HTML file
- **No network calls** — your data never leaves your machine
- **Single file** — everything (HTML + CSS + JS) in one file

### SAZ File Structure
A `.saz` file is a **ZIP archive** containing:

```
raw/
├── 01_c.txt    # Client request (method, URL, headers, body)
├── 01_s.txt    # Server response (status, headers, body)
├── 01_m.xml    # Metadata (session flags, timings)
├── 02_c.txt
├── 02_s.txt
├── 02_m.xml
├── ...
```

The tool:
1. Extracts the ZIP using **JSZip** library
2. Parses each `_c.txt` file for request method, URL, HTTP version, headers, and body
3. Parses each `_s.txt` file for status code, status text, headers, and body (binary-safe)
4. Parses each `_m.xml` file for session flags and timing metadata
5. Runs the **Issue Detection Engine** across all parsed sessions
6. Renders the results in the UI

### Dependencies
| Library | Version | Purpose | Loaded Via |
|---|---|---|---|
| [JSZip](https://stuk.github.io/jszip/) | 3.10.1 | ZIP/SAZ file parsing | CDN |

> No other external dependencies. All CSS and JS is inline.

---

## 🌐 Browser Compatibility

| Browser | Supported |
|---|---|
| Microsoft Edge | ✅ |
| Google Chrome | ✅ |
| Mozilla Firefox | ✅ |
| Safari | ✅ |
| Opera | ✅ |

> Requires a modern browser with ES6+ support (any browser updated in the last 5 years).

---

## 🔒 Privacy & Security

- **No data is uploaded** — all processing happens locally in your browser
- **No telemetry** — no analytics, no tracking, no cookies
- **No server** — no backend, no API calls, no external services
- **No network access needed** — works offline once the page is loaded (JSZip is cached by the browser)
- **Your traces stay on your machine** — safe for handling sensitive/internal network data

---

## 💡 Use Cases

### For Support Engineers
- **First response triage**: Load the SAZ, check the Issues tab, and immediately identify the top problem
- **Auth troubleshooting**: Search for `401` or `login` in URL Investigator to trace the full auth flow
- **Redirect debugging**: Spot redirect loops and broken redirect chains instantly
- **Performance analysis**: Find large responses and slow endpoints
- **Case documentation**: Export issues as JSON and attach to the case

### For Technical Advisors
- **Quick review**: Review an engineer's findings by looking at the Issues tab summary
- **Pattern recognition**: Repeated failures to a host? The tool flags it automatically
- **Header comparison**: Use URL Investigator to compare successful vs. failed requests side-by-side

### For Escalation Engineers
- **Root cause analysis**: Drill into specific sessions with full header and body inspection
- **SSL/TLS issues**: CONNECT tunnel failures are flagged with suggested actions
- **Cross-reference**: Search for specific domains or API endpoints across the entire trace

---

## 🗺️ Roadmap

Planned enhancements for future versions:

| Feature | Description | Priority |
|---|---|---|
| 🔐 JWT Token Decoder | Auto-decode JWT tokens from Authorization headers and show claims/expiry | High |
| 📊 Waterfall Timeline | Visual waterfall chart showing request timing and dependencies | High |
| 🍪 Cookie Tracker | Track cookies across sessions — show when they're set, modified, or missing | Medium |
| 🔄 Auth Flow Visualizer | Detect and visualize the full OAuth/SAML authentication flow | Medium |
| 📈 Response Time Analysis | Charts showing response time distribution and slowest endpoints | Medium |
| 🏷️ Custom Rules Engine | Let engineers define custom detection rules (e.g., "flag if header X is missing") | Medium |
| 📋 PDF Report Export | Generate a formatted PDF report of all findings | Low |
| 🔗 HAR File Support | Support for HTTP Archive (.har) files in addition to SAZ | Low |
| 🌙 Light/Dark Theme Toggle | Option to switch between dark and light themes | Low |
| 📦 Multi-file Comparison | Load two SAZ files and compare (before/after, working/broken) | Low |

---

## 🤝 Contributing

Contributions are welcome! Here's how you can help:

1. **Report bugs**: Open an issue describing the problem and attach a sample (sanitized) SAZ file if possible
2. **Suggest features**: Open an issue with the `enhancement` label
3. **Submit PRs**: Fork the repo, make your changes, and submit a pull request

### Development Notes
- The entire application is a single HTML file — edit it directly
- CSS is in the `<style>` block, JavaScript is in the `<script>` block
- No build tools or bundlers needed
- Test with real SAZ files from Fiddler (Classic or Everywhere)

---

# 🛡️ MIPLog Analyzer

**Edge MIP & Profile Sync Log Analyzer — From Raw Logs to Root Cause in Seconds**

![Version](https://img.shields.io/badge/version-1.0.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Platform](https://img.shields.io/badge/platform-Browser-orange)
![No Server](https://img.shields.io/badge/server-Not%20Required-brightgreen)
![Privacy](https://img.shields.io/badge/privacy-100%25%20Client--Side-purple)

---

## 📖 Overview

**MIPLog Analyzer** is a single-page web application that parses and analyzes `.miplog` files collected from Microsoft Edge. It automatically detects Microsoft Information Protection (MIP) failures, Edge profile sync issues, permission errors, and certificate problems — then generates actionable case notes ready to paste into your CRM or ICM.

> **One file. Zero installation. Zero data uploaded. Just drag, drop, and diagnose.**

---

## ❓ The Problem

When customers report issues with MIP-protected content in Microsoft Edge — files that won't open, "Access Denied" errors, profile sync failures — the support workflow typically involves:

1. Collecting `.miplog` files from the customer's machine
2. Opening them in a text editor
3. Manually scrolling through **thousands of tab-separated log lines**
4. Searching for `Error` entries and trying to interpret MIP SDK error messages
5. Cross-referencing function names, thread IDs, and timestamps to piece together the failure chain
6. Writing up findings for the case

**This process is slow, inconsistent, and easy to get wrong.** A single `.miplog` can contain thousands of trace-level entries, and the actual root cause is often buried in a chain of cascading failures.

---

## ✅ The Solution

MIPLog Analyzer **automates the entire analysis**. It:

- **Parses** the `.miplog` file instantly in the browser
- **Detects** 13 known issue patterns specific to Edge MIP and profile sync
- **Classifies** each issue by severity with root cause explanations
- **Provides** step-by-step troubleshooting guidance tailored for Edge support engineers
- **Generates** a copy-paste ready analysis report in bullet-point format for CRM/ICM case notes

**Result: What used to take 20–40 minutes of manual log reading now takes seconds.**

---

## 🚀 Key Features

### 1. 📊 Dashboard

The landing view after loading a `.miplog` file. At-a-glance summary of the entire log.

- **Summary cards**: Total entries, Errors, Warnings, Info, Trace, Unique Threads
- **Error & Warning Timeline**: Visual bar chart showing the distribution of errors and warnings over time
  - Hover over any bar to see the exact timestamp and count breakdown
  - Color-coded: Red = errors present, Orange = warnings only, Blue = info only

### 2. 🛑 Detected Issues (Pattern Engine)

The core intelligence of the tool. Automatically scans every log entry against **13 known issue patterns** and surfaces what matters.

Each detected issue shows:
- 🏷️ **Severity badge**: `Critical` | `High` | `Medium` | `Low`
- 📊 **Occurrence count**
- 🔍 **Root cause explanation** — written in plain English, not SDK jargon
- 🛠️ **Recommended troubleshooting steps** — specific, actionable, ordered by likelihood
- 🕐 **Affected timestamps** — when the issue occurred (up to 20 shown)

Issues are **sorted by severity** — critical items always appear first.

### 3. 📋 Log Explorer

Full log table for manual deep-dive when you need to see the raw data.

- **Color-coded rows** by log level:
  - 🔴 Error — red background highlight, bold level text
  - 🟠 Warning — orange level text
  - 🔵 Info — blue level text
  - ⚪ Trace — gray level text
- **Search** — free-text search across message, function name, source file, and timestamp
- **Filter by log level** — checkboxes to toggle Error, Warning, Info, Trace visibility
- **Filter by Thread ID** — dropdown to isolate a specific thread's activity
- **Click to expand** — click any message cell to expand the full message text (messages are truncated by default for readability)
- **Performance** — handles up to 2,000 rows in the display with a count of total filtered results

### 4. 📝 Analysis Report

Generates a structured, copy-paste ready case note in bullet-point format.

- **Log summary**: Total entries, error/warning/info/trace counts, time range, unique threads
- **Detected issues**: Each issue listed with severity, name, occurrence count, root cause, recommended steps, and first-seen timestamp
- **One-click copy** to clipboard
- **Export as `.txt`** file for attachment to tickets

---

## 🔎 Supported Issue Detection

| # | Pattern | Severity | What It Detects |
|---|---|---|---|
| 1 | **NoPermissionsError / Access Denied** | 🔴 Critical | User does not have rights to consume MIP-protected content — `AccessDenied`, `NoPermissionsError` |
| 2 | **Protection Handler Creation Failed** | 🔴 Critical | MIP SDK failed to create a protection handler for content consumption — usually a symptom of upstream failures |
| 3 | **Authentication / Token Failure** | 🔴 Critical | Auth token expired, OAuth errors, HTTP 401 Unauthorized from identity endpoints |
| 4 | **Profile Sync Failure** | 🔴 Critical | Edge profile sync errors — favorites, passwords, settings sync failures |
| 5 | **License Validation Failure** | 🟠 High | Local license store is empty or has no matching license for the content — `LicenseStore` lookup returned no rows |
| 6 | **User Certificate Store Issue** | 🟠 High | No user certificate found in the local cert store — needed for offline decryption or RMS client auth |
| 7 | **Service Discovery / DNS Failure** | 🟠 High | Cannot resolve RMS service endpoints — DNS lookup failures, missing redirect URLs |
| 8 | **Network Connectivity Issue** | 🟠 High | Timeouts, connection refused, connection reset — network path to cloud services is broken |
| 9 | **Generic API Call Failure** | 🟠 High | Any MIP SDK API call that failed (excluding protection handler, which has its own pattern) |
| 10 | **Offline Access Issue** | 🟡 Medium | Content's offline access policy is blocking access while the device is disconnected |
| 11 | **Template / Label Resolution Failure** | 🟡 Medium | Sensitivity label or RMS template could not be resolved — misconfiguration or ad-hoc protection |
| 12 | **Content ID / Encryption Issue** | 🟡 Medium | Errors with content encryption metadata, cipher mode, or the decryption process |
| 13 | **SQLite Storage Error** | 🟡 Medium | Local MIP database errors — locked, corrupt, or inaccessible SQLite storage |

---

## 📦 How to Use

### Step 1: Collect the logs
On the customer's machine, navigate to the MIP log directory:
```
%localappdata%\Microsoft\Edge\User Data\Default\MIP\Logs\
```
Collect the `.miplog` files from this folder.

### Step 2: Open the tool
Open `MIPLog_Analyzer.html` in any modern browser (Edge, Chrome, Firefox, etc.).

> **No installation, no server, no dependencies.**

### Step 3: Load the log
Drag and drop the `.miplog` file onto the upload area — or click **Browse File** to select it.

### Step 4: Review the dashboard
- Check the **summary cards** for error/warning counts
- Look at the **timeline** to see when issues occurred
- Switch to the **Detected Issues** tab for the auto-analysis

### Step 5: Investigate
- Review each detected pattern's **root cause** and **recommended steps**
- Use the **Log Explorer** to drill into specific entries or threads
- Search for specific keywords, function names, or error strings

### Step 6: Report
- Switch to the **Analysis Report** tab
- Click **📋 Copy Report** to copy the structured case note to your clipboard
- Or click **💾 Export as .txt** to download the report as a text file
- Paste directly into your CRM, ICM, or case notes

---

## 📂 MIPLog File Format

A `.miplog` file is a **tab-separated text file** with the following columns:

| Column | Description | Example |
|---|---|---|
| LogLevel | Severity level of the entry | `Error`, `Warning`, `Info`, `Trace` |
| Timestamp | Date and time of the event | `2026-01-22 11:11:41.057` |
| SourceFile | C++ source file and line number | `protection_engine_impl.cpp:1407` |
| ProcessInfo | Process name and PID | `msedge (20240)` |
| Message | Quoted log message content | `"Failed API call: ..."` |
| FunctionName | Fully qualified function name | `mipns::ProtectionEngineImpl::CreateProtectionHandlerForConsumption` |
| ThreadID | Thread identifier | `27136` |

---

## 🔬 Troubleshooting Scenarios

The MIPLog Analyzer is designed for scenarios like:

| Scenario | What You'll See |
|---|---|
| **"Access Denied" when opening a protected file** | `NoPermissionsError` detected — shows the content owner, recommends verifying permissions with the owner |
| **Protected content won't open offline** | `Offline Access Issue` and `License Validation Failure` — recommends clearing MIP cache and re-authenticating while online |
| **Edge profile sync not working** | `Profile Sync Failure` — recommends checking `edge://sync-internals`, toggling sync, and checking policies |
| **RMS-protected emails won't render** | `Protection Handler Creation Failed` with upstream `NoPermissionsError` or `UserCertStore` issue — shows the chain of failures |
| **Intermittent MIP failures** | Timeline shows error clusters — correlate with network events, VPN reconnects, or token expiry |
| **New device setup issues** | `UserCertStore` and `LicenseStore` empty — expected on first use, needs online authentication |

---

## 🔧 Technical Details

### Architecture
- **100% client-side** — runs entirely in the browser
- **No server required** — just open the HTML file
- **No network calls** — your data never leaves your machine
- **No external dependencies** — pure HTML, CSS, and JavaScript in a single file
- **Dark theme** — VS Code-inspired design with Microsoft Edge blue accent (`#0078D4`)

### How the Pattern Engine Works
1. Each log entry is tested against **13 regex-based pattern definitions**
2. Patterns are ordered by severity — `Critical` → `High` → `Medium` → `Low`
3. A single log entry can match **multiple patterns** (e.g., an API failure that is also a permissions error)
4. For each match, the tool aggregates:
   - Total occurrence count
   - All affected timestamps (capped at 20 for display)
   - Pre-written root cause explanation
   - Pre-written troubleshooting steps specific to Edge browser support

### Performance
- Parses logs with **thousands of entries** in under a second
- Log Explorer displays up to **2,000 rows** with filter counts for the full dataset
- Timeline chart auto-samples to **200 bars** for very large logs to maintain responsiveness

---

## 🌐 Browser Compatibility

| Browser | Supported |
|---|---|
| Microsoft Edge | ✅ |
| Google Chrome | ✅ |
| Mozilla Firefox | ✅ |
| Safari | ✅ |
| Opera | ✅ |

> Requires a modern browser with ES6+ support.

---

## 🔒 Privacy & Security

- **No data is uploaded** — all processing happens locally in your browser
- **No telemetry** — no analytics, no tracking, no cookies
- **No server** — no backend, no API calls, no external services
- **No network access needed** — works completely offline
- **Your logs stay on your machine** — safe for handling sensitive customer data and internal traces

---

## 💡 Use Cases

### For Support Engineers
- **First response triage**: Load the miplog, check Detected Issues, and immediately identify the root cause
- **Permission issues**: `NoPermissionsError` detected with the content owner's email — escalate to the right person instantly
- **Network troubleshooting**: DNS and connectivity patterns flagged — share with the customer's network team
- **Case documentation**: Export the analysis report and paste directly into your case notes

### For Technical Advisors
- **Quick review**: Review an engineer's miplog findings without opening the raw file
- **Pattern recognition**: The tool surfaces recurring issues that manual review might miss
- **Guidance validation**: Verify that the recommended steps align with the specific failure chain

### For Escalation Engineers
- **Root cause chain analysis**: Protection Handler failures trace back to upstream cert, license, or permission issues
- **Thread isolation**: Filter by Thread ID to follow a single operation's execution path
- **Timestamp correlation**: Cross-reference miplog timestamps with NetLog or Fiddler traces from the same session

---

## 🗺️ Roadmap

| Feature | Description | Priority |
|---|---|---|
| 📊 Thread Flow Visualizer | Visual diagram showing the sequence of operations per thread | High |
| 🔗 Cross-log Correlation | Load miplog + netlog together and correlate by timestamp | High |
| 🔐 Content ID Lookup | Extract ContentId values for backend RMS audit log queries | Medium |
| 📋 PDF Report Export | Generate a formatted PDF report for case attachments | Medium |
| 🏷️ Custom Pattern Rules | Let engineers define custom regex patterns for detection | Medium |
| 📦 Multi-file Support | Load and analyze multiple `.miplog` files from different dates | Low |
| 🌙 Light Theme Toggle | Switch between dark and light themes | Low |

---

## 📄 License

This project is licensed under the **MIT License**.

```
MIT License

Copyright (c) 2025

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## 🙏 Acknowledgments

- [JSZip](https://stuk.github.io/jszip/) — for client-side ZIP parsing (used in SAZ Analyzer Pro)
- [Fiddler](https://www.telerik.com/fiddler) by Telerik — for the SAZ file format
- [Microsoft MIP SDK](https://learn.microsoft.com/en-us/information-protection/develop/) — for the log format and error patterns
- Built with ❤️ for support engineers who spend too much time reading raw logs

---

<p align="center">
  <strong>browser-tools</strong> — Because your time is better spent solving problems, not searching for them.
</p>
