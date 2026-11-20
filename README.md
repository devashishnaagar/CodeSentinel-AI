<div align="center">

<img src="assets/app-256.png" width="120" alt="Code Sanitization Scanner">

# Code Sanitization Scanner

**Find sensitive words and hardcoded secrets anywhere in a codebase — and get a clean Excel report.**

A Windows desktop app for source-code review and sanitization.
Point it at a folder, give it your word lists, and it tells you exactly where every
term and every hardcoded key appears — file, line, and whether it is real code or a comment.

[![Download](https://img.shields.io/badge/Download-Windows%20Installer-2563eb?style=for-the-badge&logo=windows)](../../releases/latest)
[![Version](https://img.shields.io/badge/version-1.0.0-informational?style=for-the-badge)](../../releases/latest)
[![Platform](https://img.shields.io/badge/Windows-10%20%7C%2011-0078d4?style=for-the-badge)](../../releases/latest)

**[⬇ Download the latest release](../../releases/latest)**

</div>

---

## What it does

Security reviews and sanitization passes usually mean grepping a repo for a long list of
risky names — `accountNumber`, `password`, `apiKey`, `aadhaar` — and then checking every hit
by hand. This app does the whole pass in one run and hands you a workbook you can actually
work through: one sheet per term, every hit clickable straight to the file.

It also looks for the thing a word list can never find: **hardcoded secret *values***, such as
an AWS key or a live card number sitting in a string, even when the variable around it has a
perfectly innocent name.

<div align="center">
<img src="docs/screen-scan.png" width="760" alt="Scan tab">
</div>

---

## Features

### Scanning
- **Any file type.** Scans the folder and every subfolder. Default is *every* file type; restrict it to `.js, .html, .py` or anything else when you want a narrower pass.
- **Multiple word lists.** Load as many `.txt` lists as you like; duplicates across lists are merged automatically.
- **Literal, safe matching.** Terms are matched literally, so `a:b*c[d]` never behaves like a regular expression.
- **Whole-word mode.** Uses identifier boundaries, so `id` matches `user.id` and `(id)` but not `userId` or `identity`. Switch it off for substring matching.
- **Case-sensitive or not**, your choice.
- **Binary files are skipped automatically**, detected by content rather than by extension, so images, DLLs and archives never pollute the report.
- **Large-file guard** with a configurable size limit, and folders such as `node_modules`, `.git`, `dist`, `build` skipped by default.

### Code vs. comment — per occurrence
A real lexer, not a naive search. For every match it tells you whether that *specific*
occurrence sits in executable code or inside a comment:

- JavaScript / TypeScript / JSON and C-like languages (Java, C, C++, C#, Go, Swift, Kotlin, PHP): `//`, `/* */`, JSDoc, strings, template literals with `${ }`, and regex literals
- HTML / XML / SVG / Vue: `<!-- -->` across lines
- CSS, SCSS, LESS
- Python, Shell, Ruby, YAML, PowerShell, TOML: `#` comments
- Anything else is still scanned, and reported as `Unknown` rather than guessed

So `const id = 5; // temporary id` correctly produces **both** a `Code` row and a `Comment` row —
and `"/* not a comment */"` inside a string is correctly reported as code.

### Hardcoded secret detection
Independent of your word lists, 16 built-in patterns look for real secret *values*:

| | |
|---|---|
| Private key blocks | `-----BEGIN ... PRIVATE KEY` |
| Cloud / service keys | AWS access + secret keys, Google API keys, Stripe, SendGrid, npm |
| Source control | GitHub, GitLab, Slack tokens |
| Auth material | JWTs, Bearer tokens, `Authorization: Basic` headers |
| Credentials in URLs | `postgres://user:password@host/db` |
| Assignments | `password = "..."`, `apiKey = "..."` (placeholders like `${VAR}` and `process.env.X` are ignored) |
| Payment cards | card numbers validated with the **Luhn checksum**, so random digits don't flood the report |

Findings are severity-ranked (Critical / High / Medium), colour-coded, and the matched value is
**masked** in the report (`AKIA...MPLE`) so the workbook itself doesn't become a second leak.

### Optional AI triage
Turn it on and each secret finding — the matched line plus a little context — is sent to an AI
model, which labels it `real_secret`, `sanitized_or_test`, or `false_positive` with a one-line
reason, written into two extra columns.

Works with **Claude (Anthropic)**, **ChatGPT (OpenAI)** or **Gemini (Google)** — pick the provider
in Options. It is **off by default**, runs only when you supply your own API key, and has a
"max findings" cost cap.

> **Privacy:** AI triage sends the matched source lines to the provider you choose and bills your
> own API key. Leave it off if the code must not leave your machine — everything else runs fully offline.

### The Excel report
<div align="center">
<img src="docs/screen-options.png" width="760" alt="Options tab">
</div>

- **Scan_Summary** — what was scanned, counts, configuration, skipped/binary files, warnings
- **Word_Index** — every word from your lists, row by row: highlighted and **clickable** when found, grey when not
- **Secrets_Found** — hardcoded secrets by severity, with masked values and clickable paths
- **One sheet per term that has matches** — Line Number, clickable File Location, the full Code line, and Comment Status
- Frozen headers, filters, wrapped code column, sensible widths, and a **« Word Index** back-link on every sheet

Terms with no matches don't create a sheet (they stay listed in Word_Index), so the workbook
holds only what matters.

Built with the ImportExcel engine — **Microsoft Excel is not required to create the report**, only to open it.

---

## Install

1. **[Download `CodeScanner-Setup-1.0.0.exe` from the latest release](../../releases/latest)**
2. Run it. It installs for **your user only — no administrator rights needed.**
3. Choose the folder (default `%LOCALAPPDATA%\Programs\CodeScanner`), pick whether you want a desktop shortcut, and press **Install**.

<div align="center">
<img src="docs/screen-setup.png" width="560" alt="Setup window">
</div>

> **Windows SmartScreen:** the app isn't code-signed (a certificate costs money), so Windows may show
> *"Windows protected your PC"*. Click **More info → Run anyway**. If you prefer, scan the file with
> VirusTotal or your own antivirus first.

**Uninstall** any time from *Settings → Apps → Installed apps → Code Sanitization Scanner*, or run
`uninstall.exe` in the install folder.

There is also a **portable** `CodeScanner.exe` in the release — it needs no installation at all, just run it.

---

## How to use it

### 1. Make a word list
A plain `.txt` file, **one term per line**:

```text
accountNumber
password
apiKey
aadhaar
customerId
```

Tip: include the naming variants your codebase actually uses — `account_number`, `accountNo`,
`acctNum` — because whole-word mode matches identifiers exactly.

### 2. Run a scan
1. Open the app, **Scan** tab
2. **Folder to scan** → *Browse* and pick your project (subfolders are included)
3. **Word lists** → *Add...* and select one or more `.txt` files
4. **Save report to** → where the `.xlsx` should go
5. **File types** → leave `*` for everything, or type `.js, .html, .py`
6. Press **Start scan**

Progress streams into the log panel. When it finishes, **Open report** opens the workbook and
**Open folder** reveals it in Explorer.

> On the very first run the app offers to download the **ImportExcel** module (one time, needs
> internet). Say yes — reports can't be written without it.

### 3. Work through the results
Start at **Word_Index** to see which terms actually appear, click a word to jump to its sheet,
then click any file path to open the file. Check **Secrets_Found** first if you're hunting
leaked credentials — it's sorted by severity.

### 4. (Optional) Turn on AI triage
In **Options → AI triage**, tick the box and pick your provider. Set the matching API key once,
then restart the app:

```powershell
setx ANTHROPIC_API_KEY "your-key"   # Claude   - console.anthropic.com
setx OPENAI_API_KEY    "your-key"   # ChatGPT  - platform.openai.com
setx GEMINI_API_KEY    "your-key"   # Gemini   - aistudio.google.com
```

The key is read from your environment — it is never stored in the app or written into the report.

---

## Requirements

| | |
|---|---|
| OS | Windows 10 or 11 |
| PowerShell | 5.1 — already part of Windows |
| Admin rights | Not needed |
| Internet | Only for the one-time ImportExcel download, and for AI triage if you enable it |
| Microsoft Excel | Not needed to create reports, only to open them |

---

## FAQ

**Does my code leave my machine?**
No — unless you explicitly enable AI triage, which sends only the matched lines to the provider you pick.

**Why was a file skipped?**
It was binary (detected by content), larger than the size limit, or inside an excluded folder.
Exact counts are in `Scan_Summary`.

**How fast is it?**
A 120,000-line benchmark against 286 search terms completes in well under a minute; a real
13,700-file project with 312 terms and 205 secret findings runs end to end in about 13 minutes,
most of it spent writing the workbook.

**Where is the source code?**
This repository publishes the application only. The source is kept in a private repository.

---

## Credits

Created by **Devashish**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Devashish-0A66C2?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/devashishdevashish/)

---

## ☕ Buy me a coffee

If this tool saved you an afternoon of grepping, a small tip is very welcome.
Scan with **PhonePe, Google Pay, Paytm** or any UPI app:

<div align="center">
<img src="assets/donate-phonepe.jpg" width="300" alt="UPI donation QR for Devashish">

**Thank you for your support!**
</div>

---

<div align="center">
<sub>© 2026 Devashish · Free to use · Provided as-is, without warranty</sub>
</div>
