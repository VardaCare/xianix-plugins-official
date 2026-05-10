---
name: post-test-report
description: Phase 3 of web-app-tester. Computes the overall verdict (PASSED / FAILED / BLOCKED) from the inline step results, composes a single GitHub comment that conforms exactly to the report template, and posts it via the GitHub provider. The report is strictly bounded — no recommendations, no root-cause analysis, no commentary outside the defined sections.
disable-model-invocation: true
---

# Phase 3 — Post Test Execution Report

This skill is invoked by the **orchestrator** agent. It is not a standalone slash command.

## Inputs

| Variable | Source | Description |
|---|---|---|
| Inline result list | run-playwright-session | One entry per step: `{ n, desc, status, reason, screenshot }` |
| `TEST_URL` | gather-test-context | URL that was tested |
| `PRODUCTION_WARNING` | gather-test-context | Whether read-only mode was applied |
| `ENTRY_TYPE`, `ENTRY_ID` | orchestrator | PR or Issue number |

## Outputs

A single GitHub comment posted on the PR or issue, plus a one-line confirmation written to stdout.

---

## Step 1: Compute Overall Verdict

Determine the overall result from the per-step statuses:

| Condition | Overall Result |
|---|---|
| All steps passed | **PASSED** |
| One or more steps failed (all steps were attempted) | **FAILED** |
| One or more steps could not execute (element not found, page error, timeout, auth gate, production-mode skip) | **BLOCKED** |

A run with both FAILED and BLOCKED steps uses **BLOCKED** as the overall result.

Store as `OVERALL_RESULT`.

---

## Step 2: Compose the Report Body

Build the comment body using the **exact** structure defined in `styles/report-template.md`. The report comment must contain **only** the sections defined in that template. Do not add suggested fixes, recommendations, next steps, root cause analysis, explanations, or any content not defined in the template. If you have observations beyond the test results, discard them — they do not belong in this comment.

The skeleton is:

```
🤖 web-app-tester — Test Execution Report
URL tested: {TEST_URL}
{PRODUCTION_WARNING ? "⚠️ URL appears to be production. Executed read-only steps only." : ""}
Total: N | ✅ Passed: X | ❌ Failed: Y | 🔴 Blocked: Z
Overall: PASSED / FAILED / BLOCKED

| # | Step | Status |
|---|------|--------|
| 1 | {step description} | ✅ PASSED |
| 2 | {step description} | ❌ FAILED |

[For each FAILED or BLOCKED step:]
**Step N — {description}**
Reason: {what went wrong after 3 retries}
[Screenshot attached if available]
```

Step descriptions in the report must be in **business language** — describe the user action and observed outcome, not the Playwright command. See the `Step Description Format` section in `styles/report-template.md` for examples.

For FAILED and BLOCKED steps, screenshots are **described inline** as "captured at point of failure" — GitHub PR/issue comments do not support direct file attachments via `gh comment`, so the PNG files referenced in `screenshot` are not uploaded.

Store as `REPORT_BODY`.

---

## Step 3: Post the Report

Read and follow `providers/github.md` to post the report comment. The provider file owns the exact `gh pr comment` / `gh issue comment` invocation and the heredoc pattern needed to preserve formatting.

Post a **single comment**. Never split the report across multiple comments.

---

## Step 4: Final Output

After posting, write a single confirmation line to stdout:

```
web-app-tester complete for {ENTRY_TYPE} #{ENTRY_ID}: {OVERALL_RESULT} — {PASSED}/{TOTAL} steps passed
```

If posting fails, output a single error line describing what failed and stop.
