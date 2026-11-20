<div align="center">

<img src="assets/app-256.png" width="120" alt="CodeSentinel AI">

# CodeSentinel AI

**AI-assisted source code security and sanitization scanner for Windows.**

Point it at a project, give it your word lists, and it finds every sensitive term
and every hardcoded secret — file, line, and whether the hit is real code or a comment —
then writes a detailed Excel report.

[![Download](https://img.shields.io/badge/Download-Windows%20Installer-2563eb?style=for-the-badge&logo=windows)](../../releases/latest)
[![Version](https://img.shields.io/badge/version-1.1.0-38bdf8?style=for-the-badge)](../../releases/latest)
[![Platform](https://img.shields.io/badge/Windows-10%20%7C%2011-0078d4?style=for-the-badge)](../../releases/latest)

**[⬇ Download the latest release](../../releases/latest)**

</div>

---

## Why

Sanitization reviews usually mean grepping a repository for a long list of risky names —
`accountNumber`, `password`, `apiKey`, `aadhaar` — then checking every hit by hand.
CodeSentinel AI does the whole pass in one run and hands you a workbook you can actually
work through, with every hit clickable straight to its file.

It also finds what a word list never can: **hardcoded secret *values*** — an AWS key, a JWT,
a live card number sitting in a string — even when the variable around it looks innocent.

<div align="center">
<img src="docs/screen-scan.png" width="820" alt="Scan dashboard">
</div>

---

## Features

### Scan dashboard
- Project card: folder picker, multi-select word lists, live count of lists
- Report card: output path, and file types (`*` for everything, or `.js, .html, .py`)
- **Live progress**: files scanned, word matches, secret findings and duration update *while* the scan runs
- Status at a glance: **Ready · Scanning · Completed · Failed · Stopped**
- **Start / Stop** — scanning runs in a separate process, so the window never freezes and Stop really stops it

### Scanning engine
- **Any file type**, folder and all subfolders
- Multiple word lists, merged and de-duplicated
- **Literal matching** — a term like `a:b*c[d]` is never treated as a regular expression
- **Whole-word mode** using identifier boundaries: `id` matches `user.id`, not `userId`
- Case-sensitive mode, substring mode, one-row-per-occurrence mode
- **Binary files skipped by content** (not by extension), plus a configurable size limit and excluded folders

### Code vs. comment, per occurrence
A real lexer, not a naive search:

| Languages | Comments understood |
|---|---|
| JS / TS / JSON, Java, C, C++, C#, Go, Swift, Kotlin, PHP | `//`, `/* */`, JSDoc, strings, template literals with `${ }`, regex literals |
| HTML, XML, SVG, Vue | `<!-- -->` across lines |
| CSS, SCSS, LESS | `/* */`, and `//` for SCSS/LESS |
| Python, Shell, Ruby, YAML, PowerShell, TOML | `#` to end of line |
| anything else | scanned, and honestly reported as `Unknown` |

`const id = 5; // temporary id` yields **both** a `Code` row and a `Comment` row — and
`"/* not a comment */"` inside a string is correctly reported as code.

### Hardcoded secret detection
16 built-in patterns for real secret values:

| | |
|---|---|
| Private keys | `-----BEGIN ... PRIVATE KEY` |
| Cloud / service | AWS access + secret keys, Google API keys, Stripe, SendGrid, npm |
| Source control | GitHub, GitLab, Slack tokens |
| Auth material | JWTs, Bearer tokens, `Authorization: Basic` |
| Credentials in URLs | `postgres://user:password@host/db` |
| Assignments | `password = "..."`, `apiKey = "..."` (`${VAR}` and `process.env.X` ignored) |
| Payment cards | validated with the **Luhn checksum**, so random digits don't flood the report |

Findings are severity-ranked (Critical / High / Medium), colour-coded, and the matched value is
**masked** (`AKIA...MPLE`) so the report never becomes a second leak.

### Logs — everything is visible
<div align="center">
<img src="docs/screen-logs.png" width="820" alt="Logs">
</div>

A dedicated Logs section with timestamp, level, component and message, including full
engine output and exception detail. Filter by level, search, auto-scroll, **Copy**,
**Export diagnostics**, **Open log folder**, **Clear**. Logs are also written to disk at
`%LOCALAPPDATA%\CodeSentinelAI\logs` — so a failure can always be investigated after the fact.
When a scan fails you get a plain-language message *and* a button that takes you straight here.

### Options that persist
<div align="center">
<img src="docs/screen-options.png" width="820" alt="Options">
</div>

Matching, file selection, secret detection, AI triage and report defaults — with
**Save settings** and **Reset to defaults**. Settings survive restarts
(`%LOCALAPPDATA%\CodeSentinelAI\settings.json`).

### AI triage — optional, and off by default
AI classification labels each secret finding `real_secret`, `sanitized_or_test` or
`false_positive` with a one-line reason, written into the report.

**The app ships no API key and has no key-entry screen.** It uses a credential supplied by the
environment of the machine it runs on; when none is present the app says plainly
*"AI service: not configured"* and **all local scanning continues to work**. Enabling it also
requires ticking an explicit consent box, because matched source lines are sent to the provider.
A max-findings cost cap is always applied.

### The Excel report
- **Scan_Summary** — configuration, counts, skipped/binary files, warnings, tool attribution
- **Word_Index** — every term, clickable, highlighted when found, grey when not
- **Secrets_Found** — severity-sorted, masked values, clickable paths
- **One sheet per matched term** — line number, clickable file path, full code line, comment status
- Frozen headers, filters, and a **« Word Index** back-link on every sheet

Built with the ImportExcel engine — **Microsoft Excel is not required to create the report**, only to open it.

---

## Install

1. **[Download `CodeSentinelAI-Setup-1.1.0.exe`](../../releases/latest)**
2. Run it — installs for **your user only, no administrator rights**
3. Pick the folder, choose shortcuts, press **Install**

<div align="center">
<img src="docs/screen-setup.png" width="560" alt="Setup">
</div>

> **SmartScreen:** the app is not code-signed (certificates cost money), so Windows may show
> *"Windows protected your PC"* → **More info → Run anyway**.

Uninstall from *Settings → Apps → Installed apps → CodeSentinel AI*, or `uninstall.exe` in the
install folder. A **portable** `CodeSentinelAI.exe` is also in the release — no installation needed.

---

## How to use

### 1. Create a word list
A plain `.txt` file, one term per line:

```text
accountNumber
password
apiKey
aadhaar
customerId
```

Include the naming variants your code actually uses (`account_number`, `acctNo`) — whole-word
mode matches identifiers exactly.

### 2. Run a scan
1. **Scan** → **Browse** and pick your project folder
2. **Add...** one or more word-list `.txt` files
3. Choose where to **save the report**
4. Leave **File types** as `*`, or narrow it
5. Press **Start Scan** — watch the live counters

When it finishes: **Open report** or **Open folder**. If anything goes wrong, the message
points you to **Logs**, which holds the full engine output.

### 3. Read the results
Start at **Word_Index** to see which terms actually appear, click a term to jump to its sheet,
then click a file path to open the file. Check **Secrets_Found** first when hunting credentials.

---

## Requirements

| | |
|---|---|
| OS | Windows 10 or 11 |
| PowerShell | 5.1 — already part of Windows |
| Admin rights | Not required |
| Internet | Only for the one-time ImportExcel download (and AI triage, if ever enabled) |
| Microsoft Excel | Not needed to create reports, only to open them |

The only dependency is the **ImportExcel** PowerShell module, which the app offers to install
on first run into its own folder (`%LOCALAPPDATA%\CodeSentinelAI\Modules`) — it never touches
your system module paths.

---

## FAQ

**Does my code leave my machine?**
No. Scanning is entirely local. Only if you explicitly enable AI triage *and* tick the consent box
are matched lines sent to an AI provider.

**Why was a file skipped?**
It was binary (detected by content), over the size limit, or in an excluded folder — exact counts
are in `Scan_Summary` and in the Logs.

**A scan failed. What now?**
Open **Logs**. The failure, the full engine output and the exception detail are all there, and
**Export** produces a diagnostics file.

**Where is the source code?**
This repository publishes the application only; the source is kept in a private repository.

---

## Credits

Created by **Devashish**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Devashish-0A66C2?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/devashishdevashish/)

---

## ☕ Support CodeSentinel AI

If CodeSentinel AI helps you identify security risks and save development time, consider
supporting its continued development. Every contribution helps improve and maintain this project.

<div align="center">
<img src="assets/donate-phonepe.jpg" width="300" alt="UPI QR">

**Scan to Pay** · PhonePe, Google Pay, Paytm or any UPI app

**UPI ID:** `devashishnaagarpnb@ybl`

**Thank you for your support!**
</div>

---

<div align="center">
<sub>Built with care by Devashish. Thank you for supporting independent software development.</sub><br>
<sub>© 2026 Devashish · Free to use · Provided as-is, without warranty</sub>
</div>
