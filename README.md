#  Commit Slip Agent

> An AI-powered RevOps agent that automatically detects slipped forecast deals, generates structured post-mortems, and delivers leadership-ready reports via Slack — before the first forecast meeting of each month.

---

##  The Problem

Every month, Sales Leaders spend 2–3 hours manually:
- Identifying which "Commit" deals didn't close
- Pulling call recordings to understand why
- Cross-referencing SPICED notes in Salesforce
- Preparing a summary for the leadership forecast meeting

This agent does all of it automatically, overnight, on the 1st of each month.

---

##  Architecture

```
Salesforce (CData)          Attention (Call AI)       Google Calendar
       │                           │                        │
       ▼                           ▼                        ▼
  Slip Cohort              Call Transcripts          Forecast Meeting
  Detection                & Signal Analysis         Date Lookup
       │                           │                        │
       └───────────────────────────┘                        │
                       │                                    │
                       ▼                                    │
              AI Post-Mortem Engine  ◄──────────────────────┘
              (Claude Sonnet 4.6)
                       │
                       ▼
              Slack DM to Leadership
              (1st of every month, before 8am)
```

---

##  How It Works — Step by Step

### Step 1 — Capture the Slip Cohort
On the last business day of each month, the agent snapshots all Salesforce Opportunities where:
- `Type = 'New Business'`
- `ForecastCategoryName = 'Commit'`
- `CloseDate` falls within the current month
- `StageName NOT IN ('Closed Won', 'Closed Lost')`
- `IsDeleted = false`

Compares against the snapshot from 30 days prior to isolate net-new slips.

### Step 2 — Pull SPICED Notes from Salesforce
For each slipped deal, retrieves SPICED methodology fields and flags risk signals the rep may have missed:
- No named Economic Buyer or Champion
- Vague Critical Event (e.g. "end of quarter") with no external forcing function
- No mutual close plan documented
- Impact/Pain not tied to a quantified business outcome
- Decision criteria not captured

### Step 3 — Analyze Attention Call Data
Queries Attention for all calls linked to each slipped Opportunity and extracts:
- Call dates and participants
- Topics discussed (deal stage, objections, stakeholder dynamics)
- Rep commitments vs. follow-through
- Buyer hesitation signals, scope creep, executive disengagement
- **Flags deals with no call activity in the last 14 days of the month**

### Step 4 — Generate Per-Deal Post-Mortem (AI)
Claude synthesizes SPICED + call data into a structured diagnosis:

| Field | Content |
|-------|---------|
| Deal Summary | AE, Account, ARR, original vs. new close date |
| What Rep Believed | Based on last forecast notes / SPICED |
| What Signals Showed | Call themes, last activity date, engagement drop-off |
| Root Cause | Taxonomy: No Critical Event / Champion not mobilized / No Economic Buyer / Scope misalign / Internal blocker / Competition / Sandbagging |
| Risk Rep Missed | Specific SPICED gaps or call signals |
| Recommended Action | What leadership should do next month |

### Step 5 — Check Calendar for Forecast Meeting
Queries Google Calendar / Outlook for the first "Forecast Leadership" or "Pipeline Review" event in the new month — so the Slack message contextualizes urgency correctly.

### Step 6 — Send Slack DM
Sends Hassan a direct Slack message on the 1st of each month before 8am:

```
📋 Commit Slip Report — [Month]
[N] deals committed to close in [Month] did not close.
Here's the post-mortem before your [Date] forecast meeting.

Deal 1: Acme Corp — $85,000 | AE: Sarah M.
• Original Close Date: May 31 / New Projected: June 30
• Root Cause: No true Critical Event — "end of quarter" was rep-driven, not buyer-driven
• Risk Rep Missed: Champion (VP Ops) went dark after April 12 call. No exec sponsor confirmed.
• Recommended Action: Re-qualify with economic buyer before re-committing to June

[...repeat per deal...]

Team-Level Pattern: 4 of 6 slips had no documented external forcing function.
This is a forecasting discipline issue, not a market signal.

📅 Your next forecast leadership meeting is Tuesday, June 3 @ 9:00am.
This is ready as a standing agenda item.
```

---

## 🛠️ Tech Stack

| Component | Tool |
|-----------|------|
| Workflow Automation | n8n |
| CRM Data | Salesforce via CData connector |
| Call Intelligence | Attention |
| AI Analysis | Claude Sonnet 4.6 (Anthropic API) |
| Calendar | Google Calendar / Outlook |
| Messaging | Slack API |
| Scheduling | n8n Cron (runs last business day of month) |

---

## Repository Structure

```
commit-slip-agent/
├── README.md
├── .gitignore
├── config.example.env          # Environment variables template
├── mock_data/
│   ├── mock_opportunities.json # Sample Salesforce data (anonymized)
│   ├── mock_calls.json         # Sample Attention call data
│   └── mock_output.json        # Sample Slack message output
├── n8n/
│   └── workflow.json           # Importable n8n workflow
└── prompts/
    └── postmortem_prompt.txt   # AI prompt for post-mortem generation
```

---

##  Setup (with real data)

### Prerequisites
- n8n instance (cloud or self-hosted)
- Salesforce org with SPICED fields configured
- Attention account connected to Salesforce
- Slack bot with DM permissions
- Google Calendar or Outlook connected
- Anthropic API key

### Environment Variables
```bash
cp config.example.env .env
# Fill in your credentials — never commit .env to git
```

### Import n8n Workflow
1. Open your n8n instance
2. Go to **Workflows → Import**
3. Upload `n8n/workflow.json`
4. Configure credentials for each node
5. Activate the workflow

---

##  Key Metrics Tracked

- **Slip Rate** = # Commit deals that didn't close / total Commit deals at month-start *(target: <15%)*
- **Signal Miss Rate** = % of slipped deals with at least one identifiable SPICED gap
- **Attention Coverage** = % of slipped deals with at least 1 call in the last 14 days of the month

---

##  Important Notes

- **Point-in-time snapshot is critical** — Salesforce doesn't natively preserve forecast category history. This workflow logs the cohort to a separate table at month-end before reps can retroactively re-stage deals.
- SPICED field API names vary by org — confirm with RevOps before deploying.
- "Managed push" (customer-requested) is excluded from slip cohort to avoid false positives.

---

##  Business Impact

> *"This used to take me 2–3 hours every month. Now it's in my Slack DM before I wake up."*

Built to solve a real RevOps problem: giving Sales Leadership the signal, not just the data, before the forecast call.

---

## 📄 License

MIT — use freely, attribute appreciated.
