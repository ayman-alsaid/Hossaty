# Hossaty Engineering Decisions

## ADR-01 — Curriculum grounding is an authority boundary

**Decision:** use teacher-owned uploaded material as the grounding source for curriculum-oriented AI assistance.

**Rejected alternative:** treat retrieved curriculum as a preference while freely falling back to generic model knowledge.

**Reason:** a fluent answer from a different method or curriculum can be educationally misleading while remaining invisible to the learner.

**Trade-off:** the assistant may be narrower and less conversationally expansive.

---

## ADR-02 — Personalization comes from measured evidence

**Decision:** derive level from recorded ratings and weaknesses from observed low ratings / grading errors.

**Rejected alternative:** ask the model to infer ability directly from conversational style.

**Reason:** recorded evidence is inspectable and correctable; a transient model inference is not.

**Trade-off:** new learners experience a cold start before enough evidence accumulates.

---

## ADR-03 — Grading output is structured

**Decision:** require a schema containing totals, score, error objects, summary and suggested pages.

**Reason:** machine-readable mistakes can feed weakness detection and render consistently across student and teacher views.

**Trade-off:** model output must be validated against a stricter shape.

---

## ADR-04 — Assessment must feed future support

**Decision:** convert low ratings and flagged homework mistakes into persistent weakness records.

**Reason:** otherwise grading remains a dead-end event and personalization depends on human memory.

**Trade-off:** topic extraction and weakness lifecycle become additional state that must be managed carefully.

---

## ADR-05 — Document processing is asynchronous and stateful

**Decision:** process PDFs through BullMQ with explicit processing/failure states.

**Rejected alternative:** block the upload request until indexing finishes or silently accept an empty index.

**Reason:** large documents should not block the user, and indexing failure must remain visible.

---

## ADR-06 — AI provider is abstracted behind three capabilities

**Decision:** isolate chat, embedding and vision behind a provider interface.

**Reason:** model provider, pricing and procurement requirements can change without changing education workflows.

---

## ADR-07 — AI usage is quota-gated before execution

**Decision:** check a teacher's monthly AI quota before chat and grading calls; hard-block zero quota.

**Reason:** variable AI cost must have a bounded operational exposure.

**Trade-off:** this is less flexible than an overage/billing system but easier to reason about safely.

---

## ADR-08 — Payment confirmation requires the counterparty

**Decision:** the payment creator cannot confirm their own payment record.

**Reason:** encode mutual acknowledgment structurally rather than relying on self-reported final state.

---

## ADR-09 — Track teacher and student no-shows separately

**Decision:** represent `no_show_student` and `no_show_teacher` as distinct session outcomes.

**Reason:** accountability should not be asymmetric; operational evidence should describe what actually happened.

---

## ADR-10 — Teacher remains in the grading loop

**Decision:** AI grading is reviewable and can be supplemented or re-evaluated.

**Reason:** a vision model's structured output is useful first-pass evidence, not infallible educational judgment.

---

## ADR-11 — Arabic is a layout concern, not only a translation concern

**Decision:** support full RTL behavior and direction-aware interface layout.

**Reason:** bilingual education software is not genuinely bilingual when translated text is forced through LTR interaction patterns.

---

## Design pattern

Across these decisions, the recurring principle is:

> **Prefer explicit state, bounded authority and inspectable evidence over invisible inference.**