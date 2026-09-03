# Hossaty — Grounded AI for Tutoring That Learns From Evidence

![Hossaty](https://img.shields.io/badge/Hossaty-Grounded_AI_Education-5B7FA3?style=flat-square) ![Curriculum](https://img.shields.io/badge/AI-Curriculum_Grounded-5D8C83?style=flat-square) ![Learning Loop](https://img.shields.io/badge/Learning-Closed_Loop-7A6F9B?style=flat-square) ![Bilingual](https://img.shields.io/badge/EN%20%2F%20AR-RTL-6B7280?style=flat-square)

> **What if an AI tutor were not allowed to teach from whatever it happens to know — only from what the student's actual teacher chose to teach?**

Hossaty is an AI-native platform for private tutors and small tutoring centers. It combines the operational system a tutor needs — schedules, sessions, payments, performance ratings, groups and administration — with a deliberately bounded AI layer built around a different idea of educational intelligence:

**the AI should amplify the teacher's curriculum and judgment, not replace them.**

A student can photograph handwritten homework. The system returns a structured per-mistake review and can point back to relevant textbook pages. Those mistakes become persistent weakness signals. The next tutoring conversation is then assembled from the teacher's uploaded curriculum, the student's measured level, known weaknesses, and recent conversation context.

That creates a closed learning loop:

```text
Teacher-owned curriculum
        │
        ▼
Curriculum-grounded AI ───────────────┐
        │                             │
        ▼                             │
Student learning / homework           │
        │                             │
        ▼                             │
Structured grading + tutor ratings    │
        │                             │
        ▼                             │
Weakness detection                    │
        │                             │
        └──────► future personalization
```

The platform is designed so a student's difficulty is not simply graded once and forgotten.

---

## Why Hossaty matters

Independent tutors often run a surprisingly complex educational operation through disconnected tools: messaging for schedules, spreadsheets or memory for payments, ad-hoc notes for performance, and personal recollection for which student is struggling with which topic.

The real cost is not administrative inconvenience. It is **loss of educational continuity**.

A weak topic can appear in homework, appear again in a session, and still remain invisible as a recurring pattern until an exam exposes it.

Hossaty connects those signals into one system.

| Educational problem | Hossaty's design response |
|---|---|
| AI can confidently teach outside the assigned curriculum | Ground responses in the teacher's uploaded material and narrow behavior when curriculum evidence is insufficient |
| Personalization can become an unverifiable model guess | Derive level from recorded performance and weaknesses from observed events |
| Homework feedback is often a dead end | Convert grading into structured evidence that feeds future tutoring context |
| Tutors must remember every student's recurring difficulty | Persist weakness detections across sessions |
| Operational fragmentation reduces tutor capacity | Integrate scheduling, sessions, ratings, payments, groups and administration |
| AI costs can become open-ended | Meter AI usage per teacher and check quota before AI-consuming actions |
| Parent/tutor payment records can conflict | Require confirmation by the counterparty rather than the payment initiator |

---

## The engineering thesis

Hossaty is not primarily a chatbot attached to an education dashboard.

Its core engineering thesis is:

> **Trustworthy educational AI should be grounded in an explicit authority, personalized from measured evidence, and continuously updated by structured learning events.**

That principle produces four important constraints.

### 1. Authority is bounded

The teacher's material is the instructional source of truth for curriculum-grounded interactions. The product is intentionally designed not to treat generic model knowledge as equivalent to the assigned curriculum.

### 2. Personalization is measured, not guessed

The student's level comes from recorded ratings. Weaknesses come from low ratings and flagged homework errors. The model is not asked to silently infer an educational profile from conversational style.

### 3. Assessment changes future assistance

A grading event is not merely a result screen. Structured mistakes feed weakness detection; weakness detections feed future tutoring context.

### 4. AI is operationally bounded

Usage is quota-controlled per teacher. Model calls are treated as a cost-bearing system resource, not an unlimited feature.

---

# The four-signal tutoring context

Before a tutoring response, `chatBuilder` combines four independent signals:

```text
                 Student question
                       │
        ┌──────────────┼──────────────┬──────────────┐
        ▼              ▼              ▼              ▼
  CURRICULUM       MEASURED        KNOWN        CONVERSATION
  GROUNDING          LEVEL        WEAKNESSES       MEMORY
        │              │              │              │
        └──────────────┴──────┬───────┴──────────────┘
                              ▼
                     Context assembly
                              │
                              ▼
                    Bounded AI response
```

### Curriculum grounding

The system retrieves the top three similar chunks from material uploaded by the teacher, scoped to the relevant subject through PostgreSQL + pgvector similarity search.

### Measured level

Recorded rating averages map the student to a broad instructional level such as beginner, intermediate or advanced.

### Known weaknesses

Active `weakness_detections` records are injected explicitly. These are persisted observations, not ephemeral model guesses.

### Conversation memory

Recent turns preserve local conversational continuity without making the full chat history the sole source of personalization.

This combination means the assistant can answer the *same curriculum question* differently depending on what the student has actually demonstrated.

---

# The closed-loop learning engine

The most important system behavior is not any single AI call. It is the loop between **teaching, assessment, evidence and future teaching**.

## Step 1 — a student submits handwritten work

A vision-capable model evaluates the image and returns structured data rather than only prose:

```json
{
  "total_problems": 6,
  "correct": 4,
  "score": 75,
  "errors": [
    {
      "problem_number": 3,
      "issue": "Forgot the common denominator",
      "correction": "The answer is 3/4, not 2/3"
    }
  ],
  "summary": "Solved 4 of 6 correctly",
  "suggested_pages": [45]
}
```

The schema matters. A paragraph saying “needs work on fractions” is useful to a human, but difficult to use reliably as machine-readable state. Structured errors can become inputs to the next stage.

The teacher remains in the loop: both teacher and student can see the review, and the teacher can add human feedback or request re-evaluation.

## Step 2 — weakness detection converts events into memory

A rating below 6/10 or a grading event containing flagged mistakes triggers `weaknessDetector`.

The system can create a new weakness record or increment the occurrence count of an existing one.

```text
low rating ───────────┐
                      ├──► weaknessDetector ──► persistent weakness state
homework mistake ─────┘                              │
                                                    ▼
                                         future tutoring context
```

This is the educational flywheel: **the system remembers recurring difficulty without requiring the tutor to manually re-enter the pattern after every session.**

## Step 3 — the next conversation starts with better evidence

The weakness state becomes one of the four signals used in future tutoring responses.

Assessment therefore changes subsequent assistance.

That is the difference between an AI feature and an AI-enabled learning system.

---

# Curriculum ingestion: the teacher controls the knowledge boundary

A teacher uploads a PDF textbook or learning document.

Processing occurs asynchronously:

```text
PDF upload
   │
   ▼
BullMQ document-processing job
   │
   ├──► text extraction with pdf-parse
   │
   ├──► paragraph-aware overlapping chunks
   │      1,000 chars / 200 overlap
   │
   ├──► text-embedding-3-small
   │      1536 dimensions
   │
   └──► PostgreSQL + pgvector
          HNSW cosine index
```

A controlled live-deployment example documented for the project produced **69 indexed chunks from a six-page test PDF**.

Just as important is the failure behavior. If a scanned document contains no extractable text, the current pipeline records the document as failed with an explicit `OCR not supported` reason rather than pretending an empty index is usable.

That distinction matters in education: **“file uploaded” and “curriculum successfully available to the AI” are not the same claim.**

---

# Human-centered operational system

The AI layer sits inside a broader tutoring platform rather than replacing the educational relationship.

## Tutor workflow

Tutors can manage:

- student enrollment;
- recurring weekly schedules;
- automatically generated sessions across a rolling 30-day horizon;
- session states including both student and teacher no-shows;
- six-dimensional performance ratings;
- payment records requiring counterparty confirmation;
- study groups and membership approval;
- curriculum documents;
- homework review;
- student weakness history;
- AI usage quotas.

## Six dimensions instead of one vague score

Performance can be recorded across:

**general · homework · participation · exams · understanding · behavior**

This creates richer evidence for both humans and the personalization layer than a single aggregate “good/bad student” judgment.

## Payment trust by structure

Either party may record a payment, but the party that created it cannot confirm its own record. Confirmation or rejection belongs to the other party.

```text
Tutor records payment ─────► Student confirms / rejects
Student records payment ───► Tutor confirms / rejects
```

The system reduces a trust problem by constraining the workflow instead of adding a larger dispute-management feature afterward.

## Groups and motivation

Study groups include capacity-controlled join workflows, group schedules, optional leaderboards and achievement mechanics. Seven achievement definitions are documented as seeded in the current system.

Gamification is positioned as an engagement layer around learning activity, not as a replacement for teacher assessment.

---

# Why education organizations should care

Although Hossaty began around independent tutors and small tutoring centers, the architecture demonstrates several patterns relevant to organizations operating education, tutoring, catch-up, after-school or learning-support programs.

### 1. Educational continuity at larger caseloads

When educators manage many learners, important signals become easy to lose. Persisted weakness records can make recurring difficulty visible instead of relying entirely on individual memory.

### 2. Curriculum fidelity

An organization can define which material should govern assistance rather than treating an LLM's general training corpus as the instructional authority.

### 3. Evidence-driven personalization

The system separates *measured learner evidence* from *model inference*. This is especially valuable where organizations need to explain why a learner received a particular level of support.

### 4. Teacher-in-the-loop AI

The AI performs repetitive work — retrieval, first-pass grading, pattern detection — while teachers retain the instructional role and can review grading output.

### 5. Bilingual access

The interface is English-first with full Arabic RTL support. The layout uses direction-aware behavior rather than treating Arabic as translated strings inserted into an English-only interface.

### 6. Operational accountability

Scheduling, attendance/no-show states, ratings, payment confirmation, activity logs and administration sit alongside the learning layer, allowing educational delivery and program operations to share one system.

> Hossaty should not be interpreted as a completed institutional LMS or as proof of learning-outcome improvement at organizational scale. It is evidence of an implemented platform architecture that can support those workflows; institutional pilots and outcome studies would be separate validation work.

---

# Architecture

```text
┌───────────────────────────────────────────────────────────────┐
│                    Next.js 14 Frontend                       │
│      Tutor · Student · Admin · EN/AR · RTL · KaTeX          │
└─────────────────────────────┬─────────────────────────────────┘
                              │ JWT + API client
                              ▼
┌───────────────────────────────────────────────────────────────┐
│                       Fastify 5 API                           │
│ Auth · Scheduling · Sessions · Ratings · Payments · Groups   │
│ Documents · Homework · Chat · Memory · Admin                 │
└──────────────┬──────────────────────┬─────────────────────────┘
               │                      │
               ▼                      ▼
┌────────────────────────┐   ┌──────────────────────────────────┐
│ PostgreSQL 16          │   │ Redis + BullMQ                  │
│ + pgvector             │   │ document processing            │
│ 28-table model         │   │ homework review                │
│ curriculum vectors     │   │ session generation             │
│ learning state         │   │ quota reset                    │
└────────────────────────┘   └─────────────────┬────────────────┘
                                              │
                                              ▼
                                  ┌──────────────────────────┐
                                  │ AI Provider Abstraction  │
                                  │ chat · embeddings ·      │
                                  │ vision                   │
                                  └──────────────────────────┘
```

The documented API surface contains **50+ endpoints** across authentication, tutoring core, payments, groups, documents/RAG, homework vision grading, chat, memories and administration.

---

# AI provider abstraction

Chat, embedding and vision calls route through a three-method provider interface.

OpenAI is the documented implementation. The application architecture is intended to allow another provider to implement the same contract without rewriting the route layer.

This is important for an education platform because model availability, pricing, privacy requirements and organizational procurement constraints can change independently of product workflows.

---

# Cost control is part of product safety

Every AI-consuming action is not merely a UX event; it is a variable-cost operation.

Hossaty therefore meters usage per teacher with a monthly quota checked before chat and homework-grading actions. A zero-quota/free tier is hard-blocked from those AI actions rather than silently accumulating overage.

A scheduled job resets quotas monthly, and administrative views expose usage.

The engineering principle is simple:

> **An AI feature whose maximum cost exposure cannot be reasoned about is not operationally complete.**

---

# Security and reliability posture

The project documentation records the following controls:

- JWT authentication with role claims;
- bcrypt password hashing;
- activation tokens with expiry and single-use locking using `FOR UPDATE`;
- password-policy enforcement;
- global and endpoint-specific rate limits;
- Helmet HTTP hardening;
- strict CORS allowlisting;
- parameterized database queries;
- email, phone and UUID validation;
- container ports bound to localhost behind TLS termination;
- dependency upgrades that cleared documented Dependabot alerts, including a critical JWT-bypass advisory affecting an earlier dependency state;
- daily automated backups with documented 30-day retention;
- per-teacher AI quotas as cost controls.

See [`docs/SECURITY_AND_PRIVACY.md`](docs/SECURITY_AND_PRIVACY.md) for boundaries and caveats.

---

# Evidence snapshot

| Claim | Evidence status | Public evidence |
|---|---|---|
| Full tutoring platform exists | **IMPLEMENTED / source-documented live** | source project documentation + live URL |
| Curriculum retrieval uses pgvector | **IMPLEMENTED** | documented architecture |
| Four-signal tutoring context | **IMPLEMENTED** | documented `chatBuilder` behavior |
| Vision homework review returns structured mistakes | **IMPLEMENTED** | documented grading schema |
| Weakness events feed future personalization | **IMPLEMENTED** | documented `weaknessDetector` pipeline |
| Six-page PDF produced 69 chunks | **VERIFIED IN A CONTROLLED RUN** | documented live-deployment test |
| 28-table data model | **IMPLEMENTED** | source architecture inventory |
| 50+ API endpoints | **IMPLEMENTED** | source API inventory / Swagger reference |
| EN/AR + RTL interface | **IMPLEMENTED** | source frontend documentation |
| WhatsApp notification delivery | **SCAFFOLDED** | Twilio wiring/templates not claimed complete |
| Payment gateway subscriptions | **PLANNED** | roadmap |
| Mobile application | **PLANNED** | roadmap |
| Improved student outcomes at scale | **NOT YET VALIDATED** | requires longitudinal educational evaluation |

---

# Deliberate trade-offs

### Hard grounding over maximum fluency

A narrower answer is accepted when the alternative is an authoritative-sounding answer outside the intended curriculum.

### Measured personalization over instant personalization

A new student has a cold-start period because the system waits for evidence instead of inventing a learner profile.

### Structured grading over unconstrained prose

The vision pipeline is asked to produce machine-usable output so assessment can feed later computation.

### Asynchronous ingestion over instant completion

Teachers receive explicit processing states rather than a synchronous upload path that could hide indexing failures.

### Teacher review over autonomous educational authority

Vision grading is a first-pass system output that can be reviewed or re-evaluated; it is not presented here as infallible assessment.

---

# Known limitations

The architecture is intentionally explicit about what remains unresolved:

- learner personalization has a cold-start problem until measured evidence accumulates;
- vision grading depends on image/handwriting quality;
- the documented current grading pipeline does not yet provide a mature confidence-threshold workflow for low-quality images;
- scanned PDFs requiring OCR are not supported by the documented ingestion path;
- payment gateway integration is planned rather than claimed live;
- WhatsApp/SMS notification integration is scaffolded rather than claimed operational;
- organizational-scale learning outcomes, educator workload reduction and retention effects have **not yet been validated through a formal longitudinal study**.

These are product and research boundaries, not details to hide. See [`docs/LIMITATIONS.md`](docs/LIMITATIONS.md).

---

# What generalizes beyond tutoring

Hossaty's technical pattern can be abstracted as:

```text
Authoritative source
        │
        ▼
Hard-grounded retrieval
        │
Measured individual evidence ──► personalization
        │                         │
Graded artifact ──► weakness ─────┘
        │
        ▼
Future bounded assistance
```

The same pattern may be adapted to domains where an assistant must speak from a controlled source and personalize from observable evidence — for example enterprise knowledge support, policy-grounded training or field-service learning.

That does **not** mean Hossaty's education-specific rules can simply be copied into higher-stakes domains. Authority models, document versioning, privacy, safety and validation requirements change materially by domain.

---

# Repository map

- [`docs/CASE_STUDY.md`](docs/CASE_STUDY.md) — problem, educational loop and engineering thesis
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — system topology and data flows
- [`docs/ENGINEERING_DECISIONS.md`](docs/ENGINEERING_DECISIONS.md) — major design decisions and trade-offs
- [`docs/TESTING_AND_VERIFICATION.md`](docs/TESTING_AND_VERIFICATION.md) — what the available evidence supports
- [`docs/SECURITY_AND_PRIVACY.md`](docs/SECURITY_AND_PRIVACY.md) — security controls and educational-data boundaries
- [`docs/LIMITATIONS.md`](docs/LIMITATIONS.md) — known gaps and validation boundaries
- [`evidence/README.md`](evidence/README.md) — evidence classification

---

## Live references

**Product:** https://tutor.agentcraft.info  
**API documentation:** https://api.tutor.agentcraft.info/docs  
**Portfolio:** https://agentcraft.info

---

## Portfolio notice

This public repository is an **Engineering Evidence / Technical Case Study** for Hossaty. It is not a mirror of the private production source code and does not grant permission to operate or commercially reproduce the underlying product.

Built by **Ayman Alsaid** as part of the AgentCraft portfolio.