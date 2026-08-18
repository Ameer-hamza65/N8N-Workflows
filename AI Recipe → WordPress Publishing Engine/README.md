# AI Recipe → WordPress Publishing Engine

An automated, human-in-the-loop content generation and publishing pipeline built in **n8n**. This engine fetches recipe concepts from Google Sheets, generates localized Dutch culinary content and food photography using OpenAI models, handles interactive editorial reviews over Telegram, and publishes the validated recipes to a WordPress custom post type (`recept`) with full Advanced Custom Fields (ACF) support.

---

## Architecture Overview

```text
[ Schedule Trigger / Cron ]
             │
             ▼
[ Google Sheets: Fetch Pending Queue ] ──► [ Filter Today's Batch (Breakfast, Lunch, Dinner, Snack) ]
             │
             ▼
[ OpenAI Chain: Generate Dutch Recipe Text ]
             │
             ▼
[ Loop Over Items ] ◄──────────────────────────────────────────────────────────────────┐
  │                                                                                    │
  ▼                                                                                    │
[ AI Image Stylist & Food Photo Generation ]                                           │
  │                                                                                    │
  ▼                                                                                    │
[ Telegram: Send Preview & Review Prompt ]                                             │
  │                                                                                    │
  ▼                                                                                    │
[ Wait Node (Webhook Resume Token) ] ◄── [ Telegram Webhook Trigger (User Reply) ]     │
  │                                                                                    │
  ├─► [ LLM Intent Classifier ] ──► [ Switch ]                                         │
  │                                   ├── [ change_photo ]  ──► (Rerun Stylist/Photo) ─┤
  │                                   ├── [ change_recipe ] ──► (Rerun Text/Prompt)  ──┤
  │                                   ├── [ skip ]          ──► (Skip to next item)   ─┤
  │                                   └── [ approve ]                                  │
  │                                           │                                        │
  ▼                                           ▼                                        │
[ Timeout (24h) ──► Skip & Notify ]     [ Format Image Binary ]                        │
                                              │                                        │
                                              ▼                                        │
                                        [ WordPress: Upload Media ]                    │
                                              │                                        │
                                              ▼                                        │
                                        [ WordPress: Create 'recept' Post ]            │
                                              │                                        │
                                              ▼                                        │
                                        [ WordPress: Patch ACF Custom Fields ]         │
                                              │                                        │
                                              ▼                                        │
                                        [ Google Sheets: Log URL & Mark Used ]         │
                                              │                                        │
                                              ▼                                        │
                                        [ Gmail: Dispatch Notification Email ] ────────┘

```

---

## Key Features

* **Automated Batch Ingestion:** Daily cron trigger checks the Google Sheet queue (`Receptenwachtrij`) for `pending` ideas across distinct meal types (Breakfast, Lunch, Dinner, Snack).
* **Localized Dutch Copywriting:** Generates SEO-optimized recipe titles, ingredients, numbered cooking instructions, prep/cooking times, macronutrients, and dietitian tips in strict JSON.
* **Dynamic Image Direction:** Translates dish concepts and reviewer feedback into detailed image generation prompts before generating photorealistic food assets.
* **Human-in-the-Loop via Telegram:**
* Dispatches generated images and recipe titles directly to an editorial reviewer on Telegram.
* Captures the execution resume URL in workflow static data to enable asynchronous execution resumption upon reply.
* Classifies natural language feedback into four operational intents: `approve`, `change_photo`, `change_recipe`, or `skip`.
* Features an automated 24-hour timeout safeguard that clears pending execution state and releases the queue.


* **Two-Step WordPress & ACF Publishing:**
* Uploads AI-generated image binaries to the WordPress Media Library with custom slug formatting.
* Creates custom post type `recept` entries linked to categories and featured media.
* Executes a secondary REST API patch for Advanced Custom Fields (`recipe_preparation_time`, `recipe_carbs`, `recipe_protein`, etc.) to bypass REST hook race conditions.


* **Auditability & Notifications:** Logs live published URLs with Brussels timestamps to Google Sheets and dispatches an HTML email report via Gmail.

---

## Workflow Step-by-Step Breakdown

### 1. Ingestion & Pre-Processing

* **Schedule Trigger:** Triggers daily (configured via Cron).
* **Get Recipe Ideas & Select Todays Batch:** Reads rows from the Google Sheet spreadsheet, filters for rows with `status = 'pending'`, and isolates one item per meal slot.
* **Define Meal Types:** Attaches WordPress category taxonomy IDs (`Breakfast: 5`, `Lunch: 6`, `Dinner: 26`) and preserves sheet row numbers for status updating.

### 2. Recipe & Visual Asset Generation

* **Generate Recipe Text (AI):** Uses an LLM chain with strict system prompts to output Dutch JSON containing full recipe instructions, nutritional information, and structured metadata.
* **Loop Over Items:** Batches processing one recipe at a time.
* **Smart Image Stylist (AI):** Evaluates recipe metadata and any revision feedback to build high-end culinary photography prompts.
* **Create Food Photo (AI):** Generates 1024x1024 food imagery matching brand visual guidelines.

### 3. Review & Feedback Routing

* **Send a photo message & Send Review Request:** Dispatches the generated photo and meal title to the designated Telegram chat.
* **Store Resume URL & Wait For Reply:** Caches `$execution.resumeUrl` into workflow static data and suspends execution for up to 24 hours awaiting a webhook response.
* **Telegram Listener:** Listens for incoming chat messages, validates the sender chat ID, forwards user feedback to the resume URL, and triggers resumption.
* **Message a model (Classifier) & Switch:** Evaluates reviewer intent:
* `approve`: Continues straight to publishing.
* `change_photo`: Routes back to **Smart Image Stylist** with the exact styling feedback.
* `change_recipe`: Routes to **Extract Recipe Feedback** and **Regenerate Recipe Text (AI)** before updating the photo.
* `skip`: Aborts the current item and pulls the next recipe from the batch loop.


* **Handle Review Timeout:** Triggers if 24 hours elapse with no reviewer input, clearing memory and notifying the channel.

### 4. Publishing & Synchronization

* **Format Image Files:** Pulls binary buffer data from the generation node and assigns standard `.png` file names.
* **Upload to Media Library:** Performs a multipart POST to `/wp-json/wp/v2/media`.
* **Publish Live Recipe:** Creates the custom post `/wp-json/wp/v2/recept` in `publish` status with assigned category taxonomy and initial metadata.
* **Retry ACF Fields (Separate Call):** Issues a dedicated secondary POST to `/wp-json/wp/v2/recept/{id}` to ensure all ACF nutritional values, prep/cook times, and HTML list blocks are saved.
* **Log to Google Sheets & Mark Recipe Used:** Updates the tracking spreadsheet (`Recipe Generation Logs`) with published post URLs and marks the corresponding row status to `used`.
* **Send a message:** Emails the team with live links and publication summaries.

---

## Prerequisites & External Services

| Service | Requirement | Purpose |
| --- | --- | --- |
| **n8n** | Self-hosted or Cloud (v1.x+) | Workflow automation engine |
| **OpenAI API** | API Key (`gpt-5-mini`, `gpt-5.4-mini`, image model) | Text copywriting, image styling, intent routing, and photo generation |
| **WordPress** | Application Password / REST API credentials | Custom post creation (`/wp/v2/recept`), media upload, and ACF management |
| **Telegram Bot** | Bot Token & Chat ID | Real-time preview delivery and conversational approval interface |
| **Google Sheets** | OAuth2 Client Credentials | Recipe concept queuing and publication audit logging |
| **Gmail** | OAuth2 Client Credentials | Post-publication email dispatch |

---

## Configuration & Setup

1. **Import the Workflow:**
* In your n8n dashboard, click **Add Workflow** > **Import from File / JSON** and paste the workflow JSON.


2. **Configure Credentials:**
* Attach your OpenAI API key to all OpenAI/LangChain nodes.
* Connect your Google Service Account / OAuth2 credentials to the **Google Sheets** nodes.
* Attach your Telegram Bot token to **Telegram Listener**, **Send a photo message**, and related nodes.
* Add your WordPress application username and password to the **HTTP Request** nodes.
* Configure Gmail OAuth2 for the notification node.


3. **Verify Environment IDs & URLs:**
* Ensure the spreadsheet IDs match your Google Drive setup.
* In `Define Meal Types`, adjust category IDs to match your WordPress installation's `recept_categorie` taxonomy.
* In Telegram nodes and `Build Resume Payload`, update `chatId` to your target Telegram user or channel ID.
* Ensure your WordPress endpoint URLs point to your target domain.
