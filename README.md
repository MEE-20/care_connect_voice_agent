# CareConnect — AI Voice Receptionist

CareConnect is a production-inspired AI voice receptionist designed for GP practices.

It enables patients to interact with a voice-based receptionist for common administrative tasks such as practice information, appointment availability, appointment booking, prescription-related queries, test-result queries, and human handoff.

The system combines real-time voice interaction, LLM reasoning, RAG, tool calling, safety guardrails, explicit confirmation for consequential actions, and Google Calendar integration.

> **Demo project:** Uses fictional GP-practice data only. No real patient data or clinical decision-making is involved.

---

## Overview

CareConnect is designed around a simple engineering principle:

> **LLM decides; guardrails constrain; tools execute; external systems remain authoritative.**

The LLM is responsible for understanding the user's request and selecting the appropriate action.

Guardrails constrain what the system is allowed to do.

Tools perform deterministic operations such as checking appointment availability or creating a calendar event.

External systems such as Google Calendar remain the source of truth for transactional operations.

---

## Architecture

```text
                    ┌─────────────────────┐
                    │       Caller        │
                    │   Voice / Browser   │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │       Twilio        │
                    │ ConversationRelay   │
                    │                     │
                    │ STT / VAD /         │
                    │ Barge-in / Streaming│
                    └──────────┬──────────┘
                               │
                         Transcript
                               │
                               ▼
                    ┌─────────────────────┐
                    │      FastAPI        │
                    │   WebSocket Layer   │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │     LangGraph       │
                    │ Conversation State  │
                    │ Routing / Workflow  │
                    └──────────┬──────────┘
                               │
                    ┌──────────┴──────────┐
                    │                     │
                    ▼                     ▼
             ┌──────────────┐      ┌──────────────┐
             │    Gemini    │      │  Guardrails  │
             │     LLM      │      │              │
             └──────┬───────┘      └──────┬───────┘
                    │                     │
             ┌──────┴───────┐             │
             │              │             │
             ▼              ▼             ▼
          ┌──────┐      ┌────────┐    Safety /
          │ RAG  │      │ Tools  │    Validation
          └──┬───┘      └───┬────┘
             │              │
             ▼              ▼
       Knowledge Base   Google Calendar
             │
             └──────────┬───────────┐
                        │           │
                        ▼           ▼
                 Output Guardrails  Human Handoff
                        │
                        ▼
                 Response Generation
                        │
                        ▼
                 ElevenLabs TTS
                        │
                        ▼
                      Caller
```

---

## Key Features

### Voice Interaction

- Real-time voice conversation
- Speech-to-text through the managed voice layer
- Voice activity detection
- Interruption / barge-in handling
- Streaming responses
- British English voice configuration

### LLM-powered Receptionist

The receptionist can handle:

- Practice information
- Opening hours
- Available services
- Appointment requests
- Appointment availability
- Appointment cancellation
- Repeat prescription information
- Test-result policy questions
- Accessibility information
- Human receptionist requests

### RAG

A fictional GP-practice knowledge base is used for information retrieval.

RAG is used for:

- Practice policies
- Opening hours
- Services
- Appointment policies
- Prescription policies
- Test-result policies
- Accessibility information
- Safety and escalation policies

Transactional operations are **not** performed through RAG.

> **RAG answers knowledge questions; tools perform actions.**

---

## Appointment Booking

Appointment booking follows a confirmation-first transaction flow.

```text
User requests appointment
        ↓
Check availability
        ↓
Offer available slot
        ↓
User selects slot
        ↓
Summarize booking
        ↓
Explicit confirmation
        ↓
Validate booking request
        ↓
Google Calendar
        ↓
Booking confirmed
```

The system never books an appointment merely because the user says something ambiguous such as:

> "Okay."

Confirmation must be associated with the current pending booking transaction.

---

## Safety & Guardrails

Guardrails are first-class components of the architecture.

### Input Guardrails

- Scope validation
- Urgency detection
- Emergency detection
- Transcript quality checks
- Prompt-injection-style input detection
- Unsafe or abusive input detection

### Safety Guardrails

CareConnect does **not**:

- Diagnose medical conditions
- Prescribe medication
- Interpret clinical results
- Recommend treatment
- Make clinical decisions

When a request falls outside the system's safe administrative scope, CareConnect escalates rather than improvising.

### Tool Guardrails

Before executing a consequential tool:

```text
LLM requests tool
        ↓
Validate arguments
        ↓
Validate context
        ↓
Check authorization
        ↓
Check explicit confirmation
        ↓
Check business rules
        ↓
Execute tool
```

Unauthorized booking execution should be:

> **Zero successful executions**

### Output Guardrails

Responses are checked for:

- Grounding
- Safety
- Scope
- Hallucination
- Privacy

---

## Tools

The initial tool set includes:

| Tool | Purpose |
|------|---------|
| `get_practice_information()` | Practice information lookup |
| `search_knowledge_base()` | Knowledge retrieval |
| `check_appointment_availability()` | Find available appointment slots |
| `book_appointment()` | Create confirmed appointment |
| `cancel_appointment()` | Cancel an appointment |
| `transfer_to_human()` | Escalate to a receptionist |
| `end_call()` | End the interaction |
| `create_call_summary()` | Generate structured post-call summary |

The LLM does not directly manipulate external systems.

---

## Knowledge Base

The project uses fictional GP-practice content.

```text
knowledge_base/
├── practice/
│   ├── practice_information.md
│   ├── opening_hours.md
│   ├── services.md
│   └── contact_information.md
│
├── appointments/
│   ├── appointment_policy.md
│   ├── cancellation_policy.md
│   └── appointment_types.md
│
├── prescriptions/
│   └── repeat_prescription_policy.md
│
├── test_results/
│   └── test_result_policy.md
│
├── accessibility/
│   └── accessibility_policy.md
│
└── safety/
    ├── urgent_care_policy.md
    ├── emergency_policy.md
    └── human_handoff_policy.md
```

No real patient information is used.

---

## Technology Stack

| Layer | Technology |
|------|------------|
| Language | Python |
| Backend | FastAPI |
| Voice | Twilio ConversationRelay |
| STT / Turn Detection | Deepgram via Twilio |
| TTS | ElevenLabs via ConversationRelay |
| LLM | Google Gemini |
| Agent Orchestration | LangGraph |
| RAG | FAISS |
| Embeddings | Gemini embeddings |
| Tools | Python function calling |
| Calendar | Google Calendar API |
| State | LangGraph state + session store |
| Local Database | SQLite / mock data |
| Development Tunnel | ngrok |
| Testing | Pytest |
| Observability | Structured logs + metrics |

---

## Project Structure

```text
careconnect/
│
├── app/
│   ├── main.py
│   ├── config.py
│   ├── api/
│   ├── voice/
│   ├── agent/
│   ├── rag/
│   ├── tools/
│   ├── guardrails/
│   ├── calendar/
│   └── observability/
│
├── knowledge_base/
│   ├── practice/
│   ├── appointments/
│   ├── prescriptions/
│   ├── test_results/
│   ├── accessibility/
│   └── safety/
│
├── tests/
│
├── .env.example
├── .gitignore
├── requirements.txt
└── README.md
```

The structure will evolve as individual features are implemented.

---

## Conversation State

CareConnect maintains structured conversation state including:

```python
{
    "session_id": "...",
    "messages": [],
    "intent": None,
    "urgency": None,
    "retrieved_documents": [],
    "tool_calls": [],
    "appointment_context": {},
    "handoff_required": False,
    "interruption_count": 0,
    "response_latency_ms": None,
    "final_response": None
}
```

Appointment transactions maintain separate context:

```python
{
    "requested_slot": "...",
    "appointment_type": "...",
    "patient_name": "...",
    "confirmation_status": "pending"
}
```

---

## Observability

The system records structured information for each conversation turn.

Example:

```json
{
  "session_id": "abc123",
  "turn_id": 7,
  "intent": "appointment_request",
  "stt_latency_ms": 240,
  "llm_latency_ms": 680,
  "tool_latency_ms": 120,
  "tts_first_audio_latency_ms": 310,
  "total_response_latency_ms": 1350,
  "rag_used": true,
  "retrieved_chunks": 3,
  "tool_called": "check_appointment_availability",
  "tool_success": true,
  "interrupted": false,
  "safety_status": "safe"
}
```

---

## Evaluation

The system will be evaluated at the **agent/system level**, not only at the LLM level.

The evaluation suite will contain approximately 30 scenarios covering:

- Practice information
- Appointment requests
- Cancellation
- Prescription requests
- Test-result requests
- Human handoff
- Emergency scenarios
- Urgent scenarios
- Interruptions
- Backchannel speech
- Booking confirmation
- Booking cancellation
- Clinical-boundary violations
- Prompt injection
- Unauthorized tool execution

Example:

```text
Input:
"I need an appointment tomorrow morning."

Expected:
intent = appointment_request
tool = check_appointment_availability
safety = safe
```

---

## Key Metrics

### Voice

- Time to first audio
- Turn latency
- Customer interruption rate
- Agent interruption rate
- Dead-air duration

### AI

- Intent accuracy
- Tool selection accuracy
- Tool success rate
- RAG relevance
- Grounded response rate

### Safety

- Safety escalation recall
- False escalation rate
- Guardrail trigger rate
- Input-block rate
- Output-block rate
- Tool-guardrail rejection rate

### Operations

- Call duration
- STT/LLM/TTS errors
- Tool errors
- Handoff rate
- Task completion rate
- Calendar conflicts

### Cost

- LLM usage
- Voice usage
- RAG retrieval count
- Tool-call count
- Estimated cost per call
- Estimated cost per completed interaction

---

## Development Approach

The project is being implemented incrementally.

Each feature follows:

```text
Requirement
    ↓
Implementation
    ↓
Unit / Integration Test
    ↓
Manual Voice Test
    ↓
Troubleshooting
    ↓
Freeze Feature
    ↓
Next Feature
```

No major feature is considered complete until it has been tested.

---

## Development Roadmap

### Phase 1 — Voice Foundation

- [ ] Development environment
- [ ] FastAPI application
- [ ] Twilio integration
- [ ] ConversationRelay WebSocket
- [ ] Browser voice demo
- [ ] Voice interruption handling

### Phase 2 — LLM

- [ ] Gemini integration
- [ ] Receptionist system prompt
- [ ] Conversation state
- [ ] Basic intent handling

### Phase 3 — Agent Orchestration

- [ ] LangGraph workflow
- [ ] Routing
- [ ] State management

### Phase 4 — RAG

- [ ] Knowledge base
- [ ] Document ingestion
- [ ] Embeddings
- [ ] FAISS retrieval
- [ ] Grounded responses

### Phase 5 — Tools

- [ ] Appointment availability
- [ ] Google Calendar
- [ ] Appointment booking
- [ ] Appointment cancellation
- [ ] Explicit booking confirmation

### Phase 6 — Safety

- [ ] Input guardrails
- [ ] Safety gateway
- [ ] Tool guardrails
- [ ] Output guardrails
- [ ] Human handoff

### Phase 7 — Observability

- [ ] Structured logging
- [ ] Latency metrics
- [ ] Tool metrics
- [ ] RAG metrics
- [ ] Safety metrics
- [ ] Cost metrics

### Phase 8 — Evaluation

- [ ] Evaluation dataset
- [ ] Automated evaluation
- [ ] Voice regression testing
- [ ] Safety regression testing

### Phase 9 — Deployment

- [ ] Production configuration
- [ ] HTTPS / WSS
- [ ] Secret management
- [ ] Deployment
- [ ] Final demo regression

---

## Design Principles

### 1. LLM decides; guardrails constrain; tools execute.

The LLM interprets natural language, while deterministic components enforce safety and business rules.

### 2. RAG is not a transaction system.

RAG provides grounded information.

Tools perform actions.

### 3. External systems remain authoritative.

Google Calendar determines actual appointment availability and booking state.

### 4. Consequential actions require confirmation.

The system requires explicit user confirmation before booking an appointment.

### 5. Safety takes precedence over completion.

When the system cannot safely handle a request, it escalates instead of guessing.

### 6. The system is evaluated as an agent.

Evaluation covers the complete workflow:

```text
Voice → STT → Agent → RAG/Tools → Guardrails → Response → TTS
```

rather than evaluating the LLM in isolation.

---

## Security & Privacy

This project is a demonstration system.

- No real patient data
- No production NHS data
- Fictional GP-practice information only
- Secrets stored in environment variables
- API keys never committed to Git
- External systems accessed through controlled tools
- Consequential actions protected by confirmation and validation

---

## Status

🚧 **Under active development**

The project is being built incrementally as a production-inspired voice AI demonstration.

---

## License

This project is intended for educational, portfolio, and demonstration purposes.
