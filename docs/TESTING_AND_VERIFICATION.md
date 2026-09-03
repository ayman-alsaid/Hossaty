# Hossaty — Testing & Verification

This document separates implementation evidence from educational-impact claims.

## Evidence classification

### Source-documented implementation

The primary project source documents:

- Next.js 14 tutor/student/admin frontend;
- Fastify 5 API;
- PostgreSQL 16 + pgvector;
- Redis + BullMQ workers;
- 28-table data model;
- 50+ API endpoints;
- curriculum ingestion and vector retrieval;
- four-signal tutoring context;
- vision-based homework grading;
- automatic weakness detection;
- per-teacher AI quota checks;
- recurring session generation;
- six-dimensional ratings;
- bidirectional payment confirmation;
- study groups, achievements and leaderboards;
- English/Arabic RTL interface;
- documented live product and Swagger endpoints.

These support **IMPLEMENTED** claims in this evidence repository.

## Controlled ingestion evidence

The source records a live-deployment test in which a **six-page PDF produced 69 indexed chunks**.

Status: **VERIFIED IN A CONTROLLED RUN**.

What it supports:

- the ingestion pipeline produced chunk records for that test document;
- the documented extraction/chunking/indexing path executed on the live deployment.

What it does **not** support:

- universal indexing quality for arbitrary textbooks;
- OCR capability;
- retrieval accuracy across all subjects/languages;
- educational effectiveness of retrieved content.

## Failure-state evidence

The documented ingestion behavior marks a no-extractable-text scan as failed with an `OCR not supported` reason.

This supports the architectural claim that ingestion distinguishes file receipt from usable AI indexing.

## Structured grading evidence

The source documents a structured homework-review schema with totals, score, per-problem errors, correction, summary and suggested textbook pages.

This supports the implementation claim that the grading pipeline is designed to create machine-readable artifacts suitable for downstream weakness processing.

It does **not** establish human-equivalent grading accuracy across handwriting styles, subjects or image conditions.

## Weakness flywheel evidence

The documented `weaknessDetector` reacts to sub-6 ratings and flagged homework errors by creating or incrementing weakness records. Those records are documented as context inputs to subsequent tutoring responses.

This supports the architectural claim:

**assessment event → weakness state → future personalization**.

It does not establish that this personalization improves grades or learning outcomes without longitudinal study.

## Security verification evidence

The source records dependency hardening that cleared Dependabot alerts including a critical JWT-bypass advisory affecting an earlier dependency state.

This supports a historical remediation claim, not a claim that the application can never contain future vulnerabilities.

## Claims intentionally not made

This repository does not claim:

- statistically significant improvement in student outcomes;
- institutional-scale deployment success;
- bias-free AI grading;
- perfect hallucination elimination;
- OCR support;
- production payment-gateway integration;
- operational WhatsApp notification delivery;
- a native mobile application.

## Recommended next validation layer

For education organizations, the next meaningful evidence should come from a controlled pilot measuring outcomes such as:

- educator time spent on grading/administration;
- time from recurring weakness emergence to educator visibility;
- retrieval-grounding failure rate;
- teacher override/re-evaluation rate for AI grading;
- learner engagement and homework completion;
- retention/attendance signals;
- learning outcomes using a pre-defined study design.

Those metrics are recommendations for future evaluation, not current results.