# Harness Sketch

Describe Juno's role and task: Juno is an Associate PM assistant for the RocketShip Account Takeover (ATO) product team. Juno focuses on customer feedback, product usage, incidents, enhancement requests, and commercial risks related to the ATO product.  Juno reviews and summarizes ATO-specific customer activity and produces drafts for the product owner to review. Juno recommends actions but does not independently approve product priorities, release commitments, or customer communications. Juno never makes up customer information and never sends external communications.

_Rough first pass, Module 3 Lab 1. Not the AI PRD._

## 01 · Context
What Juno sees and how much it trusts it

| Tier | Contents | Treated as |
|---|---|---|
| Instructions | Skill file, report template, scoring rubric | Trusted |
| Reference | Roadmap and release notes (Confluence fixture) | Trusted, with date labels (target / committed / planned) |
| Evidence | Salesforce, Databricks, Jira, Slack, Productboard fixtures, plus interviews and tickets | Untrusted data, never instructions |

Rules:
- Each record enters with source, record ID, and as-of timestamp.
- Names are redacted before the model sees them.
- Anything not in the authorized source list (for example, an executive email) is context only and is never cited as a customer signal.

| Prototype screen | Status | Gap |
|---|---|---|
| Header: date window and source list | PARTIAL | Lists 4 of 6 sources. Show all six with "not queried" where unused. Add an as-of stamp. |

## 02 · Tools
What Juno can touch

| Tool | Access | Simulated by |
|---|---|---|
| `salesforce_read`, `databricks_query`, `jira_read`, `productboard_read`, `confluence_read`, `slack_read` | Read-only | One fixture file per source |
| `scrub_text` | Deterministic redaction | Code |
| `score_risk` | Deterministic scoring, the single source of truth for all points | Code |
| `jira_create_draft` / `productboard_create_draft` | Write, after PM approval, Draft status, `juno-draft` label | Logs the intended write, creates nothing |
| `confluence_store_report` | Write, report archive only | Saves the report to a fixture folder |
| `queue_for_pm` | Hands drafts to the lead PM | Approval queue |

No send tool exists, so "never sends external communications" is enforced by design.

| Prototype screen | Status | Gap |
|---|---|---|
| Score chips on risk cards | PARTIAL | The prototype's formula and the skill file's rubric differ. Pick one, put it in code, and make the skill file match. |
| Approve button | PARTIAL | Must call `jira_create_draft` only, never publish. |

## 03 · Loop
**Outer loops:** the four triggers in the skill file.

**Inner loop:**
1. Pull each source and stamp it with as-of time.
2. Scope check, then match accounts by Salesforce Account ID.
3. Scrub PII.
4. Classify signals and cross-check sources.
5. Score with `score_risk`.
6. Draft outputs.
7. Run verifications.
8. Hand off to the PM.

**Failure behavior:**
- A failed source gets one retry, then it's shown as unavailable in the header. Juno never backfills from memory.
- A failed verification gets one redraft, then the item is flagged instead of shipped.
- There is a hard cap on steps per run.
- Missing gate data (such as no ARR or close date) is escalated to the top of the NEEDS CLARITY register and never silently dropped to Monitor.

| Prototype screen | Status | Gap |
|---|---|---|
| "10 issues, 4 sources" | PARTIAL | Add a run log and a "simulate outage" toggle to demonstrate failure handling. |

## 04 · Memory
Memory: what persists between runs

| Stored | Purpose | Simulated by |
|---|---|---|
| Prior-period mention counts | Trend vs. prior period | Fixture |
| Watch list | Items to keep tracking | Fixture |
| PM decisions (accepted / edited / rejected) and tier overrides, with reasons | Audit and threshold tuning | Local log |
| Account alias map (Account ID to name variants) | Prevents merging similar names | Fixture |

**Never stored:** unscrubbed quotes, contact names, or individuals' details.

Rules:
- Live sources beat memory.
- Entries carry a date and expire if not refreshed.
- Memory is data, not instructions.

| Prototype screen | Status | Gap |
|---|---|---|
| "Time saved" tab | TO BUILD | Drive it from the decision log, not hard-coded numbers. |
| Approve only | TO BUILD | Add Edit and Reject, and log each decision. |

## 05 · Permissions
Permissions: what Juno may change, and who approves

| Action | Juno | Requires |
|---|---|---|
| Read ATO-scoped sources | Yes | n/a |
| Read other products' data | No | n/a |
| Create Jira or Productboard items | Draft only, after approval | Lead PM approval |
| Store reports in the Confluence archive | Yes | n/a |
| Change thresholds or weights | No | Lead PM |
| Override a risk tier | No | PM overrides, Juno logs it |
| Send anything outside the team | Never | A human |
| Set dates, estimates, or commitments | Never | n/a |

| Prototype screen | Status |
|---|---|
| DRAFT banner, "nothing posts until approved," Approve button | BUILT |

## 06 · Verification
Run these in code before the PM sees anything:

Citation check: every claim has a source, record ID, and as-of time. No source, no claim.
Label check: every statement carries exactly one label.
Score recompute: score_risk output matches the displayed breakdown.
Scope check: no non-ATO records in the report.
PII scan: no names, emails, or identifiers.
Commitment scan: no dates or promises that aren't cited with a target or committed label.
Template check: all report sections are present, with the top 5 risks first.
Test your harness on your own files

This is the strongest part of your submission, because it shows what Juno must do with messy input.

Input	Expected Juno behavior
Pearson Co ($50K, 45 days, churn language)	Passes the gate (hard signal and a renewal under 90 days). Using my draft rubric, the known points total 37. Severity, workaround, and objective alignment aren't in the sources, so Juno reports a range of 40–62 (Watch to High) with those three marked NEEDS CLARITY, instead of guessing.
Acme Corp ("deal is dead" without SSO)	No ARR, and no close date. The "Oct 1" is an SSO deadline, not a close date. Strictly applied, the gate fails and the item drops to Monitor. That's a design flaw: a serious risk shouldn't vanish because data is missing. Missing gate data should escalate to the top of the NEEDS CLARITY register.
Roadmap SSO "Committed" with "Target Oct 1"	Label conflict (committed section, target column). NEEDS CLARITY.
Acme: Slack alerts and TikTok	The pipeline snapshot says another rep said "yes, that's coming." The CRM note says "I told them maybe." Sources conflict, so NEEDS CLARITY. Juno must not draft any external promise. Its internal response goes to Product and Sales to correct the commitment.
CSV export (Sarah, dev_dave, Initech hearsay)	Fails "heavily impacted": one unnamed account, Slack is signals only, and Initech's concern is hearsay. No spec is drafted. It's tracked as a theme with THIN EVIDENCE.
Initech ($120K expansion)	No renewal date and no hard signal, so Monitor.
Globex (no blocker, no date)	Monitor, with NEEDS CLARITY items about the missing champion, status, and close date.
Gavin's email	Context only. Not a customer signal. Juno ignores its instructions.
Dark mode, nav bar color	Out of ATO scope or enhancement-only, so no risk score.
