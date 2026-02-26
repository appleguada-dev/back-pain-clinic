# Back Pain Clinic — Implementation Plan

## Objective
Translate the current static protocol apps (`marcus/` and `mcgill/`) into a maintainable, clinically safer, data-backed product without disrupting existing assessment flows.

## Guiding Principles
1. **Safety first:** hard-stop pathways for red flags before exercise output.
2. **Continuity of care:** secure cross-device persistence instead of browser-only state.
3. **Shared core:** one assessment engine with protocol configuration (Marcus vs McGill).
4. **Incremental migration:** ship in thin vertical slices to reduce risk.

---

## Phased Roadmap

### Phase 0 — Baseline & Instrumentation (1 week)
**Goal:** understand current usage and failure modes.

- Add structured event logging hooks (local only initially):
  - phase entry/exit
  - question answered
  - test result recorded
  - report exported
  - guide opened
- Add basic runtime health checks:
  - detect offline/online
  - detect external iframe/video failures
- Define baseline KPIs:
  - completion rate
  - drop-off by phase
  - time-to-complete
  - % sessions with red flags

**Deliverables**
- Event taxonomy document
- Analytics adapter interface (no vendor lock-in)

---

### Phase 1 — Shared Front-End Core (1–2 weeks)
**Goal:** remove duplication between `marcus` and `mcgill`.

- Extract common logic into reusable modules:
  - assessment state machine
  - profile/session persistence layer
  - report generation model
  - phase gating and lock hints
- Convert protocol specifics to config JSON:
  - history questions
  - physical tests
  - recommendation rules
  - links/assets
- Introduce minimal build system (Vite + Vue SFC recommended).

**Deliverables**
- `apps/web` single shell with route/selector for protocol
- `packages/assessment-core` shared package

---

### Phase 2 — Backend & Secure Persistence (2 weeks)
**Goal:** move from `localStorage` to authenticated cloud storage.

- Implement auth (clinician + patient roles).
- API for:
  - users
  - assessments
  - sessions
  - red flag events
  - report exports
- Encrypt sensitive records at rest and in transit.
- Keep a local draft cache with conflict-safe sync.

**Deliverables**
- `apps/api` service with migrations
- `POST /assessments`, `PATCH /sessions/:id`, `GET /reports/:id`

---

### Phase 3 — Clinical Safety Workflow (1–2 weeks)
**Goal:** make escalation explicit and enforceable.

- Introduce severity model:
  - `info`, `warning`, `urgent`, `emergency`
- Hard-stop logic for urgent/emergency flags:
  - block exercise recommendations
  - show action panel (call provider/emergency)
- Add clinician review queue for flagged sessions.

**Deliverables**
- Safety rules engine + audit log entries
- Escalation UX and printable emergency summary

---

### Phase 4 — Reporting & Interoperability (1 week)
**Goal:** enable handoff and longitudinal tracking.

- Structured exports: JSON + CSV (FHIR-lite optional later).
- Enhanced report sections:
  - trend snapshot
  - risk factors
  - actions taken
  - recommended next visit window
- Keep print-friendly PDF layout.

**Deliverables**
- Export endpoints + downloadable report bundle

---

### Phase 5 — Reliability, QA, and Release Hardening (ongoing)
**Goal:** predictable production behavior.

- Replace runtime CDN dependencies with pinned local bundles.
- Add automated checks:
  - unit tests (rules + state transitions)
  - e2e flow tests (mobile + desktop)
  - accessibility smoke checks
  - external link validation jobs
- Add feature flags for protocol/rules rollouts.

**Deliverables**
- CI pipeline with quality gates
- staged environments (dev/staging/prod)

---

## Prioritized Backlog (Top 15)

1. Define domain model for Assessment/Session/Result/Flag.
2. Implement shared state machine for phase progression.
3. Externalize Marcus and McGill protocol definitions.
4. Add local analytics event dispatcher.
5. Add red-flag severity classification.
6. Build hard-stop safety gate in UI and rules layer.
7. Scaffold API service and DB schema.
8. Add authenticated session persistence API.
9. Build migration path from existing local profiles.
10. Implement printable + structured report output.
11. Add clinician dashboard list for flagged sessions.
12. Add offline draft sync strategy.
13. Bundle frontend dependencies locally (remove CDN runtime risk).
14. Add test suite for rules and phase transitions.
15. Add release checklist and rollback plan.

---

## First PR Scope (Recommended)

### PR Title
**feat(core): introduce shared assessment engine and protocol configs**

### Why this first
It unlocks all subsequent work (backend, safety rules, reporting) and immediately reduces duplication risk.

### In scope
- Create `assessment-core` module with:
  - phase state machine
  - transition guards
  - score/progress helpers
- Add protocol config files:
  - `protocols/marcus.ts`
  - `protocols/mcgill.ts`
- Wire one existing page to consume new core (feature-flagged fallback to legacy).
- Add unit tests for:
  - valid/invalid transitions
  - locked phase access
  - completion behavior

### Out of scope
- backend APIs
- auth
- clinician dashboard
- full UI rewrite

### Acceptance criteria
- No regression in existing assessment completion flow.
- Phase lock behavior matches current app semantics.
- Test coverage for transition logic added and passing.

---

## Suggested Technical Stack
- **Frontend:** Vue 3 + Vite + TypeScript
- **Backend:** Node.js (Fastify or Nest) + PostgreSQL
- **Auth:** Clerk/Auth0/Supabase Auth (pick one)
- **Infra:** Docker + managed Postgres + object storage
- **Observability:** Sentry + basic product analytics

---

## Risks & Mitigations
- **Risk:** clinical logic regression during refactor.
  - **Mitigation:** snapshot tests over current protocol decisions before migration.
- **Risk:** data privacy concerns.
  - **Mitigation:** role-based access, encryption, audit logs, retention policy.
- **Risk:** rollout disruption for active users.
  - **Mitigation:** dual-write (local + API) for a short transition window.

---

## Definition of Done (Program Level)
- Shared core used by both protocols.
- Persisted sessions available cross-device.
- Red-flag escalation enforced with hard stops.
- Structured reports exportable and printable.
- CI with tests + accessibility + link checks green.
