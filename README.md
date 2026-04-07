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
