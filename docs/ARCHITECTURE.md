# Hossaty Architecture

## System topology

```text
Next.js 14
Tutor / Student / Admin
EN default + Arabic RTL
        │
        ▼
Fastify 5 API
        │
        ├── Auth / enrollment
        ├── Scheduling / sessions
        ├── Ratings / payments
        ├── Groups / achievements
        ├── Documents / RAG
        ├── Homework / vision grading
        ├── Chat / memories / weaknesses
        └── Admin / usage / licensing
        │
        ├──────────────► PostgreSQL 16 + pgvector
        │                 28-table documented model
        │
        └──────────────► Redis + BullMQ
                          ├ document processing
                          ├ homework review
                          ├ session generation
                          └ quota reset
                                   │
                                   ▼
                         AI provider abstraction
                         chat / embed / vision
```

## Curriculum retrieval flow

```text
Teacher PDF
   │
   ▼
Upload record
   │
   ▼
BullMQ processing job
   │
   ├─ pdf-parse text extraction
   ├─ paragraph-aware chunks
   │    1000 chars / 200 overlap
   ├─ embedding batches
   │    text-embedding-3-small
   │    1536 dimensions
   └─ document_chunks
        pgvector + HNSW cosine search
```

A no-text scanned PDF is documented as an explicit failure (`OCR not supported`) rather than a successful empty index.

## Tutoring-response flow

```text
student message
     │
     ├─ retrieve top-3 curriculum chunks scoped to subject
     ├─ derive measured level from rating average
     ├─ load active weakness detections
     └─ load recent conversation turns
                │
                ▼
           chatBuilder
                │
                ▼
       quota-controlled AI call
                │
                ▼
        persisted conversation
```

The architecture intentionally keeps curriculum, level, weakness and conversational state as distinct inputs.

## Assessment-to-personalization flow

```text
homework image ──► vision review ──► structured errors ──┐
                                                        ├─► weaknessDetector
sub-6 tutor rating ─────────────────────────────────────┘
                                                                  │
                                                                  ▼
                                                       weakness_detections
                                                                  │
                                                                  ▼
                                                         future chat context
```

## Scheduling flow

A fixed weekly template per student generates concrete sessions across a rolling 30-day horizon. Generation occurs when the template is created and through a scheduled worker that maintains the horizon.

Application-level protections include duplicate prevention and overlap checks. The documented lifecycle includes `scheduled`, `completed`, `cancelled`, `no_show_student`, and `no_show_teacher`.

## Data model

The source documents 28 tables spanning identity, tutor/student profiles, subjects, enrollments, schedules, sessions, ratings, payments, groups, competition settings, achievements, documents/chunks, homework, memories, weaknesses, AI conversations/messages, subscriptions, licenses, settings, logs and activation tokens.

The important architectural point is that educational evidence and operational state live in the same system, allowing a rating or homework event to affect future AI context without a separate manual synchronization process.

## Provider boundary

All three model capabilities — chat, embeddings and vision — route through a provider interface. OpenAI is the documented implementation.

The boundary is intended to isolate model-provider choice from route and product logic.

## Internationalization

The frontend is documented as English-default with Arabic RTL support. Direction-aware layout is treated as part of the interface architecture rather than a post-processing translation step.

## Deployment posture

The source documents Docker Compose, Nginx Proxy Manager with TLS, localhost-bound container ports and daily backups. This repository records the architecture as evidence; it does not expose private production configuration or credentials.