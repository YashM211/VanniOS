
# VaaniOS — Multilingual AI Voice Receptionist

**Reliable, multilingual voice automation for Indian businesses.**

VaaniOS is an AI-powered voice receptionist designed for Indian businesses, supporting **English, Hindi, and Hinglish**. It automates customer interactions by answering business-related queries, managing appointments and restaurant reservations, capturing messages, and arranging callbacks.

Built with a configurable architecture, VaaniOS supports multiple business domains, including clinics and restaurants, without requiring changes to the core conversation engine.

## Key Features

- **Multilingual Conversations:** Supports English, Hindi, and Hinglish for natural interactions with Indian customers.
- **AI-Powered Intent Recognition:** Uses LLMs to understand caller intent, identify required information, and extract structured fields.
- **Deterministic Business Logic:** Enforces application-level rules for identity verification, booking confirmation, and human escalation.
- **Appointment Management:** Handles appointment scheduling, availability checks, and booking modifications.
- **Restaurant Reservations:** Supports table allocation based on party size, reservation verification, and waitlist callbacks.
- **Business-Specific Configuration:** Adapts conversation flows, response templates, and verification rules through independent configuration files.
- **Safety & Human Handoff:** Handles emergency situations, avoids medical advice, and escalates uncertain conversations to human operators.
- **Real-Time Voice Interaction:** Supports browser-based voice conversations using Pipecat and Sarvam AI.
- **Horizontal Scaling:** Uses Redis-backed shared state to support multiple application instances.

## Core Architecture

**The language model suggests. Code decides.**

VaaniOS separates language understanding from business decision-making.

The LLM is responsible for:
- Intent extraction.
- Language identification.
- Entity and field extraction.
- Understanding caller requests.

Application code is responsible for:
- Business rule enforcement.
- Identity verification.
- Appointment and reservation validation.
- Read-back confirmation before bookings.
- Emergency handling and human escalation.
- Consistent, language-specific response generation.

This architecture makes the system more predictable, reliable, and suitable for business-critical workflows.

## System Architecture

```text
              Caller
                 |
                 v
          Next.js Frontend
                 |
          WebRTC / Pipecat
                 |
                 v
          Python Agent Engine
                 |
          Intent Extraction
                 |
                 v
       Conversation State Machine
                 |
                 v
       Deterministic Rule Engine
                 |
          Fastify Backend API
                 |
                 v
           SQLite Database
                 |
                 v
       Booking & Business Logic
                 |
                 v
         Response Generation
                 |
                 v
          Sarvam Bulbul TTS
                 |
                 v
              Caller
```

## Technology Stack

| Component | Technology |
|---|---|
| Frontend | Next.js |
| Backend API | Fastify |
| ORM | Drizzle ORM |
| Database | SQLite |
| Agent Engine | Python |
| Voice Framework | Pipecat |
| Speech-to-Text | Sarvam Saaras |
| Text-to-Speech | Sarvam Bulbul |
| Language Understanding | Sarvam AI |
| Shared State | Redis |
| Schema Validation | TypeScript, Zod |
| Testing | Vitest, Pytest |

## Project Structure

```text
.
├── apps/
│   └── web/                 # Next.js frontend
│
├── services/
│   ├── api/                 # Fastify + Drizzle + SQLite
│   │
│   └── agent/               # Python agent engine
│
├── packages/
│   └── shared/              # Shared schemas and contracts
│
├── clinics/                 # Business configuration files
│
├── docs/                    # Architecture and development docs
│
├── docker-compose.yml
└── package.json
```

## Getting Started

### Prerequisites

- Node.js 22+
- pnpm 12
- [uv](https://docs.astral.sh/uv/) for Python dependency management
- Python 3.12
- Optional: Sarvam API key for real speech and language processing
- Optional: Docker for running Redis locally

### Installation

**1. Clone the repository**

```bash
git clone <repository-url>
cd vaani-os
```

**2. Install Node.js dependencies**

```bash
pnpm install
```

**3. Install Python dependencies**

```bash
cd services/agent
uv sync
cd ../..
```

**4. Configure environment variables**

```bash
cp .env.example .env
```

Add your Sarvam API key:

```env
SARVAM_API_KEY=your_sarvam_api_key
```

The application can also run using its offline configuration without a real Sarvam API key.

**5. Start the application**

```bash
pnpm demo
```

This starts the application services:

| Service | URL |
|---|---|
| Frontend | http://localhost:3000 |
| Backend API | http://localhost:4000 |
| Python Agent | http://localhost:8000 |

## Using VaaniOS

### Voice Receptionist

Open:

http://localhost:3000

You can interact with VaaniOS through:
- Text-based conversations.
- Browser microphone input.
- Real-time voice conversations using Sarvam AI.

Confirmed bookings appear on the mini schedule.

### Business Dashboard

Open:

http://localhost:3000/login

Demo credentials:

```text
Username: smile-dental
Password: demo-password-123
```

The dashboard provides:
- Schedule management.
- Call history and review.
- Manual call labels.
- Business-specific operational visibility.

## Real-Time Voice Pipeline

VaaniOS uses **Pipecat with WebRTC** to enable real-time browser-based voice conversations.

The pipeline integrates:

- **Pipecat:** Audio transport and pipeline orchestration.
- **Sarvam Saaras:** Speech-to-text conversion.
- **Sarvam Bulbul:** Text-to-speech generation.
- **Python Agent Engine:** Conversation processing and deterministic business logic.

### Voice Processing Flow

```text
Caller Speech
     |
     v
WebRTC Audio Stream
     |
     v
Pipecat Pipeline
     |
     v
Sarvam Saaras (STT)
     |
     v
Recognized Transcript
     |
     v
Python Agent Engine
     |
     v
Business Rule Validation
     |
     v
Fixed Language Response
     |
     v
Sarvam Bulbul (TTS)
     |
     v
Generated Speech
```

Pipecat manages WebRTC transport, Voice Activity Detection (VAD), and barge-in handling.

Both text and voice interactions use the same underlying conversation logic, ensuring consistent business rule enforcement.

Per-turn listening, processing, and speaking latency is logged and exposed through the frontend.

## Horizontal Scaling with Redis

VaaniOS supports multi-instance execution through Redis-backed shared state.

Without Redis, the application uses in-memory state, suitable for single-instance development.

With Redis enabled, the following state is shared:

- Live conversation state.
- Reminder execution claims.
- Rate-limiting information.
- Business configuration cache.

Start Redis locally:

```bash
docker compose up -d redis
```

Configure the same `REDIS_URL` across the API, agent, and web services.

## Testing and Quality Checks

### TypeScript

```bash
pnpm lint
pnpm typecheck
pnpm test
```

### Python

```bash
cd services/agent

uv run ruff check src tests
uv run pytest
```

### Docker Validation

```bash
docker build -f Dockerfile.ci -t vaani-ci .
docker run --rm vaani-ci
```

Live voice tests require `SARVAM_API_KEY` and are skipped when the key is unavailable.

### Updating Shared Contracts

When schemas change inside `packages/shared`, regenerate the Python contracts:

```bash
pnpm schemas:export

cd services/agent
uv run python scripts/gen_contracts.py
```

## Documentation

Additional engineering documentation is available under `docs/`.

- `docs/plan-m1.md` – Initial implementation plan.
- `docs/plan-m2.md` – Development milestones.
- `docs/plan-m3.md` – Voice pipeline implementation.
- `docs/plan-m4.md` – Further development plans.
- `docs/plan-redis.md` – Redis and horizontal scaling.
- `docs/decisions.md` – Architecture and engineering decisions.
- `docs/findings.md` – Testing findings and implementation insights.

---

**VaaniOS**

*Building reliable, multilingual voice agents for real-world Indian business workflows.*
