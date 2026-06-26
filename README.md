<div align="center">

# CareerPilot

### AI-Powered Job Interview Preparation Agent with Live Dynamic UI

**CareerPilot** is not a chatbot. It is a stateful AI agent that builds your entire interview preparation dashboard live, inside the conversation — no page reload, no separate app, no starting over.

[Live Demo](#) · [Report Bug](issues) · [Request Feature](issues)


## The Problem

Over **1.5 million engineering graduates** enter India's job market every year. Most walk into their first technical interview completely unprepared — relying on scattered YouTube videos, generic question banks, and crowded WhatsApp groups.

Existing tools like InterviewBit and LeetCode offer static question banks but:
- Have **no memory** of your past sessions or mistakes
- Cannot **adapt questions** to your target company or role
- Don't understand **your resume** or skill gaps
- Treat every candidate **the same way**

**CareerPilot changes all of this.**

---

## What Makes CareerPilot Different

| Feature                      | Generic Tools        | CareerPilot                         |
|------------------------------|----------------------|-------------------------------------|
| Remembers past sessions      | ❌                   | ✅ Vector-based RAG memory         |
| Adapts to your resume        | ❌                   | ✅ AI resume parser + gap analyser |
| Company-specific questions   | ❌                   | ✅ TCS, Google, Infosys, Startups  |
| Dynamic UI mid-conversation  | ❌                   | ✅ Components render live in chat   |
| Adjusts difficulty per answer| ❌                   | ✅ Adaptive scoring engine         |
| Long-term progress tracking  | ❌                   | ✅ Persisted across all sessions   |
| Agent execution transparency | ❌                   | ✅ Full observability trace        |

---

## Core Features

### Stateful AI Agent Pipeline
A full agentic execution loop — every user message passes through:
1. **Intent Classifier** — detects what the user wants (mock interview, roadmap, resume review, etc.)
2. **Tool Orchestrator** — selects and runs the right tools in parallel
3. **RAG Synthesiser** — retrieves relevant long-term memory before generating a response
4. **JSON Renderer** — appends a structured payload that triggers live UI components
5. **Observability Logger** — traces every step with latency, token usage, and success state

### Vector-Based Long-Term Memory
- Embeds every interaction using **OpenAI text-embedding-3-large** (3072 dimensions)
- Stores memory chunks by type: `resume`, `interview`, `learning`, `career`
- MD5-cached embeddings to prevent re-computation
- Auto-prunes per-user memory at 500 chunks
- **The agent remembers your mistakes from last week and adapts today's session around them**

### Dynamic UI Rendering (Track 04 Core)
The AI response contains a hidden JSON payload alongside text. The frontend parser:
- Strips the JSON from the display text
- Routes it to the matching React component via a **whitelisted component registry**
- Renders the component **mid-conversation with Framer Motion animations**
- Zero page reload. Zero separate API call.

**9 live UI components:** `dashboardCard` · `readinessScore` · `topicChecklist` · `feedbackCard` · `roadmapCard` · `skillGapCard` · `interviewQuestionCard` · `memoryInsightCard` · `sessionSummaryCard`

### Resume Intelligence
- Upload PDF or DOCX resume
- AI extracts skills, experience, and education
- Generates a **skill gap analysis** against the target company's requirements
- Produces a **personalised day-by-day prep roadmap**
- All powered by your existing backend resume pipeline

### Adaptive Mock Interviews
- Questions tailored to role + company + experience level
- **5-dimensional evaluation:** correctness, completeness, clarity, confidence, communication
- Difficulty auto-adjusts — low score → easier follow-up · high score → harder next question
- Company-aware prompting: Google interviews differently from TCS
- Each answer produces a **live FeedbackCard** with score, model answer, and concept breakdown

### Live Readiness Score
- Baseline calculated from experience vs. days to interview
- Animates upward as mock questions are answered correctly
- **Agent-controlled** — the AI decides when and how much to increment based on answer quality

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                    React Frontend                    │
│  ChatPage → DynamicRenderer → ComponentRegistry     │
│  Zustand Store · Framer Motion · SSE Consumer       │
└──────────────────────┬──────────────────────────────┘
                       │ HTTP + SSE
┌──────────────────────▼──────────────────────────────┐
│                  Express API (Node.js)               │
│  Auth · Chat · Interview · Resume · Analytics       │
│  JWT + Google OAuth · Helmet · Rate Limiter         │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│              CareerPilot Agent Core                  │
│                                                      │
│  IntentClassifier → ToolRegistry → Aggregator       │
│                         │                           │
│          ┌──────────────┼──────────────┐            │
│          ▼              ▼              ▼            │
│      ChatAI       InterviewAI    ResumeAI           │
│          │              │              │            │
│          └──────────────┼──────────────┘            │
│                         ▼                           │
│              RAG Memory + Embeddings                │
│           (text-embedding-3-large, MongoDB)         │
│                         │                           │
│                Observability Tracer                 │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│                  Infrastructure                      │
│  MongoDB Atlas · Redis · BullMQ (5 processors)      │
│  Cloudinary · Bull Board · Docker Compose           │
└─────────────────────────────────────────────────────┘
```

---

## Tech Stack

### Backend
| Layer | Technology |
|---|---|
| Runtime | Node.js 20 + TypeScript 5 |
| Framework | Express.js |
| AI / LLM | OpenAI GPT-4o + text-embedding-3-large |
| Database | MongoDB Atlas (Mongoose) |
| Cache / Queue | Redis + BullMQ |
| Auth | JWT + Google OAuth 2.0 |
| File Storage | Cloudinary + Multer |
| Security | Helmet, CORS, Rate Limiter, HttpOnly Cookies |
| Validation | Zod |
| Logging | Winston + Morgan |
| Testing | Jest + Supertest |
| API Docs | Swagger UI (`/api/docs`) |
| Infra | Docker Compose |

### Frontend
| Layer | Technology |
|---|---|
| Framework | React 18 + Vite |
| State Management | Zustand |
| Animations | Framer Motion |
| Styling | Tailwind CSS |
| Streaming | Server-Sent Events (SSE) |
| Charts | Recharts |
| PDF Export | html2pdf.js |

---

## Project Structure

```
careerpilot/
├── backend/
│   ├── src/
│   │   ├── agent/
│   │   │   ├── careerPilotAgent.ts     # Main agent orchestrator
│   │   │   ├── intentClassifier.ts     # Classifies user intent
│   │   │   ├── toolRegistry.ts         # Tool definitions + executor
│   │   │   ├── aggregator.ts           # Result synthesis
│   │   │   └── observability.ts        # Execution tracing
│   │   ├── ai/
│   │   │   ├── chat.ai.ts              # Dynamic UI chat AI
│   │   │   ├── interview.ai.ts         # Adaptive interview AI
│   │   │   └── resumeAnalysis.ai.ts    # Resume parser + analyser
│   │   ├── services/
│   │   │   ├── memory.service.ts       # RAG vector memory
│   │   │   ├── embedding.service.ts    # OpenAI embeddings + caching
│   │   │   ├── interview.service.ts    # Interview session logic
│   │   │   ├── resume.service.ts       # Resume pipeline
│   │   │   └── sse.service.ts          # Server-Sent Events
│   │   ├── queues/
│   │   │   └── processors/             # 5 BullMQ job processors
│   │   ├── controllers/                # Route handlers
│   │   ├── models/                     # Mongoose schemas
│   │   ├── middleware/                 # Auth, rate limit, error handling
│   │   └── config/                     # App, DB, Redis, BullMQ config
│   └── tests/                          # Unit + integration tests
│
└── frontend/
    └── src/
        ├── components/
        │   ├── renderer/
        │   │   └── DynamicRenderer.tsx # Core JSON → component router
        │   ├── ui/                     # 9 dynamic UI components
        │   └── chat/                   # Chat interface components
        ├── registry/
        │   └── componentRegistry.tsx   # Whitelisted component map
        ├── hooks/
        │   └── useChat.ts              # SSE + message state hook
        ├── store/
        │   └── chatStore.ts            # Zustand global state
        └── pages/
            ├── ChatPage.tsx            # Main chat interface
            └── DashboardPage.tsx       # Session overview
```

---

## Getting Started

### Prerequisites
- Node.js 20+
- MongoDB Atlas account
- Redis (local or Upstash)
- OpenAI API key

### 1. Clone the repository
```bash
git clone https://github.com/your-username/careerpilot.git
cd careerpilot
```

### 2. Backend setup
```bash
cd backend
npm install
cp .env.example .env
# Fill in your credentials in .env
npm run dev
```

### 3. Frontend setup
```bash
cd frontend
npm install
cp .env.example .env
# Set VITE_API_URL=http://localhost:5000
npm run dev
```

### 4. Run with Docker (recommended)
```bash
docker-compose up --build
```

### Environment Variables
```env
# Backend .env.example
NODE_ENV=development
PORT=5000
MONGODB_URI=your_mongodb_atlas_uri
REDIS_URL=your_redis_url
OPENAI_API_KEY=your_openai_key
JWT_SECRET=your_jwt_secret
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
CLOUDINARY_CLOUD_NAME=your_cloudinary_name
CLOUDINARY_API_KEY=your_cloudinary_key
CLOUDINARY_API_SECRET=your_cloudinary_secret
```

---

## 🔌 API Reference

Full Swagger documentation available at `/api/docs` when running locally.

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/auth/register` | Register new user |
| `POST` | `/api/auth/login` | Login + get JWT |
| `GET` | `/api/auth/google` | Google OAuth |
| `POST` | `/api/chat/message` | Send message to agent |
| `GET` | `/api/chat/stream/:sessionId` | SSE stream for live typing |
| `POST` | `/api/interview/start` | Start adaptive mock interview |
| `POST` | `/api/interview/answer` | Submit + evaluate answer |
| `POST` | `/api/resume/upload` | Upload + analyse resume |
| `GET` | `/api/session/:id` | Get full session state |
| `GET` | `/api/analytics/readiness` | Get readiness score history |

---

## 🧪 Testing

```bash
# Run all tests
cd backend && npm test

# With coverage report
npm run test:coverage

# Watch mode
npm run test:watch
```

Test coverage includes: auth service, chat service, interview service, resume service, readiness scoring, JWT utilities.

---


## Roadmap

- [ ] Voice answer mode (Web Speech API)
- [ ] Live confidence meter while typing
- [ ] Agent execution trace panel (show reasoning steps)
- [ ] Peer benchmark — percentile ranking vs other users
- [ ] Shareable session "battle card" (PNG export)
- [ ] Memory insight cards ("Last time you struggled with this...")
- [ ] Company-specific question tagging ("Asked in TCS NQT 2024")
- [ ] Mobile app (React Native)

---

## Team

Built for **HackIndia Spark 9 — South India Region**
Hosted at St. Joseph's Institute of Technology, Chennai
Track: **Stateful AI Agents with Dynamic UI (Track 04)**

---

