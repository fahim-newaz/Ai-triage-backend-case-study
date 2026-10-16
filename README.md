# Siza AI Triage: patient triage backend (internship project)

A rule-based patient triage and doctor-assistant backend, built solo during a supervised, CDU-partnered internship with Siza Telehealth (2026).

This repository contains documentation only. It has no source code, credentials or patient data. All data used in the project was synthetic.

## Honest status

- Works locally. Not deployed and not validated on staging.
- **No AI model.** Triage uses draft clinical rules, and summaries are built from a template. A screen label that says "AI-assisted" is wrong (recorded as a defect).
- Notifications, calendar, file storage, OCR and the login code are mocked, and so are the EMR, lab, pharmacy and billing integrations.
- Clinical rules are drafts. Only the "chest pain" rule set is marked Approved. Not for use with real patients.

## What it does

A patient registers and gives consent, logs in with a one-time code (mock), and describes symptoms. The system classifies risk, creates an escalation record for urgent cases, and puts the case in a doctor queue. A doctor reviews and approves the summary, books an appointment, and can see uploaded documents. Every step leaves an audit record.

## Architecture

- **API:** Node.js and Express, with 50+ REST endpoints.
- **Data:** PostgreSQL 16 (19 tables) and Redis 7, run in Docker for local development.
- **Triage engine:** rule-based red-flag matching across five categories (chest pain, stroke, breathing difficulty, severe allergic reaction, suicidal ideation).
- **Integrations (simulated):** FHIR R4 read mappings for patient, appointment, questionnaire response, observation, document reference and clinical summary. Mock EMR, lab, pharmacy and billing adapters with retry, circuit breaker and dead-lettering. Signed, de-duplicated webhooks. Versioned OpenAPI specification.
- **Security:** JWT authentication and role checks on doctor and admin routes, audit logging, consent tracking.

## Testing

| Suite | Result |
|---|---|
| Access-control tests (JWT and role checks, 14 routes) | 28 pass |
| Contract tests (FHIR mappings and adapters) | 13 pass |
| Chest pain regression (19 phrasings) | 19 of 19 pass |
| Negation tests (a negated phrase cannot hide another red flag) | 7 pass |

A scripted demo also covers a notification-failure scenario: three retries, the failure recorded and flagged in the incident register, then recovery (4 of 4 checks).

## What I found and fixed

- **Chest pain phrasing gaps.** Literal keyword matching missed some wordings. I rebuilt detection to match a chest word near a symptom word, with narrow handling of phrases such as "no chest pain", and fixed two safety bugs (a lower rule overwriting a higher one, and a missing pain score lowering High to Medium).
- **Duplicate appointments** were allowed. Found in failure-path testing and reported.
- **Doctor and admin routes had no login or role checks.** Fixed with JWT and role checks on 14 routes and 28 tests. Merged to main with supervisor approval.

## Known limits

- Staff login still uses shared keys, not individual accounts.
- Patient routes and some others are still unprotected.
- Some end-to-end emergency test cases (including stroke, heavy bleeding and anaphylaxis wording) were still failing at handover and are recorded in the defect register.
- Clinical rules need approval by a qualified clinician.
- A defect register and a 30-day backlog were handed over with the project.

## Stack

Node.js, Express, PostgreSQL, Redis, Docker, JWT, FHIR R4, OpenAPI, Node's built-in test runner.

## Author

Fahim Newaz. Master of Information Technology (Cybersecurity), Charles Darwin University.
