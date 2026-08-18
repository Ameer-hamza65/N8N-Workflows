# TAVLI Diamonds — Production Social Video Automation

An enterprise-grade, event-driven video generation and multi-platform publishing engine built in **n8n**. The pipeline automates the generation of ultra-luxury short-form video content for TAVLI Diamonds using **Google Gemini** for copywriting, **HeyGen API v3** for digital avatar rendering, and **GoHighLevel (GHL) v3 API** for multi-account social syndication (TikTok, Pinterest, YouTube Shorts, etc.).

---

## 1. System Architecture

```text
[ Daily Cron Schedule / Trigger ]
               │
               ▼
[ Gemini LLM: Script, Hook & Caption Generation ]
               │
               ▼
[ Schema & Word-Count Validation (Loop <= 3) ] ──(Fail > 3)──► [ Operational Alert Webhook ]
               │ (Valid)
               ▼
[ Look Selection & Dynamic Engine Resolution (avatar_v / avatar_iv) ]
               │
               ▼
[ Create Context State in n8n Data Table ]
               │
               ▼
[ HeyGen API v3: Trigger Video with Idempotency Key ]
               │
               │ (Asynchronous Video Generation)
               ▼
[ HeyGen Webhook: avatar_video_caption.success ]
               │
               ▼
[ HMAC Signature Check & In-Memory Lock (State = PROCESSING) ]
               │
               ▼
[ Query Authoritative Status & Binary Download ]
               │
               ▼
[ Binary Integrity Guard (>100KB, MIME: video/mp4) ]
               │
               ▼
[ Upload to GoHighLevel (GHL) Media Storage ]
               │
               ▼
[ Verify Live CDN Media URL ]
               │
               ▼
[ GoHighLevel v3: Publish Multi-Account Post ]
               │
               ▼
[ Update State Lock to DONE & Save Publication Record ]

```

---

## 2. Core Operational Pillars

### 1. Ultra-Luxury Brand Copywriting (Google Gemini)

* **Audience Targeting:** Silently directs scripts toward three distinct Ultra-High-Net-Worth demographics: *Yachting Elite*, *Legacy Guardians*, and *Tech Vanguard*.
* **Strict Constraints:** Enforces a 60–90 word range, unhurried pacing, conversational tone, hook openers, non-salesy calls to action, and strict JSON output formatting.
* **Resilience Loop:** If the LLM generates non-compliant JSON or breaks word limits, the workflow increments an internal state counter and retries up to 3 times before dispatching an incident alert.

### 2. Intelligent Avatar & Motion Engine Dispatch (HeyGen API v3)

* **Dynamic Engine Compatibility:** Queries the HeyGen API at runtime to check if an avatar look supports `avatar_v` or `avatar_iv`. It assigns optimal settings dynamically (e.g., adding `expressiveness: 'high'` only to `avatar_iv` to avoid 400 validation errors).
* **Dynamic Motion Injection:** Randomly assigns natural-language motion prompts (e.g., hand gestures, deliberate pauses, subtle head tilts) to prevent static avatar output.
* **Idempotent Job Requests:** Issues custom UUID-based `Idempotency-Key` headers on render requests to avoid duplicate billing and redundant jobs.

### 3. Canonical Webhook Ingestion & State Locking

* **Instant 200 OK Acknowledgment:** Returns an immediate HTTP 200 to HeyGen to prevent webhook retries.
* **HMAC Signature Verification:** Verifies raw bytes against cryptographic headers for security.
* **Canonical Event Filtering:** Drops intermediate render events in <5ms, only continuing on `avatar_video_caption.success`.
* **Concurrency Lock:** Acquires a `tavli:lock:{videoId}` inside n8n Data Tables. If another concurrent execution is already processing the video, execution halts safely.

### 4. Binary Integrity Guard & GHL Social Syndication

* **Corruption Prevention:** Validates that downloaded video streams are valid `video/mp4` containers and exceed the minimum 100KB size threshold.
* **Permanent Cloud Hosting:** Pushes video assets directly to GoHighLevel Media Storage (`/medias/upload-file`) and verifies HTTP 200 read access before post scheduling.
* **Multi-Account Posting:** Dispatches the final post simultaneously across multiple connected business profiles and handles YouTube Shorts metadata.

### 5. Dedicated Disaster Recovery & Re-conciliation Webhook

* **Endpoint:** `/webhook/tavli-recovery`
* **Functionality:** Authenticates recovery tokens, inspects previous video generation states, and performs idempotent search scans on GHL's `/posts/list` to confirm whether a post was published during network drops.

---

## 3. Technology Stack

* **Workflow Engine:** n8n Cloud
* **AI Copywriting:** Google Gemini (`@n8n/n8n-nodes-langchain.lmChatGoogleGemini`)
* **Avatar & Speech Synthesis:** HeyGen API v3 (`avatar_v` / `avatar_iv`, ElevenLabs voice integration)
* **Social Distribution & CDN:** GoHighLevel (LeadConnector) API v3
* **State Management:** n8n Internal Data Tables (`tavli_state`, `tavli_looks`)
* **Security & Auth:** HMAC SHA256 Signature Verification, Header-based Auth Tokens

---

## 4. n8n Data Table State Schemas

### `tavli_state` Key Conventions

| Key Format | Sample Value | Purpose |
| --- | --- | --- |
| `tavli:job:{jobId}` | `{"status": "MEDIA_READY", "ghl_media_url": "https://..."}` | Main execution lifecycle context. |
| `tavli:video:{videoId}` | `{"job_id": "tavli_xxx", "created_at": "..."}` | Alias map linking HeyGen video IDs to job records. |
| `tavli:lock:{videoId}` | `PROCESSING` or `DONE` | Concurrency guard preventing duplicate social posts. |
| `tavli:published:{videoId}` | `{"post_id": "ghl_post_xxx", "published_at": "..."}` | Permanent audit log of published social posts. |

---

## 5. Environment Variables & Credentials Setup

Add the following credentials and variables in your n8n workspace:

### Required Credentials

* **Google Gemini (PaLM) API:** Attached to `Google Gemini Chat Model1`
* **HeyGen API:** Configured under `heyGenApi` for HTTP Request nodes
* **GoHighLevel API:** Header Auth configured with `Version: v3` and `Authorization: Bearer <GHL_TOKEN>`

### Environment Variables

* `TAVLI_RECOVERY_KEY`: Secure secret token required to invoke the disaster recovery webhook.

---

## 6. Repository Structure

```text
tavli-video-automation/
├── README.md
├── workflows/
│   ├── tavli_production_social_engine.json
│   └── tavli_look_factory.json
├── prompts/
│   ├── luxury_copywriter_gemini.txt
│   └── motion_prompts.json
└── docs/
    ├── disaster-recovery-guide.md
    └── state-machine-reference.md

```

---

## 7. Setup & Deployment

1. **Import Workflow:** Load the workflow JSON into your n8n Cloud instance.
2. **Bind Data Tables:** Ensure that the `tavli_state` and `tavli_looks` Data Tables exist in your n8n project.
3. **Verify Webhook Paths:** Ensure the HeyGen developer console points to your production webhook URL: `https://<your-instance>.app.n8n.cloud/webhook/heygen-webhook`.
4. **Activate Trigger:** Turn on `Schedule Trigger1` to run daily automated productions.
