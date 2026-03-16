# Fuel iX — Threads, Copilots & Files

> **Source**: docs.fuelix.ai — User Guides (March 2026)  
> **Tags**: #fuelix #ai #copilot #threads #files #rag #knowledge-hub

---

## 🧵 Threads (Conversation Management)

**Threads** are the individual chat conversations you have with any Fuel iX Copilot or the Employee Assistant. Each thread is a persistent, manageable conversation session.

### Chat Experience
- **Long Message Handling**: Extended text wraps properly; scroll bars appear automatically for very long messages
- **Enhanced Markdown Rendering**: Responses render code blocks (with syntax highlighting), tables, ordered/unordered lists, and block quotes correctly

### During a Conversation
| Action | How |
|--------|-----|
| **Regenerate response** | Click the refresh icon next to any response |
| **Stop generation** | Click the "Stop" button while the AI is responding |
| **Copy response** | Click the "Copy" button next to any response |

### All History
The **All History** page is a centralized hub for all your conversations across all copilots.

- **Access**: Main menu → "All History"
- **Unified view**: Shows chat titles, copilot/assistant name, and timestamps
- **Pinned chats**: Star ⭐ important conversations to pin them to the top
- **Rename**: Hover → pencil icon → edit title
- **Delete**: Hover → trash icon → confirm (⚠️ cannot be recovered)
- **Continue**: Click any conversation to resume it

> 💡 **Best Practices**: Use descriptive titles, pin crucial conversations, and periodically clean up old threads.

### Hidden Chat History
Admins can enable **Hide Chat History** for security-sensitive teams:
- Conversations are not accessible after you start a new one or log out
- The All History page is disabled
- Save important outputs externally using the Copy Response feature

### Prism Workspace (in Threads)
When using Copilot Prisms (Generative UI), you can manage UI artifacts within a thread:
- Enable/disable Generative UI per conversation
- Access generated UIs through the Prism module
- To enable: Click copilot name at top → "Manage the current chat" → toggle "Generative UI"

---

## 🤖 Copilots

Copilots are personalized AI assistants built on top of foundation models. There are three types:

| Type | Created By | Visible To | Best For |
|------|-----------|------------|----------|
| **Custom** | Any user | Private or shared | Personal/team-specific tasks |
| **Community** | Any user | All org users | Sharing useful tools org-wide |
| **Official** | Admins only | Entire organization | Standardized, approved AI tools |

### Creating a Custom Copilot
1. Go to the Fuel iX Copilots page → click **"+ Create your own copilot"**
2. **Basic Settings**: Name, description, image, and system prompt
3. **Advanced Settings**: Model selection, tools, conversation starters
4. **Knowledge tab**: Upload documents or connect external data sources
5. **Apps tab**: Connect external integrations (Atlassian, Jira, etc.)

### System Prompt
The system prompt is the core of your copilot — it defines:
- **Role & persona** (who the copilot is)
- **Behavior rules** (how it should respond)
- **Restrictions** (what it should NOT do)

> 💡 Use the Fuel iX Employee Assistant to help you write a great system prompt!

### Model Selection
| Option | Description |
|--------|-------------|
| **Model Family** | Auto-updates to latest evaluated model (Anthropic, OpenAI, Google) |
| **Pinned Model** | Locks to a specific version — best for compliance/consistency |

**Available model families:**
- **Anthropic** (default): Claude 3.5 Sonnet — great for context & nuanced responses
- **OpenAI**: GPT-4 — strong reasoning & creative problem-solving
- **Google**: Gemini 1.5 Pro — cutting-edge language understanding

### Available Tools
| Tool | What It Does |
|------|-------------|
| 🖼️ Image Creator | AI-powered image generation |
| 🔍 Search Internet | Real-time web search |
| 🎨 Generative UI | Generate and render UI interfaces in conversation |
| 📷 Image Analysis | Understand and extract info from images |
| 📁 File Upload | Upload documents for in-conversation analysis |
| 🧠 Reasoning | Step-by-step thinking before responding |
| 🕐 Date and Time | Adds current date/time awareness |
| 💻 Code Execution *(Beta)* | Run code, analyze data, generate files |
| 🔎 Query Files *(Beta)* | Query entire content of uploaded documents |

### Conversation Starters
Up to **5 suggested prompts** that appear above the chat input to guide users:
- Keep them concise and aligned with the copilot's purpose
- Use placeholders like `[topic]` or `[campaign name]` that users can customize
- Update periodically based on common user interactions

### Managing Copilots (My Copilots Hub)
| Tab | What It Shows |
|-----|--------------|
| **My Copilots** | Copilots you created |
| **Starred** | Copilots you've favorited |
| **Shared** | Copilots others shared with you |
| **Team** | Copilots available to your whole team |
| **Recent** | Recently accessed copilots |

### Access & Sharing
- **Restricted** (default): Only visible to you
- **Shared**: Share with specific individuals or teams (viewers by default; can be upgraded to Admin)
- **Community**: Published org-wide
- **Official**: Admin-promoted, available to all users

> ⚠️ Only the **owner** can delete a copilot. Admins can edit but not delete.

### Upgrading to Official
If a Custom Copilot proves valuable org-wide → contact your admin → they can convert it to an Official Copilot.

---

## 📁 Files (Knowledge Hub)

The **Knowledge Hub** is how you give Copilots access to your organization's documents and data. There are two distinct approaches:

### Two RAG Approaches

| | Conversational RAG | Copilot Knowledge |
|--|-------------------|------------------|
| **Scope** | Temporary — current conversation only | Permanent — all conversations |
| **Best for** | Quick one-time document analysis | Building specialized AI assistants |
| **Use cases** | Ad-hoc research, reviewing a report | HR policies, product docs, SOPs |

### File Upload Limits (Both Approaches)
| Limit | Value |
|-------|-------|
| Max file size | 30 MB per file |
| Supported formats | PDF, DOCX, TXT, PPTX |
| Max per batch | 20 files |
| Additional batches | ✅ Allowed |

---

### 📎 Conversational RAG (Temporary File Upload)

Upload files directly into a chat conversation for temporary analysis:

1. Click the **file upload button** in the chat interface
2. Select up to 20 files from your device (PDF, TXT, DOCX)
3. Files are processed and added to the conversation context
4. Upload additional batches of 20 as needed

**Upload from Google Drive:**
1. Click file upload → select "Upload from Google Drive"
2. Authenticate with any Google account (doesn't need to match Fuel iX login)
3. ⚠️ Check the permission box in step 3 during auth
4. Browse and select files → they're added to the conversation

**Resync from Google Drive:**
- Look for the Google Drive icon on previously uploaded files
- Click "Resync" to pull the latest version

**Manage Google Drive connection:**
- View/disconnect: Settings → Profile → Connections

---

### 🧠 Copilot Knowledge (Permanent Knowledge Base)

Build a persistent knowledge base for your copilot:

1. Navigate to your copilot's settings → **Knowledge tab**
2. Upload relevant documents (up to 20 files per batch)
3. Repeat for larger knowledge bases
4. Click **"Save changes"**

> 💡 You can continue configuring the copilot while documents process in the background — but the copilot won't use new knowledge until processing is complete.

**Best Practices:**
- Use well-structured documents with clear headings
- Name files descriptively (e.g., `sales_summary_2026-03.pdf` not `report.pdf`)
- Keep files focused — one topic per document works better
- Regularly review and remove outdated content

---

### 🔗 External Database Integrations

#### Google Drive Folder Sync
Connect entire Google Drive folders to a copilot's knowledge base:

- Connect **unlimited folders** to a single copilot
- Bulk add/remove multiple folders at once
- Folder hierarchy view with file type icons and update dates
- **Auto-sync**: Every 24 hours (if enabled by admin)
- **Manual sync**: Click the sync button anytime

**Setup:**
1. Copilot settings → Knowledge tab → External databases → Google Drive → "Add folder"
2. Authenticate (any Google account)
3. Browse and select folder
4. Click "Save changes"

> ⚠️ Files over 30MB or in unsupported formats are skipped during sync.

#### SharePoint Integration
Connect Microsoft SharePoint folders to a copilot's knowledge base:

**Requires:**
1. **Admin setup**: Organization admin must configure SharePoint in External Connections settings
2. **User connection**: Profile → Connections → "Connect SharePoint" → Microsoft auth
3. **Copilot setup**: Knowledge tab → External databases → SharePoint → "Add folder"

> ⚠️ You must **save/create the copilot first** before adding a SharePoint folder.

#### Auto-Sync (Both Google Drive & SharePoint)
| Setting | Details |
|---------|---------|
| Frequency | Every 24 hours |
| What syncs | New files, updated files, deleted files |
| Prerequisite | Admin must enable "Allow Auto-Sync" in External Connections |
| Toggle | Copilot Knowledge tab → folder → toggle Auto-Sync On/Off |

**Use Auto-Sync when:** Documents change frequently (policies, product specs, training materials)  
**Use Manual Sync when:** Documents are stable or you want to control when updates happen

---

## 🔄 Two Google Drive Integrations — Key Difference

| | Conversation File Upload | Copilot Knowledge Folder Sync |
|--|------------------------|-------------------------------|
| **What** | Individual files for one conversation | Entire folders for permanent knowledge |
| **Scope** | Temporary (current conversation only) | Permanent (all conversations) |
| **Limit** | 20 files per batch | No batch limit |
| **Managed via** | Chat interface upload button | Copilot's Knowledge tab |

---

## 📚 Related Notes
- [[Fuel iX — Platform Overview]]
- [[Fuel iX — How to Create a Copilot]]
- [[Fuel iX — My Project — Sales Intelligence Copilot]]
