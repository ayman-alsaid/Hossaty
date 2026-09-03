# Hossaty — Security & Privacy Posture

Education systems handle identity, learner performance, communications and potentially sensitive behavioral records. Hossaty's documented architecture includes several controls, but this public case study does not claim compliance certifications that have not been evidenced.

## Documented controls

### Authentication and authorization

- JWT authentication with role claims;
- bcrypt password hashing;
- password policy enforcement;
- activation tokens with expiry;
- single-use activation locking with `FOR UPDATE`;
- administrative safeguard preventing an admin from suspending themselves.

### Request protection

- global rate limiting;
- stricter endpoint-specific limits for authentication and AI actions;
- Helmet HTTP hardening;
- strict CORS allowlist;
- parameterized database queries;
- validators for email, phone and UUID inputs.

### Network/deployment posture

The source documents container ports bound to `127.0.0.1`, TLS through Let's Encrypt behind Nginx Proxy Manager, and host firewall controls.

### Supply-chain maintenance

The project documentation records dependency upgrades that cleared Dependabot alerts, including a critical JWT-bypass advisory affecting a prior dependency version.

This is evidence of remediation work, not a permanent security guarantee.

### Data resilience

Daily automated backups are documented for database, files and git bundle with 30-day retention.

### AI cost controls

Per-teacher quota checks bound model usage before AI-consuming operations.

## Privacy-relevant architecture

Hossaty stores educational state such as ratings, homework review, weaknesses, conversations and memories. These data are useful precisely because they can personalize future support, which also means their access and retention deserve explicit governance in any organizational deployment.

This evidence repository does not claim a completed institutional privacy framework, FERPA/GDPR certification, child-data compliance certification, or jurisdiction-specific education compliance.

An organizational deployment should explicitly define:

- learner/guardian consent requirements;
- retention and deletion policies;
- access boundaries for teachers, administrators, learners and parents/guardians;
- model-provider data handling;
- export/correction rights;
- audit-log retention;
- age-appropriate product requirements;
- institutional data-processing agreements.

## AI-specific privacy boundary

The provider abstraction is architecturally useful because model-provider choice may be constrained by organizational privacy/procurement policy. However, provider abstraction alone is not evidence of privacy compliance.

## Principle

> **Educational personalization creates value by remembering learner evidence; trustworthy deployment requires equally deliberate rules about who may see, retain, correct and delete that evidence.**