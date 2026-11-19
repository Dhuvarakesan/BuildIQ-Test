# Architecture Requirements Document (ARD)

> Replace all `<PLACEHOLDER>` values. Every material architecture decision or constraint should have a stable ID that feature specifications can reference.

## Document Metadata

- **Product/Application:** Single-File HTML Todo Prototype
- **Document ID:** ARD-HTML-TODO-PROTOTYPE
- **Version:** 0.1
- **Status:** Draft
- **Architecture Owner:** TBD
- **Contributors:** TBD
- **Last Updated:** 2026-11-19
- **Reference PRD:** `PRD.md`

---

## 1. Architecture Summary

The architecture is a same-day, single-file, local HTML prototype for a simple todo list. The prototype is delivered as a root-level `todo-prototype.html` file that stakeholders open directly in Desktop Chrome. It uses plain semantic HTML, inline CSS, and vanilla JavaScript with in-memory task state only.

The design intentionally avoids backend services, hosting, external dependencies, user accounts, persistence, analytics, formal accessibility validation, and production operations. The key drivers are fast stakeholder review, clear comparison against competitor todo apps, and reliable demonstration of add, blank-validation, complete, delete, and refresh-reset behavior.

## 2. Architecture Drivers

- PRD-FR-001 — Display an empty todo list with a prompt when the prototype first opens.
- PRD-FR-002 — Allow a user to add a non-blank task.
- PRD-FR-003 — Prevent blank task submissions with a simple inline error message.
- PRD-FR-004 — Allow a user to mark a task complete.
- PRD-FR-005 — Delete a task immediately without confirmation.
- PRD-FR-006 — Reset tasks on browser refresh with no persistence.
- PRD-NFR-001 — Implement as one self-contained HTML file with inline CSS and JavaScript and no backend dependency.
- PRD-NFR-002 — Support the core interactions in Desktop Chrome.
- PRD-NFR-003 — Do not require formal accessibility validation for this prototype phase.
- CON-001 — The company context is a small business.
- CON-002 — The prototype must be delivered same-day.
- CON-003 — The prototype must be a single self-contained HTML file with inline CSS and JavaScript.
- CON-004 — The prototype must not use a backend.
- CON-005 — The prototype targets Desktop Chrome only.
- CON-006 — The prototype must use a clean modern layout with simple responsive styling.
- CON-007 — The prototype must not persist tasks after refresh.
- CON-008 — Formal accessibility compliance is not required for this prototype phase.

## 3. Architecture Principles

1. Keep the prototype local, self-contained, and review-ready for same-day delivery.
2. Prefer plain browser capabilities over frameworks, build tooling, hosting, or backend services.
3. Reuse established repository and platform patterns where appropriate.
4. Keep presentation, orchestration, domain, integration, and persistence concerns separated.

## 4. System Context

### 4.1 Actors

- General Prototype Visitor — Opens `todo-prototype.html` in Desktop Chrome and performs add, blank-validation, complete, delete, and refresh-reset interactions.
- Small Business Stakeholder — Reviews the prototype, compares the core todo experience against competitor todo apps, and provides informal proceed/no-proceed feedback.
- Developer or Prototyping Resource — Creates and shares the single HTML file for same-day stakeholder review.

### 4.2 External Systems

| ID | System | Purpose | Interface | Owner |
|---|---|---|---|---|
| EXT-001 | Desktop Chrome | Runtime environment for the local HTML prototype | Browser/File | Reviewer |
| EXT-002 | Local File System | Opens the shared `todo-prototype.html` file directly | File | Reviewer |

## 5. Technology Stack

| Layer | Technology | Version/Constraint | Rationale |
|---|---|---|---|
| Frontend | Semantic HTML, inline CSS, vanilla JavaScript | Single self-contained `.html` file | Satisfies PRD-NFR-001 and supports same-day delivery without build tooling. |
| Backend | None | Backend services are out of scope | Satisfies CON-004 and keeps the prototype local-only. |
| Data | In-memory JavaScript array of task objects | Page-session only; no persistence after refresh | Satisfies PRD-FR-006 and CON-007. |
| Hosting | None; opened locally in Desktop Chrome | Shared directly as a single file | Satisfies PRD-NFR-002 and the delivery decision. |

## 6. Logical Architecture

### 6.1 Components

#### COMP-001 — HTML Document Shell

- **Responsibility:** Provides the page structure, task input form, empty-state container, task list container, and inline script/style blocks.
- **Inputs:** Local file open event, user-visible markup requirements.
- **Outputs:** Rendered prototype shell in Desktop Chrome.
- **Dependencies:** Desktop Chrome runtime.
- **Repository Area:** `todo-prototype.html`

#### COMP-002 — Todo State Controller

- **Responsibility:** Maintains the page-session task array and coordinates add, complete, delete, and reset-on-refresh behavior.
- **Inputs:** Valid add-task commands, complete-task commands, delete-task commands.
- **Outputs:** Updated in-memory task array and render requests.
- **Dependencies:** COMP-003, COMP-004.
- **Repository Area:** `todo-prototype.html`

#### COMP-003 — Input Validation Handler

- **Responsibility:** Checks submitted task text for empty or whitespace-only values and controls the inline error message.
- **Inputs:** Task input field value.
- **Outputs:** Validation result and inline error visibility state.
- **Dependencies:** COMP-001.
- **Repository Area:** `todo-prototype.html`

#### COMP-004 — Todo Renderer

- **Responsibility:** Renders the empty prompt, incomplete tasks, completed visual state, and delete controls from current in-memory state.
- **Inputs:** In-memory task array and validation state.
- **Outputs:** Updated DOM elements visible to the reviewer.
- **Dependencies:** COMP-001, COMP-002.
- **Repository Area:** `todo-prototype.html`

### 6.2 Data Flow

1. General Prototype Visitor opens `todo-prototype.html` from the local file system in Desktop Chrome.
2. COMP-001 initializes an empty page-session state and displays the empty prompt.
3. Visitor submits task text through the input form.
4. COMP-003 validates that task text is not empty or whitespace-only.
5. COMP-002 appends a task object with id, text, and completed state to the in-memory array.
6. COMP-004 renders the current list and clears the empty prompt when tasks exist.
7. Visitor activates a completion control; COMP-002 updates only the selected task state and COMP-004 rerenders the completed visual state.
8. Visitor activates a delete button; COMP-002 removes only the selected task and COMP-004 rerenders the list or empty prompt.
9. Browser refresh discards JavaScript memory and returns the prototype to its initial empty state.

## 7. Architecture Decisions

### ARD-DEC-001 — Deliver a Local Single-File Prototype

- **Status:** Accepted
- **Related Requirements:** PRD-FR-006, PRD-NFR-001, PRD-NFR-002

**Decision**

Deliver the prototype as a root-level `todo-prototype.html` file shared directly and opened locally in Desktop Chrome.

**Rationale**

The PRD requires same-day delivery, no backend, Desktop Chrome support, no persistence, and a self-contained file. Local file delivery avoids deployment overhead and keeps the review focused on core interactions.

**Consequences**

- Stakeholders can review without a server or hosting platform.
- Browser and file-sharing behavior become part of the review experience.
- Production deployment concerns are intentionally deferred.

**Alternatives Considered**

- Internal static hosting — Not selected because hosting is unnecessary for same-day local review.
- Public static URL — Not selected because public hosting adds avoidable exposure and operational overhead.

### ARD-DEC-002 — Use Plain HTML, Inline CSS, and Vanilla JavaScript

- **Status:** Accepted
- **Related Requirements:** PRD-NFR-001, PRD-NFR-002

**Decision**

Implement the prototype with semantic HTML, inline CSS, and vanilla JavaScript in the same HTML file.

**Rationale**

This satisfies the single-file requirement while minimizing dependencies, setup, and implementation time.

**Consequences**

- No build tooling is required.
- The implementation remains easy to inspect and share.
- Larger production patterns such as modules, package management, and automated bundling are out of scope.

**Alternatives Considered**

- CDN framework — Not selected because it introduces an external dependency and supply-chain considerations.
- Web components — Not selected because they add structure not needed for the small prototype.

### ARD-DEC-003 — Store Tasks in Page-Session Memory Only

- **Status:** Accepted
- **Related Requirements:** PRD-FR-002, PRD-FR-004, PRD-FR-005, PRD-FR-006

**Decision**

Represent tasks as an in-memory JavaScript array of task objects with id, text, and completed state.

**Rationale**

The task array cleanly supports independent add, complete, and delete behavior while ensuring refresh clears all tasks as required.

**Consequences**

- Refresh reset is automatic because JavaScript memory is discarded.
- No migration, storage encryption, or data cleanup process is required.
- User-entered tasks are lost on refresh by design.

**Alternatives Considered**

- DOM-only state — Not selected because it makes independent task updates less explicit.
- localStorage — Not selected because persistence after refresh is explicitly out of scope.

### ARD-DEC-004 — Use Manual Desktop Chrome Smoke Testing as Acceptance Validation

- **Status:** Accepted
- **Related Requirements:** PRD-FR-001, PRD-FR-002, PRD-FR-003, PRD-FR-004, PRD-FR-005, PRD-FR-006, PRD-NFR-002

**Decision**

Block acceptance on a manual Desktop Chrome smoke test covering initial empty state, add task, blank task validation, mark complete, delete, and refresh reset.

**Rationale**

The artifact is a lightweight same-day prototype with no build pipeline, no framework, and no formal production readiness requirement.

**Consequences**

- Validation is fast and aligned to stakeholder review needs.
- There is no numeric code coverage gate.
- Regression confidence is limited compared with automated test suites.

**Alternatives Considered**

- Automated unit tests plus smoke testing — Not selected because it adds setup beyond the agreed prototype scope.
- Browser automation — Not selected because same-day manual review is sufficient for this phase.

### ARD-DEC-005 — Omit Runtime Observability and Operational Telemetry

- **Status:** Accepted
- **Related Requirements:** PRD-NFR-001, PRD-NFR-002

**Decision**

Do not add logs, metrics, tracing, alerting, analytics, or external telemetry. Issues are observed manually during stakeholder review.

**Rationale**

The prototype has no backend, hosting, users at scale, or production operations requirement.

**Consequences**

- No operational monitoring setup is needed.
- No analytics data is collected.
- Stakeholder feedback and manual observation are the only review signals.

**Alternatives Considered**

- Browser console logs — Not selected because they are not needed for stakeholder review.
- External analytics endpoint — Not selected because analytics is out of scope.

### ARD-DEC-006 — Use No External Dependencies and Maintain a Manual SBOM Entry

- **Status:** Accepted
- **Related Requirements:** PRD-NFR-001

**Decision**

Use no external runtime dependencies and record a manual SBOM entry for the single HTML artifact.

**Rationale**

Avoiding dependencies reduces supply-chain risk while the manual SBOM entry documents the delivered artifact.

**Consequences**

- CVE scanning has no package dependencies to evaluate.
- The SBOM is manual rather than generated by a package manager.
- Any future dependency addition must revisit scanning and SBOM controls.

**Alternatives Considered**

- No SBOM — Not selected because the architecture decision requires a manual SBOM entry.
- Automated SBOM generation — Not selected because there is no package manifest or build system.

### ARD-DEC-007 — Skip Prototype Input Safety Controls Beyond Blank Validation

- **Status:** Accepted
- **Related Requirements:** PRD-FR-003, PRD-NFR-001

**Decision**

The prototype will not add security-specific input safety controls beyond the required blank-task validation because it runs locally and is not production-ready.

**Rationale**

The stakeholder decision confirmed that input safety controls are not required for the local file prototype.

**Consequences**

- The architecture remains minimal for same-day delivery.
- Security hardening is not demonstrated in this prototype.
- A future production version must revisit untrusted input handling.

**Alternatives Considered**

- Treat task text as untrusted and render only as text content — Not selected by stakeholder decision for this prototype phase.
- Third-party sanitization library — Not selected because external dependencies are not used.

## 8. API and Integration Architecture

### ARD-API-001 — Add Task UI Event

- **Type:** Event
- **Operation/Topic:** `submit todo form`
- **Provider:** COMP-001
- **Consumers:** COMP-003, COMP-002, COMP-004
- **Authentication:** Not applicable; local prototype has no accounts.
- **Versioning:** Not versioned; internal browser event contract within `todo-prototype.html`.

**Request/Input Contract**

Task text entered by the visitor in the input field. Blank and whitespace-only values are invalid per PRD-FR-003.

**Response/Output Contract**

For valid input, a new in-memory task object is created with id, text, and completed state set to incomplete, the input field clears, and the task list rerenders. For invalid input, no task is added and an inline error message is shown.

**Error Handling**

Blank or whitespace-only submissions are rejected inline. No retry, network error, or idempotency handling is required because the interaction is local and browser-only.

### ARD-API-002 — Complete Task UI Event

- **Type:** Event
- **Operation/Topic:** `click complete control`
- **Provider:** COMP-001
- **Consumers:** COMP-002, COMP-004
- **Authentication:** Not applicable; local prototype has no accounts.
- **Versioning:** Not versioned; internal browser event contract within `todo-prototype.html`.

**Request/Input Contract**

Selected task identifier from the rendered task item.

**Response/Output Contract**

Only the selected task's completed state changes, and the renderer updates the selected task's visual completed state.

**Error Handling**

If the selected task is not present in memory, no task state changes are made. No retry or idempotency handling is required.

### ARD-API-003 — Delete Task UI Event

- **Type:** Event
- **Operation/Topic:** `click delete button`
- **Provider:** COMP-001
- **Consumers:** COMP-002, COMP-004
- **Authentication:** Not applicable; local prototype has no accounts.
- **Versioning:** Not versioned; internal browser event contract within `todo-prototype.html`.

**Request/Input Contract**

Selected task identifier from the rendered task item.

**Response/Output Contract**

The selected task is removed immediately without confirmation, and the renderer updates the list or empty state.

**Error Handling**

If the selected task is not present in memory, no task state changes are made. No retry or idempotency handling is required.

## 9. Data Architecture

### DATA-001 — Task

- **Purpose:** Represents one visitor-created todo item during the current browser page session.
- **System of Record:** In-memory JavaScript array in `todo-prototype.html`.
- **Classification:** Internal
- **Retention:** Current browser page session only; discarded on refresh or tab close.
- **Key Relationships:** Owned by the current local page session; rendered by COMP-004 and modified by COMP-002.

### Data Migration

- **Required:** No
- **Approach:** Not applicable because the prototype has no backend, database, localStorage persistence, or pre-existing data.

## 10. Security Architecture

### ARD-SEC-001 — Local Static Prototype Security Boundary

- **Related Requirements:** PRD-NFR-001, PRD-NFR-003

The implementation must:

- Run as a local static HTML file without backend services.
- Avoid external runtime dependencies.
- Avoid user accounts, authentication flows, authorization rules, persisted data, secrets, audit trails, and analytics endpoints.
- Include a manual SBOM entry for the single HTML artifact.

The implementation must not:

- Send task data to a backend or external endpoint.
- Store task data in localStorage, cookies, IndexedDB, or a remote system.
- Introduce third-party CDN dependencies without revisiting ARD-DEC-006.
- Claim production security readiness or formal accessibility compliance for this prototype phase.

### Security Controls

- **Authentication:** Not applicable; the prototype has no accounts or protected resources.
- **Authorization:** Not applicable; all interactions are local to the open browser page.
- **Secrets Management:** Not applicable; no secrets are used or stored.
- **Encryption:** Not applicable for stored data because no persistence exists; transport encryption is not applicable because there is no network transport.
- **Audit Logging:** Not applicable; no audit log is required for local prototype review.
- **Dependency/CVE Scanning:** No external dependencies are used; blocking severity policy is not applicable for this prototype unless dependencies are added later.
- **SBOM:** Required as a manual SBOM entry for `todo-prototype.html`.

## 11. Reliability, Performance, and Scalability

### ARD-OPS-001 — Local File Availability

- **Target:** `todo-prototype.html` opens successfully in Desktop Chrome during stakeholder review.
- **Validation Method:** Manual Desktop Chrome smoke test opens the file locally and verifies the initial empty state appears.

### ARD-OPS-002 — Core Interaction Responsiveness

- **Target:** Add, blank-validation, complete, delete, and refresh-reset interactions complete without visible blocking or browser error during manual review.
- **Validation Method:** Manual Desktop Chrome smoke test executes each required interaction and records pass/fail.

### ARD-OPS-003 — Review-Scale Operation

- **Target:** The prototype supports local single-reviewer use in Desktop Chrome for stakeholder review.
- **Validation Method:** Stakeholder or developer opens the file locally and performs the PRD-defined user journeys without backend or hosting services.

## 12. Observability

- **Logging:** No runtime logging is required.
- **Metrics:** No application metrics are required.
- **Tracing:** No distributed or browser tracing is required.
- **Alerting:** No alerting is required.
- **Sensitive Data Restrictions:** The prototype must not send task text to external endpoints or persist it beyond the current page session.

## 13. Deployment Architecture

- **Environments:** Local reviewer Desktop Chrome only; no dev, test, stage, or production hosted environments are required.
- **Deployment Model:** Direct sharing of the root-level `todo-prototype.html` file for local opening.
- **CI/CD:** No CI/CD pipeline is required for this same-day local prototype.
- **Configuration Management:** Inline configuration only; no environment variables or external configuration files.
- **Rollback:** Replace the shared HTML file with a previous known-good copy if needed.
- **Infrastructure as Code:** Not applicable; no infrastructure is provisioned.

## 14. Testing Architecture

Required test levels:

- Unit tests — Not required for this prototype.
- Integration tests — Not required beyond manual verification of integrated UI interactions in Desktop Chrome.
- Contract tests — Not required because there are no external APIs.
- End-to-end tests — Manual Desktop Chrome smoke test is required.
- Security tests — Not required beyond verifying no external dependencies, no backend calls, no persistence, and manual SBOM entry.
- Performance tests, where applicable — Manual confirmation that core interactions complete without visible blocking or browser error.

Minimum quality expectations:

- **Coverage Threshold:** No numeric coverage threshold; acceptance is manual Desktop Chrome smoke testing only.
- **Blocking Security Severity:** Not applicable while no external dependencies are used; if dependencies are added later, this must be reopened under Open Architecture Questions.
- **Required Build/Lint Checks:** No build or lint checks are required; the single HTML file must pass the manual smoke test.

## 15. Architecture Constraints

- ARC-CON-001 — The company context is a small business.
- ARC-CON-002 — The prototype must be delivered same-day.
- ARC-CON-003 — The prototype must be implemented as one self-contained HTML file with inline CSS and JavaScript.
- ARC-CON-004 — The prototype must not use a backend service.
- ARC-CON-005 — The prototype must target Desktop Chrome only.
- ARC-CON-006 — The prototype must use a clean modern layout with simple responsive styling.
- ARC-CON-007 — The prototype must not persist tasks after refresh.
- ARC-CON-008 — Formal accessibility compliance is not required for this prototype phase.
- ARC-CON-009 — The artifact path is `todo-prototype.html` at the repository or delivery package root.
- ARC-CON-010 — The prototype must use no external runtime dependencies.
- ARC-CON-011 — The prototype must include a manual SBOM entry for the single HTML artifact.

## 16. Risks and Trade-offs

| ID | Risk/Trade-off | Impact | Mitigation |
|---|---|---|---|
| ARC-RISK-001 | Stakeholders expect production-level features from a prototype. | Could cause scope confusion during review. | Clearly label the artifact and review discussion as prototype-only using the PRD scope. |
| ARC-RISK-002 | Lack of formal metrics makes decision quality subjective. | Proceed/no-proceed decision may depend on qualitative feedback only. | Capture informal stakeholder feedback and the review outcome. |
| ARC-RISK-003 | Desktop Chrome-only support limits review coverage. | Behavior in other browsers is not validated. | Confirm reviewers use Desktop Chrome as specified by PRD-NFR-002. |
| ARC-RISK-004 | No persistence after refresh may be mistaken for a defect. | Reviewers may think tasks were lost unexpectedly. | Communicate reset-on-refresh behavior as an intentional PRD-FR-006 constraint. |
| ARC-RISK-005 | Skipping input safety controls limits security representativeness. | The prototype does not demonstrate production-safe handling of untrusted input. | Revisit input handling before any production implementation. |
| ARC-RISK-006 | No automated tests limits regression confidence. | Manual-only validation can miss implementation defects. | Execute the required Desktop Chrome smoke test before acceptance. |

## 17. Open Architecture Questions

| ID | Question | Owner | Status | Decision/Due Date |
|---|---|---|---|---|
| ARC-OQ-001 | No open architecture questions identified after interview; proceed with documented decisions. | Architecture Owner | Closed | 2026-11-19 |

## 18. Feature Mapping

| Feature | Relevant Components | Decision IDs | Interface IDs | Security IDs |
|---|---|---|---|---|
| FEAT-001 | COMP-001, COMP-002, COMP-003, COMP-004 | ARD-DEC-001, ARD-DEC-002, ARD-DEC-003, ARD-DEC-004, ARD-DEC-005, ARD-DEC-006, ARD-DEC-007 | ARD-API-001, ARD-API-002, ARD-API-003 | ARD-SEC-001 |

## 19. Approval

| Role | Name | Decision | Date |
|---|---|---|---|
| Solution Architect | TBD | Pending | TBD |
| Security Reviewer | TBD | Pending | TBD |
| Engineering Lead | TBD | Pending | TBD |