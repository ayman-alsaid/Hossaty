# Hossaty — Limitations & Validation Boundaries

Strong educational technology should make its uncertainty visible.

## 1. Personalization has a cold start

Hossaty intentionally prefers measured evidence to inferred learner profiling. A new student therefore begins with limited personalization until ratings, homework and weakness signals accumulate.

This is a deliberate trust trade-off.

## 2. Vision grading is not infallible

Handwriting quality, lighting, camera angle, subject complexity and ambiguous work can affect vision-model interpretation.

The current source documents teacher review/re-evaluation pathways, but does not establish universal grading accuracy or a mature confidence-threshold workflow for poor-quality submissions.

## 3. OCR is not supported in the documented ingestion path

A scanned PDF without extractable text is explicitly failed rather than silently indexed. OCR would require a separate ingestion capability and validation path.

## 4. Grounding does not automatically equal correctness

Retrieving from the intended textbook reduces one class of failure — answering from the wrong source — but does not guarantee that every retrieved passage is the best passage or that every generated interpretation is correct.

## 5. Weakness detection is an operational signal, not a diagnosis

A persisted weakness record represents recurring observed difficulty in the system's learning workflow. It should not be interpreted as a clinical, cognitive or psychological diagnosis.

## 6. Organizational outcomes are not yet validated

The current evidence does not demonstrate at institutional scale:

- higher grades;
- improved retention;
- lower dropout;
- reduced tutor workload;
- increased tutor capacity;
- lower operational cost;
- improved educational equity.

These require pilots and study design.

## 7. Payments are not the same as payment processing

The platform implements a bidirectional payment-record confirmation model. The roadmap separately identifies a payment gateway integration as planned. The ledger workflow must not be represented as completed gateway processing.

## 8. Notifications remain scaffolded

Twilio WhatsApp/SMS support is documented as scaffolded, with activation/templates not claimed complete.

## 9. Mobile is planned

The documented current interface is web-based. A camera-first native mobile app is a roadmap item.

## 10. Institutional privacy/compliance requires additional work

The security architecture contains meaningful controls, but this repository does not claim formal education/privacy certifications or jurisdiction-specific compliance.

## 11. Generalization requires domain redesign

The grounding/personalization pattern may transfer to other domains, but education-specific thresholds, authority assumptions and workflows should not be copied blindly into compliance, healthcare or other higher-stakes contexts.

## Boundary rule

**Use the strongest claim supported by evidence — never the strongest claim that would make the product sound more impressive.**