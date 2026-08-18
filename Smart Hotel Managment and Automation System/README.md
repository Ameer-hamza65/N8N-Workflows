# Smart Hotel Management & Automation System

An event-driven, multi-workflow AI automation platform that coordinates guest communications, housekeeping operations, IoT smart-room hardware, and administrative records.

> **Architecture Summary:** The system combines AI agents for natural-language understanding and decision-making with deterministic **n8n** workflows for executing reliable actions across communication channels, physical devices, staff workflows, and operational tracking databases.

---

## 1. System Architecture


                    ┌─────────────────────────┐
                    │      Guest Request      │
                    │   (Gmail / Telegram)    │
                    └────────────┬────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │   AI Classification &   │
                    │      Router Agent       │
                    └────────────┬────────────┘
                                 │
         ┌───────────────────────┼───────────────────────┐
         │                       │                       │
         ▼                       ▼                       ▼
┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│  Smart Devices   │    │   Housekeeping   │    │    Guest FAQ     │
│   (IoT Engine)   │    │  & Room Service  │    │  Knowledge Base  │
└────────┬─────────┘    └────────┬─────────┘    └────────┬─────────┘
         │                       │                       │
         ▼                       ▼                       ▼
  Heater / Lights        Cleaner Assigned            AI Response
  Set State via API              │                   Sent to Guest
                                 ▼
                        Response Monitoring
                                 │
                        ┌────────┴────────┐
                        │                 │
                        ▼                 ▼
                   [Responded]       [No Response]
                        │                 │
                        ▼                 ▼
                  Update Sheets    Voice Call Follow-Up


### Philosophy: Closed-Loop Execution

$$\text{Event} \longrightarrow \text{Understand} \longrightarrow \text{Decide} \longrightarrow \text{Act} \longrightarrow \text{Record} \longrightarrow \text{Follow-Up}$$

---

## 2. Core Automation Modules

### Module 1: Guest Lifecycle & Smart Room Automation

Automates energy consumption and environment preparation based on check-in and check-out events.

* **Check-In Event:** Validates reservation $\to$ Identifies assigned room $\to$ Sends API payload to turn **ON** heating and smart lighting $\to$ Logs execution.
* **Check-Out Event:** Identifies departure $\to$ Sends API payload to turn **OFF** appliances $\to$ Queues room for cleaning $\to$ Logs completion.


{
  "guest": "John Doe",
  "room": "204",
  "check_in": "2026-08-20",
  "check_out": "2026-08-24",
  "status": "checked_in"
}



---

### Module 2: AI Email & Intent Classification Agent

Parses incoming unstructured guest emails to extract context, target entities, and intents.


Incoming Email ──► Extract Text ──► AI Agent ──► Structured Output
                                                        │
                      ┌─────────────────────────────────┼────────────────────────┐
                      ▼                                 ▼                        ▼
              [Device Request]                  [Room Service]             [General FAQ]
                      │                                 │                        │
                      ▼                                 ▼                        ▼
              Smart Device API                 Housekeeping Alert       Knowledge Base Reply


**Parsed Intent Example:**


{
  "intent": "device_control",
  "device": "heater",
  "action": "off",
  "room": "204"
}



---

### Module 3: Housekeeping Coordination & Closed-Loop Escalation

Transforms raw booking schedules into dispatch tasks for cleaners, monitoring for acknowledgments and executing automated escalations if staff do not respond.

* **Schedule Dispatch:** Pulls turnover data and alerts staff with cleaning priorities via Telegram.
* **State Tracking:** Tracks lifecycle (`SCHEDULE_SENT` $\to$ `WAITING_FOR_RESPONSE` $\to$ `CONFIRMED` or `ESCALATED`).
* **Automated Escalation:** If an acknowledgment is not received within the timeout window, triggers an automated voice call to ensure turnover continuity.

---

## 3. Technology Stack

* **Workflow Orchestration:** n8n (Webhooks, Cron triggers, Sub-workflows)
* **AI & NLP:** LLMs (Zero-shot intent classification, RAG-based FAQ generation, entity extraction)
* **Communication Channels:** Gmail API, Telegram Bot API, Automated Voice Call APIs
* **IoT & Hardware Layer:** REST-driven Smart Device APIs (Thermostats, Smart Lighting)
* **Data & Auditing:** Google Sheets / Database state storage

---

## 4. Division of Responsibilities

| System Layer | Component | Functional Scope |
| --- | --- | --- |
| **Cognitive (AI)** | LLM Agent | Natural language understanding, intent extraction, dynamic FAQ generation. |
| **Deterministic** | n8n Orchestrator | Webhook ingestion, conditional routing, API payload formation, scheduled tasks. |
| **Physical (IoT)** | Hardware APIs | State switching for heaters, AC units, and lighting. |
| **Escalation** | Voice / Messaging | Multi-channel failover alerts for unacknowledged operations. |
| **Auditing** | Google Sheets | Operational run logging, response timestamps, room lifecycle tracking. |

---

## 5. Security & Governance

1. **Strict Authorization Scope:** AI extraction verifies room-to-guest bindings before dispatching hardware actions (e.g., Guest in Room 204 cannot toggle appliances in Room 305).
2. **Deterministic Payload Validation:** LLM output is validated against strict schema enums ($\text{action} \in \{\text{ON}, \text{OFF}\}$) before hitting device APIs.
3. **Zero-Trust Credential Storage:** API tokens, OAuth credentials, and webhook secrets are managed through environment variables and secure credential stores.

---

## 6. Repository Structure

```text
smart-hotel-management/
├── README.md
├── docs/
│   ├── architecture.md
│   ├── workflows.md
│   └── api-integrations.md
├── workflows/
│   ├── guest-arrival-automation.json
│   ├── guest-departure-automation.json
│   ├── guest-email-agent.json
│   ├── housekeeping-scheduler.json
│   ├── cleaner-response-monitor.json
│   ├── room-service-agent.json
│   └── smart-device-control.json
├── prompts/
│   ├── intent-classifier.txt
│   └── faq-system-prompt.txt
├── .env.example
└── LICENSE

```

---

## 7. Quickstart Setup

1. **Import Workflows:** Import the JSON templates from `/workflows` into your n8n instance.
2. **Configure Credentials:** Set up OAuth2/API tokens for Gmail, Telegram, Google Sheets, and your Smart Device endpoints inside n8n.
3. **Set Environment Variables:** Copy `.env.example` to `.env` and fill in necessary base URLs, webhook endpoints, and API tokens.
4. **Deploy Webhooks:** Activate the workflows to open the webhook listeners for guest emails, booking updates, and hardware events.

