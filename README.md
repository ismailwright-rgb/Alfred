# A.L.F.R.E.D — Autonomous LLM-Driven Executive Assistant

> A production-grade, voice-enabled multi-agent AI system built on n8n, deployed and running daily.

---

## What Is Alfred?

Alfred is a personal AI operations system based in Alfred Pennyworth from the batman series. the digital butler who is capable of anticapating your needs. — a Discord-native, voice-capable agent that orchestrates **9 specialized sub-agents** to manage communication, research, finance, productivity, and content. It is not a demo. It runs every day.

You speak or type a request via Discord. Alfred routes it to the right specialized agent, executes the task, and responds — in voice (ElevenLabs TTS) or text — within seconds.

---

## Architecture

```
Discord (text or voice message)
        │
        ▼
 Auth Filter (Discord user ID whitelist)
        │
        ▼
   Input Router (Switch node)
   ┌────┴────┐
  text     voice
   │          │
   │    ElevenLabs STT (transcription)
   └────┬────┘
        ▼
  ┌─────────────────────────────────────────┐
  │           A.L.F.R.E.D Orchestrator      │
  │         (LangChain Agent · OpenRouter)  │
  │                                         │
  │  Memory: PostgreSQL (persistent)        │
  │  Knowledge Base: Supabase Vector Store  │
  │  Embeddings: HuggingFace Inference      │
  │  Reasoning: Think tool                  │
  └──────────────┬──────────────────────────┘
                 │
     ┌───────────┼────────────┐
     ▼           ▼            ▼  ... (9 agents)
  News        Productivity  TrendScout
  Agent       Agent         Agent
     │           │            │
  Stocks      Doc Agent    Social Media
  Agent                    Agent
     │
  Career / Hustle / Spiritual / Reddit
        │
        ▼
  Set Reply Message
        │
   ┌────┴────┐
  Discord   ElevenLabs TTS
  Message        │
             fly.dev Voice Server → audio playback
```

---

## Sub-Agents

| Agent | Capabilities |
|---|---|
| **Productivity Agent** | Gmail read/send/reply, Calendar scheduling, Task management |
| **News Agent** | Hacker News, RSS feeds, real-time weather (Los Angeles) |
| **Stock Agent** | Real-time stock quotes, crypto prices, 24hr change data |
| **TrendScout Agent** | Market signal research, digital product opportunities, industry trend reports |
| **Career Agent** | Automated job discovery, role/location filtering, results posted to Discord |
| **Hustle Agent** | Freelance opportunity research, side income discovery |
| **Doc Agent** | Google Drive read/write, Google Sheets management, file review |
| **Social Media Agent** | Content creation and social media management |
| **Reddit** | Subreddit research, topic signal extraction |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Workflow Orchestration | n8n (self-hosted) |
| LLM Routing | OpenRouter |
| Voice Input | ElevenLabs Speech-to-Text |
| Voice Output | ElevenLabs TTS → fly.dev voice server |
| Interface | Discord (text + voice message attachments) |
| Persistent Memory | PostgreSQL (per-session chat history) |
| Knowledge Base | Supabase Vector Store (personal RAG) |
| Embeddings | HuggingFace Inference API |
| Document Storage | Google Drive / Google Sheets |
| Deployment | fly.dev (voice server), n8n cloud |

---

## Key Engineering Decisions

**Multi-agent orchestration over monolithic prompting**
Instead of one large prompt trying to do everything, Alfred routes tasks to specialized sub-agents. Each agent has a narrow, well-defined responsibility. This improves accuracy and makes each agent independently testable and upgradeable.

**PostgreSQL for persistent memory**
Session context is stored in Postgres, not in-memory. Alfred remembers context across conversations — goals, ongoing projects, preferences — without requiring the user to repeat themselves.

**Supabase Vector Store for personal RAG**
A personal knowledge base allows Alfred to retrieve relevant context about the user's goals, projects, and preferences before responding. This is retrieved via semantic search, not keyword matching.

**OpenRouter for model flexibility**
LLM calls are routed through OpenRouter rather than being hardcoded to a single provider. This allows model swapping without changing agent logic.

**ElevenLabs + fly.dev for voice loop**
Voice input is transcribed via ElevenLabs STT. Responses are synthesized back to audio via ElevenLabs TTS and streamed through a lightweight voice server deployed on fly.dev — enabling fully hands-free interaction.

**Auth filter at ingress**
A Discord user ID whitelist prevents unauthorized access before any LLM calls are made, protecting both compute costs and personal data.

---

## How to Use (Self-Hosted)

### Prerequisites
- n8n instance (self-hosted or cloud)
- Accounts: OpenRouter, ElevenLabs, Supabase, Discord Bot
- PostgreSQL database
- fly.dev account (for voice server)

### Setup
1. Import the workflow JSON into your n8n instance
2. Configure credentials for each service (Discord, OpenRouter, ElevenLabs, Supabase, PostgreSQL)
3. Set up the Supabase `agent_knowledge` table with pgvector extension
4. Deploy the voice server to fly.dev
5. Invite the Discord bot to your server and activate the workflow

### Usage
- Type a message in your Discord channel → Alfred responds in text
- Send a voice message in Discord → Alfred transcribes, processes, and responds in audio

---

## Example Requests

```
"Alfred, what's NVDA trading at and summarize today's AI news from Hacker News"
→ Stock Agent + News Agent called in parallel, combined response delivered

"Search for senior data engineer roles in Los Angeles"
→ Career Agent executes search, posts results to Discord channel

"Scout trends in AI productivity tools and find product opportunities"
→ TrendScout Agent + Reddit Agent research signals, returns structured opportunity sheet

"Draft a follow-up email to Marcus about the project proposal"
→ Productivity Agent retrieves contact, drafts HTML email, confirms before sending

"Update my goals doc in Drive with the notes I'm about to give you"
→ Doc Agent reads/writes to Google Drive
```

---

## Status

Alfred is actively running and used daily. It is not a demo or a proof of concept.

---

## Author

**Ismail Rogers-Wright**
[LinkedIn](https://linkedin.com/in/ismailrogerswright) · [Portfolio](https://portfoliowright.vercel.app) · [GitHub](https://github.com/ismailwright-rgb)
