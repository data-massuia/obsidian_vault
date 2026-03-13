# Fuel iX — Platform Overview

> **Source**: Research conducted March 2026 from fuelix.ai and docs.fuelix.ai  
> **Tags**: #fuelix #ai #telus #enterprise-ai #copilot #llm

---

## 🔥 What is Fuel iX?

**Fuel iX™** is an **enterprise generative AI platform** built by **TELUS Digital** (a subsidiary of TELUS, Canada's major telecom). It is designed to give organizations the tools to **manage, monitor, and deploy generative AI at scale** — safely and responsibly.

Think of it as an **AI operating system for enterprises**: it sits on top of foundation models (GPT-4o, Claude, Gemini, etc.) and gives you a unified layer to build AI applications, manage usage, enforce guardrails, and connect your own data.

### Key Stats (as of early 2026)
| Metric | Value |
|--------|-------|
| Active Users | 50,000+ |
| Custom Copilots in use | 18,000+ |
| Tokens processed (2025) | 2 trillion+ |
| Award | 2025 AI Breakthrough "Chatbot Platform of the Year" |
| Certification | ISO 31700-1 Privacy by Design |
| Data residency | Canadian borders |

### Platform URL
- **App**: https://app.fuelix.ai
- **Docs**: https://docs.fuelix.ai
- **Website**: https://fuelix.ai

---

## 🏗️ Core Products

### 1. 🤖 Copilots (Main Product)
The flagship product — personalized AI assistants that combine a system prompt, knowledge base, tools, and external integrations. Anyone can create one with no coding required.
→ See [[Fuel iX — How to Create a Copilot]]

### 2. 🛡️ Fortify (Red Teaming)
An AI security testing tool. Lets you run adversarial attacks against your AI models to find vulnerabilities before they're exploited. Useful for security teams validating AI deployments.

### 3. 🎓 Agent Trainer
A training simulation platform. Lets you create AI-powered training scenarios for employees (e.g., customer service agents practicing difficult conversations with an AI customer).

### 4. 🎧 Agent Assist (Beta)
Real-time AI assistance for live customer service agents — surfaces relevant information and suggested responses during live calls/chats.

---

## 🧠 LLM Integrations (20+ Models)

Fuel iX is **model-agnostic** — you can choose which foundation model powers each Copilot:

| Provider | Models Available |
|----------|-----------------|
| **OpenAI** | GPT-4o, GPT-4o mini, GPT-5.2 |
| **Anthropic** | Claude Sonnet 4, Claude Haiku 4.5 |
| **Google** | Gemini 1.5 Pro, Gemini Flash |
| **Meta** | Llama 3 |
| **Mistral** | Mistral Large |
| **DeepSeek** | DeepSeek R1 |
| **GLM** | GLM-4 |

**Context window**: Up to **1 million tokens** supported (with Claude Sonnet 4/4.5)

You can use Fuel iX's managed API keys or bring your own provider keys.

---

## 🤖 How Copilots Work

A Copilot is built from 4 components:

```
┌─────────────────────────────────────────────────────┐
│                    FUEL iX COPILOT                  │
│                                                     │
│  📝 System Prompt    → Defines role & behavior      │
│  📚 Knowledge Base   → RAG over your documents/data │
│  🔧 Tools            → Built-in capabilities        │
│  🔌 Apps (MCP)       → External system connections  │
└─────────────────────────────────────────────────────┘
```

### System Prompt
- Defines the Copilot's **personality, role, expertise, and rules**
- Written in plain English — no code needed
- Example: "You are a sales data analyst for the marketing team..."

### Knowledge Base (RAG)
- Upload documents the Copilot can reference
- Uses **Retrieval-Augmented Generation (RAG)** — finds relevant chunks and injects them into the LLM context
- Supported file types: **PDF, DOCX, TXT, PPTX**
- External sync: **Google Drive** and **Microsoft SharePoint** (auto-sync every 24h)
- Max file size: 30MB | Max 20 files per batch

### Tools (Built-in)
| Tool | What it does |
|------|-------------|
| 🔍 Search Internet | Real-time web search |
| 🖼️ Image Creator | AI image generation |
| 📷 Image Analysis | Understand/describe images |
| 📁 File Upload | Temporary document analysis in conversation |
| 🧠 Reasoning | Step-by-step thinking (visible to user) |
| 🕐 Date & Time | Current date/time awareness |
| ⏰ Scheduled Tasks (Alpha) | Automate recurring tasks, deliver via email |
| 🎨 Copilot Prisms | Generate full UI interfaces via natural language |

### Apps / MCP Integrations
- Fuel iX uses the **Model Context Protocol (MCP)** to connect Copilots to external systems
- MCP Servers can be scoped: **Public**, **Organization**, or **Team** level
- Examples: Jira, GitHub, Slack, custom internal APIs
- → See [[Fuel iX — My Project — Sales Intelligence Copilot]] for BigQuery integration options

---

## 👥 Types of Copilots

| Type | Who Creates | Who Can Use | Best For |
|------|-------------|-------------|----------|
| **Custom** | Any user | Private or shared with specific people | Personal productivity, team tools |
| **Community** | Any user | Published org-wide | Sharing useful tools across teams |
| **Official** | Admins only | Entire organization | Company-wide standardized assistants |

---

## 🔒 Security & Privacy

- **ISO 31700-1 Privacy by Design** certified — first AI platform to receive this
- **Canadian data residency** — data stays within Canadian borders
- **Guardrails**: Admins can set content restrictions, usage limits, and approved models
- **Usage monitoring**: Full analytics dashboard for admins
- **Responsible AI**: TELUS is a recipient of the Responsible AI Institute's Outstanding Organization 2023 prize

---

## 💡 Key Differentiators vs. Other AI Platforms

| Feature | Fuel iX | ChatGPT Enterprise | Microsoft Copilot |
|---------|---------|-------------------|-------------------|
| Canadian data residency | ✅ | ❌ | ❌ |
| No-code Copilot builder | ✅ | Limited | ✅ |
| Multi-LLM support | ✅ (20+ models) | ❌ (OpenAI only) | Limited |
| MCP integrations | ✅ | ❌ | Limited |
| Privacy by Design certified | ✅ | ❌ | ❌ |
| Built for enterprise scale | ✅ | ✅ | ✅ |

---

## 📚 Related Notes
- [[Fuel iX — How to Create a Copilot]]
- [[Fuel iX — My Project — Sales Intelligence Copilot]]
