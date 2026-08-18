# Claude-First Automation System (n8n)

An enterprise-grade, human-in-the-loop automation layer engineered on **n8n Cloud** for a scientific-instrument company (Droplet Smart Tech Inc.). The system uses **Claude (Anthropic API)** as a centralized reasoning, drafting, and synthesis engine, orchestrating multi-channel workflows across CRM, analytics, marketing nurture, multi-platform social distribution, and email intelligence without requiring manual intervention for routine data aggregation.

---

## 1. Core Architecture & Operating Philosophy

The platform operates on a strict **Draft-and-Approve** model. Claude performs structured information extraction, decision-making, and contextual drafting, while deterministic n8n workflows execute data pulls, queue approvals, handle API handshakes, and synchronize business systems.

```text
[ Data & Trigger Sources ]
 (GA4, GSC, Gojiberry, Encharge, Gmail, Airwallex)
                       │
                       ▼
         [ n8n Cloud Orchestrator ]
                       │
                       ▼
      [ Claude API (Anthropic REST Engine) ]
    (Synthesize, Extract Intent, Structure Drafts)
                       │
                       ▼
        [ Centralized Review Queue ]
        (Human-in-the-Loop Review & Gate)
           │                       │
     [ Rejected / Edited ]    [ Approved ]
           │                       │
           ▼                       ▼
     (Drop / Re-queue)     [ Outbound Execution Layer ]
                           (HubSpot, Encharge, Outstand, Wave)

```

### Non-Negotiable System Principles

* **Mandatory Human Gate:** Zero autonomous sends or live posts. All client communications and social content land in a dedicated approval queue for founder review.


* **Real Footage Only:** No synthetic/AI-generated images or video. Social media workflows only schedule verified product assets or recommend specific real-world visual footage.


* **Resilient Infrastructure:** Every workflow features idempotency, retry mechanisms, and automated failure notifications to prevent double-posting or silent failures.


* **Enterprise Credential Isolation:** All authentication tokens, API keys, and OAuth2 credentials live inside the n8n Credential Store.



---

## 2. Integrated Tech Stack

| Operational Function | Tool / Platform | Integration Type | Description & Role |
| --- | --- | --- | --- |
| **Reasoning & Drafting** | Claude API (Anthropic)

 | REST API

 | Context extraction, copy drafting, and data synthesis.

 |
| **Orchestration** | n8n Cloud

 | Cloud Hosted

 | Workflow automation, error alerting, and data transformation.

 |
| **CRM (System of Record)** | HubSpot (Starter)

 | REST API

 | Bi-directional contact management and activity logging.

 |
| **Social Distribution** | Outstand

 | REST + Webhooks

 | Unified social API scheduling for approved posts.

 |
| **Email Nurture** | Encharge

 | Ingest API / Webhooks

 | Behavior-triggered, segmented marketing sequences.

 |
| **Email Intelligence** | Gmail (Google Workspace)

 | Gmail API (Read)

 | Historical thread mining for client re-engagement.

 |
| **Cold Outreach** | Gojiberry

 | Existing MCP Connection

 | LinkedIn outreach pipeline tracking and metrics.

 |
| **Analytics Sources** | GA4, Search Console, Encharge, Gojiberry

 | REST APIs

 | Raw performance inputs for weekly strategic briefs.

 |
| **Content Ingestion** | Castmagic

 | Feed Export

 | Optional transcript/footage source for social repurposing.

 |
| **Accounting / Finance** | Airwallex & Wave

 | CSV Pre-categorization

 | Automated statement structuring for ledger reconciliation.

 |

---

## 3. Workflow Implementation Breakdown

### Phase 1: Automated Weekly Executive Analytics Digest

* **Trigger:** Scheduled cron execution (weekly).


* **Pipeline:** Concurrently queries REST endpoints for GA4, Google Search Console, Encharge, and Gojiberry.


* **AI Synthesis:** Passes raw multidimensional data to Claude via structured prompts. Claude correlates metrics, isolates performance deltas, and delivers an executive brief outlining what changed and recommended operational actions.


* **Delivery:** Delivers a single synthesized report directly to the founder via Email/Slack.



### Phase 2: Gmail Mining, CRM Enrichment & Review Queue

* **Gmail Thread Extraction:** Reads historical client communication via the Gmail API for selected contacts.


* **Contextual Parsing:** Claude isolates critical discussion points, unresolved technical issues, change requests, and previous commercial objections. Source thread links are preserved for verification.


* **HubSpot Synchronization:** Updates contact properties with structured background notes while flagging low-confidence extractions.


* **Draft Generation:** Generates a personalized re-engagement draft referencing exact previous interactions.


* **Central Review Gate:** Dispatches the draft and extracted context to the central review queue where the founder approves, modifies, or rejects before manual sending.



### Phase 3: Multi-Mode Social Engine & Behavioral Nurture

* **Multi-Mode Social Pipeline:**
* Generates native educational text posts without visual requirements.


* Repurposes rich media transcripts into platform-native posts when Castmagic data is supplied.


* Generates high-value, additive engagement comments tailored to scientific precision standards.


* Analyzes copy and explicitly recommends the exact real product visual or footage needed.


* Routes all posts to the review queue before scheduling via the Outstand API.




* **Behavior-Triggered Encharge Nurture:**
* Ingests behavioral triggers (report download, trial request, demo attendance).


* Dynamically maps sequences segmented by HubSpot buyer personas (e.g., QC Manager vs. R&D Scientist).





### Phase 4: Financial Data Normalization & Cross-System Sync

* **Financial Ledger Assist:** Ingests monthly Airwallex transaction exports, uses Claude to parse descriptions and pre-categorize line items, and outputs clean CSV structures for Wave import and reconciliation.


* **HubSpot Unified Sync:** Reconciles contact stages, behavioral tags, and engagement statuses across Gojiberry, Encharge, and HubSpot.


* **Error Hardening & Alerting:** Global error handler catches HTTP timeouts, validation mismatches, and rate limits, instantly notifying the founder with workflow execution links.



---

## 4. End-to-End Execution Flow

```text
[ Contact Event: Re-Engagement Selected ]
                   │
                   ▼
       [ n8n: Query Gmail API ]
  (Fetch message thread history & metadata)[cite: 1]
                   │
                   ▼
  [ n8n Code Node: Format Email Body & Clean HTML ]
                   │
                   ▼
     [ Claude API: Information Extraction ]
  - Extract client issues, requests & sentiment[cite: 1]
  - Draft re-introduction email matching brand tone[cite: 1]
  - Return strict JSON schema with thread references[cite: 1]
                   │
                   ▼
       [ n8n: Update HubSpot Record ]
  (Write extracted facts + flag confidence levels)[cite: 1]
                   │
                   ▼
     [ n8n: Post to Review Queue UI ]
  (Display draft, original thread link, and audit data)[cite: 1]
                   │
         ┌─────────┴─────────┐
         ▼                   ▼
    [ Approved ]        [ Rejected ]
         │                   │
         ▼                   ▼
[ Ready for Manual Send ] [ Archive / Edit ][cite: 1]

```

---

## 5. Security, Guardrails & Governance

* **Zero Direct Publishing:** Outbound communication requires explicit approval; no autonomous sends exist in production.


* **Anti-Hallucination Email Verification:** Any extracted business requirement or client complaint surfaces the exact source thread link to ensure facts are validated before drafting.


* **Brand Protection:** Social commenting logic prohibits generic engagement bait, enforcing high-depth, additive technical responses.


* **Scoped Token Access:** Integrations use least-privilege OAuth2 scopes (e.g., Gmail read-only).



---

## 6. Repository & Workflow Structure

```text
droplet-automation-engine/
├── README.md
├── docs/
│   ├── runbook.md
│   ├── prompt-templates.md
│   ├── error-handling.md
│   └── api-configurations.md
├── workflows/
│   ├── 01_analytics_weekly_digest.json
│   ├── 02_gmail_thread_miner_crm_sync.json
│   ├── 03_review_queue_router.json
│   ├── 04_social_outstand_pipeline.json
│   ├── 05_encharge_behavioral_nurture.json
│   ├── 06_finance_airwallex_to_wave.json
│   └── 00_global_error_alerting.json
├── schemas/
│   ├── analytics_digest_output.json
│   ├── email_extraction_schema.json
│   └── wave_import_format.json
└── .env.example

```

---

## 7. Setup & Runbook Quickstart

### 1. Import Workflow Templates

Import all JSON files located in the `/workflows` directory into your n8n Cloud instance.

### 2. Configure Credentials

Navigate to **Credentials** in n8n and set up the corresponding accounts:

* Anthropic API (Claude)


* HubSpot OAuth2 / Private App Token


* Google Workspace OAuth2 (Gmail Read, Google Search Console, GA4)


* Encharge API Key


* Outstand REST API Key


* Notification Webhooks (Slack/Email)



### 3. Verification & Acceptance Testing

* Trigger the `01_analytics_weekly_digest` manually and verify that the combined analytics digest delivers to the admin inbox with proper formatting.


* Test the approval gate using a test contact to confirm drafts land in the review queue with HubSpot properties updated.
