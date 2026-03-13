# Fuel iX — How to Create a Copilot

> **Source**: Research conducted March 2026 from docs.fuelix.ai  
> **Tags**: #fuelix #copilot #how-to #agent #rag #no-code

---

## 🎯 What You're Building

A **Copilot** is a personalized AI assistant you configure for a specific purpose. It combines:
- A **system prompt** (the AI's role and rules)
- A **knowledge base** (documents/data it can reference)
- **Tools** (built-in capabilities)
- **Apps** (external integrations via MCP)

Anyone with a Fuel iX account can create one — **no coding required**.

---

## 🚀 The 3-Step Creation Process

### Step 1: Setup — Name, Image & System Prompt

Navigate to **app.fuelix.ai → Create Copilot**

| Field | What to do |
|-------|-----------|
| **Name** | Give it a clear, descriptive name (e.g., "Sales Intelligence Assistant") |
| **Avatar** | Upload an image or pick an icon |
| **System Prompt** | Write the AI's role, behavior rules, and expertise in plain English |

#### ✍️ Writing a Great System Prompt

The system prompt is the most important part. It defines:
- **Who the Copilot is** (role, persona)
- **What it knows** (domain expertise, data it has access to)
- **How it should respond** (tone, format, level of detail)
- **What it should NOT do** (guardrails, out-of-scope topics)

**System Prompt Template:**
```
You are a [ROLE] for [TEAM/AUDIENCE].

Your purpose is to [PRIMARY GOAL].

You have access to [DATA/KNOWLEDGE SOURCES], which includes:
- [Data source 1]
- [Data source 2]

When answering:
- [Behavior rule 1]
- [Behavior rule 2]
- [Behavior rule 3]

You do NOT [restriction 1] or [restriction 2].
```

**Tips for a strong system prompt:**
- Be specific about the audience ("marketing professionals who are not data analysts")
- Define the output format ("always present numbers with $ for revenue, % for changes")
- Set clear boundaries ("only answer questions about sales data — redirect other topics")
- Include context about data freshness ("data is updated daily each morning")

---

### Step 2: Configure — Model, Tools & Advanced Settings

#### Choose Your AI Model
| Model | Best For |
|-------|---------|
| **Claude Sonnet 4** | Complex reasoning, long documents (1M token context) |
| **GPT-4o** | General purpose, strong at structured data |
| **Gemini 1.5 Pro** | Multimodal tasks, Google ecosystem |
| **Claude Haiku 4.5** | Fast responses, high-volume use cases |

#### Enable Tools
Toggle on the tools your Copilot needs:

| Tool | When to Enable |
|------|---------------|
| 🔍 **Search Internet** | When Copilot needs real-time or external info |
| 📁 **File Upload** | When users need to upload docs in conversation |
| 🧠 **Reasoning** | For complex analytical tasks (shows thinking steps) |
| 🕐 **Date & Time** | Always useful for time-sensitive data |
| ⏰ **Scheduled Tasks** | For automated recurring reports (Alpha) |
| 🎨 **Copilot Prisms** | If you want the AI to generate visual UI |
| 🖼️ **Image Creator/Analysis** | For visual content tasks |

#### Advanced Settings
- **Temperature**: Lower (0.1–0.3) for factual/data tasks; Higher (0.7–0.9) for creative tasks
- **Max tokens**: Set response length limits
- **Guardrails**: Restrict topics, languages, or content types

---

### Step 3: Add Knowledge — RAG & Data Sources

This is where you give the Copilot its "memory" — the data it can reference when answering questions.

#### Option A: Upload Files Directly
- Drag and drop files into the Knowledge Hub
- Supported: **PDF, DOCX, TXT, PPTX**
- Max: 30MB per file, 20 files per batch
- Best for: static reference documents, policies, product catalogs

#### Option B: Connect Google Drive (Recommended for dynamic data)
1. Go to **Knowledge Hub → Add Source → Google Drive**
2. Authenticate with your Google account
3. Select the folder you want to sync
4. Fuel iX will **auto-sync every 24 hours**
5. New/updated files in that folder are automatically indexed

> 💡 **This is the key integration for the Sales Intelligence Copilot** — export daily summaries from BigQuery to a Google Drive folder, and the Copilot stays current automatically.

#### Option C: Connect Microsoft SharePoint
1. Requires admin setup (Microsoft Entra app registration)
2. Provide: Client ID, Tenant ID, Client Secret, SharePoint URL
3. Per-user OAuth authentication
4. Auto-sync every 24 hours

#### How RAG Works Under the Hood
```
User asks a question
        ↓
Fuel iX searches the Knowledge Base for relevant chunks
        ↓
Top matching chunks are injected into the LLM context
        ↓
LLM generates an answer grounded in your documents
        ↓
User gets a response based on YOUR data, not just training data
```

**RAG Best Practices:**
- Use **well-structured documents** (clear headings, tables, consistent formatting)
- **Name files descriptively** (e.g., `sales_summary_2026-03-13.pdf` not `report.pdf`)
- Keep files **focused** — one topic per document works better than giant mixed files
- For tabular data, **PDF tables** work reasonably well; structured markdown is even better

---

## 📤 Sharing Your Copilot

| Sharing Level | How | Who Sees It |
|--------------|-----|-------------|
| **Private** | Default | Only you |
| **Shared** | Share link or invite specific users | Named individuals/teams |
| **Community** | Publish to Community | All users in your org |
| **Official** | Admin promotes it | Entire organization (pinned) |

> 💡 For a marketing team tool, start as **Shared** (invite your team), then ask an admin to make it **Official** once it's proven valuable.

---

## ⏰ Scheduled Tasks (Alpha Feature)

You can automate your Copilot to run tasks on a schedule and deliver results by email:

**Example use cases:**
- "Every Monday at 8 AM, summarize last week's sales performance and email it to me"
- "Daily at 7 AM, check the Google Drive folder for new sales reports and create a summary"
- "Every Friday at 5 PM, compare this week's sales to last week and highlight anomalies"

**How to set up:**
1. Enable **Scheduled Tasks** in Advanced Settings
2. In the chat, tell the Copilot: *"Schedule a task to [do X] every [day/week] at [time]"*
3. Confirm the details when prompted
4. The task runs automatically and delivers results to your conversation + email

**Limitations (Alpha):**
- Max 10 active tasks per user
- Times must be on 15-minute intervals (:00, :15, :30, :45)
- Not all MCP integrations supported yet

---

## 🔌 Connecting External Apps (MCP)

For advanced integrations beyond documents:

1. Go to **Apps Marketplace** in your Copilot settings
2. Browse available MCP integrations
3. Connect and authenticate
4. The Copilot can now call those external systems as tools

**Available integrations include:** Jira, GitHub, Slack, Google Workspace tools, and custom MCP servers registered by your organization.

---

## ✅ Copilot Creation Checklist

- [ ] Clear, descriptive name chosen
- [ ] System prompt written (role, data access, behavior rules, restrictions)
- [ ] Right LLM model selected for the use case
- [ ] Relevant tools enabled
- [ ] Knowledge base populated (files uploaded or Google Drive/SharePoint connected)
- [ ] Tested with 10+ realistic user questions
- [ ] Shared with the right audience
- [ ] Scheduled tasks set up (if needed)

---

## 📚 Related Notes
- [[Fuel iX — Platform Overview]]
- [[Fuel iX — My Project — Sales Intelligence Copilot]]
