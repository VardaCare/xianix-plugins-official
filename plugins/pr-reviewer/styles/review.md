# PR Review Output Style Guide

This file defines the formatting and tone conventions for all output produced by the `pr-review` plugin agents.

---

## General Principles

- Be **direct and specific** — every finding must reference a file path and line number
- Be **actionable** — every issue must include a concrete fix or suggestion
- Be **proportionate** — severity labels must match actual impact, not be inflated
- Be **balanced** — always acknowledge what was done well, not only what is wrong
- Avoid filler phrases: "Great job!", "This is interesting", "As an AI..."

---

## Severity Levels

Use these labels consistently across all agents:

| Label | When to use |
|---|---|
| `CRITICAL` | Security vulnerabilities, data loss risk, broken functionality, blocks merge |
| `WARNING` | Non-blocking but should be fixed before merge — correctness, reliability concerns |
| `SUGGESTION` | Nice-to-have improvements — style, readability, minor optimisation |
| `POSITIVE` | Specific call-outs of good practices worth noting |

### Severity emojis (inline comments)

When posting inline PR comments (GitHub / Azure DevOps), prefix the body with the severity tag and emoji:

| Label | Emoji | Inline tag format |
|---|---|---|
| `CRITICAL` | 🚨 | `**🚨 [CRITICAL]**` |
| `WARNING` | ⚠️ | `**⚠️ [WARNING]**` |
| `SUGGESTION` | 💡 | `**💡 [SUGGESTION]**` |

---

## Finding Format

Every finding must follow this structure:

```
- `path/to/file.ext:LINE` — Short title of the issue

  **Why:** One sentence explaining the problem or risk.

  **Fix:**
  ```language
  // concrete corrected code
  ```
```

- File path is always relative to the repo root
- Line number is always included (`:LINE`)
- The fix block always uses a fenced code block with the correct language tag
- If a fix is not a code change (e.g. a config or process issue), use plain text after **Fix:**

---

## Verdict Labels

The final PR verdict must be one of exactly three values, rendered as inline code:

| Verdict | Meaning |
|---|---|
| `APPROVE` | No critical issues; warnings and suggestions are minor |
| `REQUEST CHANGES` | One or more critical issues must be resolved before merge |
| `NEEDS DISCUSSION` | Architectural or design concerns that require team input |

---

## Section Order

The compiled PR review report must follow this section order. Use the emoji prefixes from the table below on every section heading (see `styles/report-template.md`).

| # | Section | Heading |
|---|---|---|
| 1 | Header | `## 🔍 PR Review Report` (PR title, author, file counts, verdict) |
| 2 | Summary | `### 📋 Summary` |
| 3 | Critical Issues | `### 🚨 Critical Issues (Must Fix)` |
| 4 | Warnings | `### ⚠️ Warnings (Should Fix)` |
| 5 | Suggestions | `### 💡 Suggestions (Consider Improving)` |
| 6 | Review Details | `### 📊 Review Details` with `#### 🧹 Code Quality`, `#### 🔒 Security`, `#### 🧪 Test Coverage`, `#### ⚡ Performance` |
| 7 | Files Reviewed | `### 📁 Files Reviewed` (table) |

Do not reorder or omit sections. The **verdict** field stays plain uppercase text with no emoji — provider vote mapping depends on exact strings. If a section has no findings, write:
> *No [critical issues / warnings / suggestions] found.*

---

## Code Snippets

- Always use fenced code blocks with the language tag matching the file being reviewed (e.g. ` ```ts `, ` ```cs `, ` ```py `, ` ```go `, ` ```java `)
- Do not default to TypeScript — use the language of the actual file in the PR
- Show the **before** (problematic) and **after** (fixed) when the fix is non-obvious
- Keep snippets focused — show only the relevant lines, not entire functions
- Use `// ...` (or the appropriate comment syntax for the language) to indicate omitted lines

Example (language will vary per PR):

```
// Before
[problematic code in the detected language]

// After
[corrected code in the detected language]
```

---

## Risk Rating (Files Reviewed Table)

Use these emoji indicators in the Files Reviewed table:

| Emoji | Risk level | When to use |
|---|---|---|
| 🔴 | High | Auth, payments, DB migrations, public API surface |
| 🟡 | Medium | Business logic, data transformations, external integrations |
| 🟢 | Low | Utilities, config, docs, tests, formatting |

---

## Tone

- Use **second person** when addressing the author: "Consider extracting…", "This could be simplified…"
- Avoid passive voice: say "this will cause a SQL injection" not "a SQL injection may occur"
- Be concise — a finding should rarely exceed 5 lines of prose
- Positive observations should be specific: "Clean separation of concerns in `AuthService`" not "Nice code"
