# Architecture Requirements Document (ARD)

> Replace all `<PLACEHOLDER>` values. Every material architecture decision or constraint should have a stable ID that feature specifications can reference.

## Document Metadata

- **Product/Application:** Single-File Todo App
- **Document ID:** ARD-TODO-SINGLE-HTML
- **Version:** 0.1
- **Status:** Draft
- **Architecture Owner:** Individual user/product owner
- **Contributors:** Individual user/product owner, Solution Architect
- **Last Updated:** 2026-11-20
- **Reference PRD:** `PRD.md`

---

## 1. Architecture Summary

The Single-File Todo App will be implemented as one self-contained `index.html` file at the repository root. The file will contain embedded HTML, CSS, and JavaScript with no external libraries, APIs, fonts, CDNs, analytics, backend services, or build artifacts. The user opens the file directly from the local filesystem in a current browser.

The application architecture is a local-only browser application with three logical concerns inside the single file: presentation, client-side orchestration/domain logic, and local persistence. The UI collects valid todo text, JavaScript validates and creates a local task record, and browser `localStorage` stores an ordered array of task records. Tasks are rendered newest-first using text-only DOM APIs such as `textContent` to prevent task text from being interpreted as HTML or script.

Key architecture drivers are simplicity, local-only privacy, single-file delivery, browser compatibility, and reliable persistence of at least 10 tasks across refresh in current Chrome, Edge, Firefox, and Safari.

## 2. Architecture Drivers

- PRD-FR-001 — The architecture must support adding a valid task from the page and immediately displaying it in the todo list.
- PRD-FR-002 — The architecture must enforce client-side task validation of 1–120 trimmed characters and keep the add button disabled until input is valid.
- PRD-FR-003 — The architecture must persist tasks in browser local storage so they remain available after refresh in the same browser and device.
- PRD-FR-004 — The architecture must preserve and render newest-first task ordering.
- PRD-FR-005 — The architecture must detect local storage write failures and show a clear save error without implying persistence succeeded.
- PRD-FR-006 — The architecture must provide a clean, minimal, responsive UI within the single HTML page.
- PRD-NFR-001 — The architecture must work in current Chrome, Edge, Firefox, and Safari with keyboard-operable input and add button.
- PRD-NFR-002 — The architecture must keep tasks local to browser storage and transmit no task data externally.
- PRD-NFR-003 — The architecture must be one self-contained HTML file with embedded CSS and JavaScript and no external libraries.
- CON-001 — The app must be delivered as one self-contained HTML file.
- CON-002 — CSS and JavaScript must be embedded in the HTML file.
- CON-003 — No external libraries, CDNs, APIs, or backend services may be used.
- CON-004 — Tasks must remain local to the user’s browser storage and must not be transmitted externally.
- CON-005 — The first version must not include completion, deletion, editing, collaboration, due dates, priorities, or categories.

## 3. Architecture Principles

1. Prefer the simplest local browser architecture that satisfies the approved first-version scope.
2. Keep all task data local to the user’s browser and avoid any external network dependency.
3. Reuse established repository and platform patterns where appropriate.
4. Keep presentation, orchestration, domain, integration, and persistence concerns separated.
5. Treat task text as untrusted input and render it only through text-safe DOM APIs.
6. Make acceptance evidence obtainable through manual browser testing and file inspection.

## 4. System Context

### 4.1 Actors

- Individual Todo User — Opens `index.html` from the local filesystem, enters task text, submits valid tasks, refreshes the page, and expects saved tasks to remain visible in the same browser.
- Browser Runtime — Executes the embedded HTML, CSS, and JavaScript; provides DOM events, responsive rendering, keyboard interaction, and access to `localStorage`.
- Browser Local Storage — Stores task records locally on the same device and browser; rejects writes when unavailable, disabled, or full.
- File System — Provides the local `index.html` file opened directly by the user.

### 4.2 External Systems

| ID | System | Purpose | Interface | Owner |
|---|---|---|---|---|
| EXT-001 | Local file system | Stores and opens the `index.html` deliverable | File | Individual user/product owner |
| EXT-002 | Current Chrome, Edge, Firefox, and Safari browsers | Execute the single-file app and provide DOM, keyboard, CSS, and JavaScript runtime behavior | Event | Browser vendors |
| EXT-003 | Browser localStorage | Persist task records locally in the same browser and device | Browser API | Browser vendors |

## 5. Technology Stack

| Layer | Technology | Version/Constraint | Rationale |
|---|---|---|---|
| Frontend | HTML, embedded CSS, embedded vanilla JavaScript | Current Chrome, Edge, Firefox, and Safari; no external libraries | Satisfies single-file delivery, responsive UI, and local browser execution requirements |
| Backend | None | Backend services are prohibited by CON-003 | The app is local-only and requires no server-side processing |
| Data | Browser `localStorage` | Same browser and device only; stores a JSON array under a versioned key | Satisfies local persistence without backend, database, or network transmission |
| Hosting | Local filesystem | User opens root `index.html` directly in a browser | Matches the first-release operational model and avoids deployment infrastructure |

## 6. Logical Architecture

### 6.1 Components

#### COMP-001 — Single-File Application Shell

- **Responsibility:** Provide the complete deliverable, page structure, embedded styles, embedded script, and local-only runtime boundary.
- **Inputs:** Local file open request from the user through a supported browser.
- **Outputs:** Rendered todo application UI.
- **Dependencies:** EXT-001, EXT-002.
- **Repository Area:** `index.html`

#### COMP-002 — Responsive Presentation Layer

- **Responsibility:** Render the task input, disabled/enabled add button, task list, and local storage error message in a clean minimal responsive layout.
- **Inputs:** Task state, validation state, and error state from COMP-003 and COMP-004.
- **Outputs:** Keyboard-operable UI and visible newest-first task list.
- **Dependencies:** COMP-003, COMP-004, EXT-002.
- **Repository Area:** `index.html`

#### COMP-003 — Task Controller and Validation Logic

- **Responsibility:** Handle input and add events, trim task text, enforce 1–120 character validation, create task records, clear input after successful add, and coordinate persistence and rendering.
- **Inputs:** User keyboard and click/form events, raw task text from COMP-002.
- **Outputs:** Validated task records, validation state, save requests, and render requests.
- **Dependencies:** COMP-002, COMP-004, COMP-005.
- **Repository Area:** `index.html`

#### COMP-004 — Local Task Store Adapter

- **Responsibility:** Load and save task records in browser `localStorage`, preserve newest-first ordering, and surface storage availability or quota failures.
- **Inputs:** Task record arrays from COMP-003 and persisted JSON from EXT-003.
- **Outputs:** Loaded task arrays or persistence errors.
- **Dependencies:** EXT-003.
- **Repository Area:** `index.html`

#### COMP-005 — Safe Task Renderer

- **Responsibility:** Render task text into the DOM using text-only APIs such as `textContent`, never interpreting user-provided task text as HTML.
- **Inputs:** Task records from COMP-003 and COMP-004.
- **Outputs:** DOM list items containing task text and created-order presentation.
- **Dependencies:** COMP-002, EXT-002.
- **Repository Area:** `index.html`

### 6.2 Data Flow

1. The Individual Todo User opens `index.html` from the local filesystem in a supported browser.
2. COMP-001 loads embedded HTML, CSS, and JavaScript without external network resources.
3. COMP-004 reads the persisted JSON task array from browser `localStorage`.
4. COMP-005 renders loaded tasks into COMP-002 newest-first using text-only DOM APIs.
5. The user enters task text into COMP-002.
6. COMP-003 trims and validates the task text on input and enables the add button only when the text is 1–120 characters.
7. The user activates the enabled add button by keyboard or pointer.
8. COMP-003 creates a task record containing generated local ID, trimmed task text, and created timestamp.
9. COMP-003 prepends the task record to the in-memory task array to maintain newest-first ordering.
10. COMP-004 writes the updated task array to browser `localStorage` as JSON.
11. If the write succeeds, COMP-005 renders the updated list and COMP-003 clears the input.
12. If the write fails, COMP-004 returns an error, COMP-002 shows a clear local-save error, and the app does not present the task as successfully persisted.

## 7. Architecture Decisions

### ARD-DEC-001 — Deliver as Root Single HTML File

- **Status:** Accepted
- **Related Requirements:** PRD-FR-006, PRD-NFR-003

**Decision**

The deliverable will be one self-contained `index.html` file at the repository root with embedded HTML, CSS, and JavaScript.

**Rationale**

This directly satisfies the approved single-file requirement and avoids build tooling, packaging, hosting, and dependency management for a small local app.

**Consequences**

- The app can be opened directly from the local filesystem.
- CSS and JavaScript must remain maintainable within one file.
- Separate automated tooling is not required for the first version.

**Alternatives Considered**

- Separate HTML, CSS, and JavaScript files — Not selected because PRD-NFR-003 and CON-001 require a single self-contained file.
- Static hosted web app — Not selected because the first-release operational model is local filesystem usage.

### ARD-DEC-002 — Use Vanilla Browser Runtime Only

- **Status:** Accepted
- **Related Requirements:** PRD-NFR-001, PRD-NFR-002, PRD-NFR-003

**Decision**

The app will use only standard HTML, CSS, and vanilla JavaScript supported by current Chrome, Edge, Firefox, and Safari.

**Rationale**

No external libraries, CDNs, APIs, or backend services are allowed, and the required scope is simple enough for platform-native browser capabilities.

**Consequences**

- There is no package manager, dependency install, bundler, or framework runtime.
- Browser compatibility must be validated manually across supported browsers.
- Feature complexity must remain intentionally small.

**Alternatives Considered**

- CDN framework or utility library — Not selected because external libraries and CDNs are prohibited.
- Backend-backed web app — Not selected because backend services are prohibited and unnecessary.

### ARD-DEC-003 — Persist Tasks in Browser localStorage

- **Status:** Accepted
- **Related Requirements:** PRD-FR-003, PRD-FR-004, PRD-FR-005, PRD-NFR-002

**Decision**

Tasks will be stored as a JSON array in browser `localStorage` under a versioned application key.

**Rationale**

`localStorage` provides simple same-browser persistence across refresh without external services, databases, accounts, synchronization, or network transmission.

**Consequences**

- Data persists only in the same browser and device.
- Browser storage availability, privacy settings, and quota limits can prevent persistence.
- The app must catch storage read/write errors and show a clear local-save error.

**Alternatives Considered**

- In-memory storage only — Not selected because tasks must persist after refresh.
- IndexedDB — Not selected because the data model is simple and `localStorage` is sufficient for at least 10 tasks.
- Backend database — Not selected because backend services and external synchronization are out of scope.

### ARD-DEC-004 — Store Task Text, Created Timestamp, and Local ID

- **Status:** Accepted
- **Related Requirements:** PRD-FR-001, PRD-FR-003, PRD-FR-004

**Decision**

Each task record will contain task text, created timestamp, and a generated local ID.

**Rationale**

Task text is required for display, the timestamp supports stable newest-first ordering, and the generated local ID provides stable local identity without adding out-of-scope edit/delete behavior.

**Consequences**

- Task data remains minimal and aligned to the first-version scope.
- Future features can reference stable task IDs if completion or deletion is later approved.
- The app must generate IDs locally without external services.

**Alternatives Considered**

- Task text only — Not selected because it provides no stable local identity or explicit created-order metadata.
- Task text plus timestamp only — Not selected because a local ID is a low-cost way to avoid ambiguity between duplicate task texts.

### ARD-DEC-005 — Render Task Text with Text-Only DOM APIs

- **Status:** Accepted
- **Related Requirements:** PRD-FR-001, PRD-NFR-002

**Decision**

User-provided task text will be rendered with text-only DOM APIs such as `textContent`; task text must not be assigned to `innerHTML` or otherwise interpreted as markup.

**Rationale**

Task text is untrusted user input even in a local app. Text-only rendering prevents script injection and preserves local-only safety without requiring sanitization libraries.

**Consequences**

- User input containing HTML-like text is displayed as text, not markup.
- No external sanitizer dependency is needed.
- Developers must keep rendering logic separated from HTML template construction.

**Alternatives Considered**

- Escape then assign to `innerHTML` — Not selected because text-only DOM APIs are simpler and less error-prone.
- Rely on the 120-character limit — Not selected because length limits do not prevent injection.

### ARD-DEC-006 — Validate by Manual Browser Acceptance and Inspection

- **Status:** Accepted
- **Related Requirements:** PRD-FR-001, PRD-FR-002, PRD-FR-003, PRD-FR-004, PRD-FR-005, PRD-FR-006, PRD-NFR-001, PRD-NFR-002, PRD-NFR-003

**Decision**

Acceptance for the first version will use manual browser acceptance tests, browser developer tools network verification, and file inspection. No automated coverage, build, or lint gate is required.

**Rationale**

The product is a very small self-contained local file with no build pipeline or dependencies. Manual verification directly validates the PRD acceptance criteria and architecture constraints without adding tooling complexity.

**Consequences**

- Validation effort is lightweight and appropriate for the small-business, single-user context.
- Regression coverage is manual for the first version.
- If future scope grows, automated tests and linting should be reconsidered.

**Alternatives Considered**

- Automated test runner and lint tooling — Not selected for this version because it adds project structure and tooling beyond the single-file scope.
- Embedded automated tests in the HTML file — Not selected because acceptance can be evidenced through manual browser tests and file inspection.

### ARD-DEC-007 — Operate as Local File with Manual Replacement Updates

- **Status:** Accepted
- **Related Requirements:** PRD-NFR-002, PRD-NFR-003

**Decision**

The app will have no runtime operations service. The user keeps and opens the local `index.html` file, and updates are delivered by manually replacing the file.

**Rationale**

The approved scope excludes hosting, backend services, app-store distribution, and synchronization. Local file operation is the simplest deployment model.

**Consequences**

- There is no server uptime, deployment pipeline, or rollback service.
- Rollback means restoring a prior copy of `index.html`.
- Replacing the file does not automatically migrate or clear browser `localStorage` data if the storage key remains compatible.

**Alternatives Considered**

- Static web hosting — Not selected because first release is opened from the local filesystem.
- CI/CD artifact publishing — Not selected because no deployment pipeline is required for the first version.

## 8. API and Integration Architecture

### ARD-API-001 — Add Task UI Event Interface

- **Type:** Event
- **Operation/Topic:** `input` and `submit` or `click` events within `index.html`
- **Provider:** COMP-002
- **Consumers:** COMP-003
- **Authentication:** None; local single-user browser session only
- **Versioning:** Versioned with the `index.html` file; no external interface versioning

**Request/Input Contract**

Raw task text entered by the user in the task input field. Valid task text is the trimmed input with length from 1 to 120 characters.

**Response/Output Contract**

Validation state updates the add button enabled/disabled state. A successful add request produces a task record request for persistence with `id`, `text`, and `createdAt`.

**Error Handling**

Invalid input keeps the add button disabled and prevents task creation. Storage errors are delegated to ARD-API-002 and surfaced as visible local-save errors. Add operations are not retried automatically.

### ARD-API-002 — Browser localStorage Persistence Interface

- **Type:** File
- **Operation/Topic:** `localStorage` versioned key, recommended key `singleFileTodo.tasks.v1`
- **Provider:** COMP-004
- **Consumers:** COMP-003, COMP-005
- **Authentication:** Browser local storage access in the local user’s browser context
- **Versioning:** Versioned storage key suffix `.v1`; future incompatible schemas require a new key or explicit migration decision

**Request/Input Contract**

A JSON-serializable array of task records. Each record has:

```json
{
  "id": "locally-generated-string",
  "text": "1 to 120 character trimmed task text",
  "createdAt": "ISO-8601 timestamp string"
}
```

**Response/Output Contract**

On load, returns a task record array or an empty array when no valid stored task array exists. On save, returns success only after `localStorage` accepts the updated serialized task array.

**Error Handling**

The adapter must catch unavailable storage, quota exceeded, serialization, and parse errors. Save failure must produce a visible message that tasks cannot be saved locally and must not present the task as successfully persisted. Parse failure handling may ignore invalid stored data and show no tasks, with the detailed recovery behavior listed under Open Architecture Questions.

### ARD-API-003 — Static File Delivery Interface

- **Type:** File
- **Operation/Topic:** `index.html`
- **Provider:** EXT-001
- **Consumers:** EXT-002
- **Authentication:** Local operating system file access controls
- **Versioning:** File replacement; version is tracked in ARD/PRD metadata and any file-level visible version if implemented

**Request/Input Contract**

The browser opens `index.html` from the local filesystem.

**Response/Output Contract**

The browser receives a single HTML document containing all CSS and JavaScript required to run the app.

**Error Handling**

If the file cannot be opened, the browser or operating system displays the relevant local file error. The app has no runtime mechanism to recover from a missing or unreadable file.

## 9. Data Architecture

### DATA-001 — Todo Task

- **Purpose:** Represents one user-entered personal task to display and persist locally.
- **System of Record:** Browser `localStorage` in the same browser and device.
- **Classification:** Internal
- **Retention:** Retained in browser `localStorage` until the user or browser clears site/file storage, the file is run in a different browser context, or a future version changes retention behavior.
- **Key Relationships:** Stored as part of DATA-002; no relationship to users, accounts, external systems, categories, due dates, completion state, or deletion state in this version.

### DATA-002 — Todo Task Collection

- **Purpose:** Ordered collection of todo tasks rendered newest-first and persisted as JSON.
- **System of Record:** Browser `localStorage` under a versioned application key.
- **Classification:** Internal
- **Retention:** Same as DATA-001; no automatic expiration is defined for the first version.
- **Key Relationships:** Contains zero or more DATA-001 records ordered by `createdAt` newest-first or by array order maintained newest-first.

### Data Migration

- **Required:** No
- **Approach:** No migration is required because there is no existing application data source in scope. Future incompatible storage schema changes must be handled through a new architecture decision and either a new versioned key or explicit migration logic.

## 10. Security Architecture

### ARD-SEC-001 — Local-Only Data Boundary

- **Related Requirements:** PRD-NFR-002, PRD-NFR-003

The implementation must:

- Keep task data only in browser `localStorage` on the same device and browser.
- Avoid all external scripts, stylesheets, fonts, images, APIs, analytics, CDNs, and backend calls.
- Be verifiable by file inspection and browser developer tools showing no external network activity during add and refresh flows.

The implementation must not:

- Transmit task data to any external service.
- Include tracking, analytics, or remote error reporting.
- Depend on external resources to create, display, or persist tasks.

### ARD-SEC-002 — Safe Task Text Rendering

- **Related Requirements:** PRD-FR-001, PRD-NFR-002

The implementation must:

- Treat task text as untrusted input.
- Render task text through text-only DOM APIs such as `textContent`.
- Preserve user-entered text as visible text after trimming, without interpreting it as HTML.

The implementation must not:

- Assign user-provided task text to `innerHTML`.
- Execute or inject user-provided task text as markup, script, style, or event-handler content.
- Rely on task length validation as the only injection protection.

### ARD-SEC-003 — Storage Failure Transparency

- **Related Requirements:** PRD-FR-005, PRD-NFR-002

The implementation must:

- Catch browser storage write failures, including unavailable or full `localStorage`.
- Show a clear visible error when tasks cannot be saved locally.
- Avoid presenting a task as successfully persisted when the save operation fails.

The implementation must not:

- Silently ignore local storage failures.
- Claim cross-browser or cross-device persistence.
- Fall back to external persistence.

### Security Controls

- **Authentication:** None; single-user local file with no accounts or login.
- **Authorization:** Local operating system and browser profile access controls only; no application-level roles.
- **Secrets Management:** No secrets, API keys, tokens, credentials, or environment variables are permitted.
- **Encryption:** No application-managed encryption; transport encryption is not applicable because there is no network transport; at-rest protection depends on the user’s device, browser profile, and operating system controls.
- **Audit Logging:** No persistent audit log; browser console logging should not include task content unless needed for local debugging and removed before acceptance.
- **Dependency/CVE Scanning:** No third-party dependencies are permitted. Acceptance is blocked if file inspection finds external libraries, scripts, styles, fonts, APIs, analytics, or CDN references.
- **SBOM:** Not required for third-party packages because there are no dependencies; the deliverable inventory is the single `index.html` file.

## 11. Reliability, Performance, and Scalability

### ARD-OPS-001 — Local Persistence Reliability

- **Target:** 10 valid tasks persist across browser refresh in current Chrome, Edge, Firefox, and Safari without data loss during manual acceptance.
- **Validation Method:** Manual acceptance test in each supported browser: add 10 valid tasks, refresh, verify all 10 remain visible newest-first.

### ARD-OPS-002 — Input Validation Reliability

- **Target:** 100% of manually tested empty, whitespace-only, valid 1-character, valid 120-character, and invalid over-120-character inputs produce the expected add button state and add behavior.
- **Validation Method:** Manual UI test using boundary-value inputs in a supported browser.

### ARD-OPS-003 — Local Storage Failure Handling

- **Target:** When `localStorage` is unavailable or rejects writes during manual testing, the app shows a visible error and does not present the task as successfully persisted.
- **Validation Method:** Manual browser test using a simulated or configured storage failure, plus code inspection for `try/catch` around storage access.

### ARD-OPS-004 — Responsive Usability

- **Target:** The task input, add button, error message area, and task list are usable without horizontal scrolling at desktop-sized and mobile-sized browser widths.
- **Validation Method:** Manual responsive browser test using desktop and mobile-sized viewport widths.

### ARD-OPS-005 — Network Independence

- **Target:** Zero external network requests are required during initial open, add task, refresh, and display flows.
- **Validation Method:** Browser developer tools network panel verification and file inspection.

### ARD-OPS-006 — Scalability Boundary

- **Target:** The first version is validated for at least 10 persisted tasks; larger task volumes are not an architectural target for this release.
- **Validation Method:** Manual acceptance test for KPI-001; any higher volume behavior is considered outside validated scope.

## 12. Observability

- **Logging:** No production-style logging is required. Console logs should be absent or limited to non-sensitive local debugging and removed before acceptance when possible.
- **Metrics:** No runtime metrics collection is required or permitted through external analytics.
- **Tracing:** No tracing is required because there are no distributed services or external calls.
- **Alerting:** No automated alerting is required because there is no runtime service operation.
- **Sensitive Data Restrictions:** Task text must not be sent externally, included in analytics, logged to remote systems, or embedded into executable HTML.

## 13. Deployment Architecture

- **Environments:** Local development and local user acceptance only; no separate test, stage, or production hosting environments are required for the first version.
- **Deployment Model:** One root `index.html` file opened directly from the local filesystem in a supported browser.
- **CI/CD:** No CI/CD pipeline is required for the first version; acceptance gates are manual browser testing, browser developer tools network verification, and file inspection.
- **Configuration Management:** No external configuration, environment variables, secrets, or runtime configuration files are required.
- **Rollback:** Restore a prior known-good copy of `index.html`; browser `localStorage` task data remains browser-managed unless manually cleared by the user or changed by a future storage schema decision.
- **Infrastructure as Code:** Not applicable; no hosting infrastructure is used.

## 14. Testing Architecture

Required test levels:

- Unit tests — Not automated for this version; logic is validated through manual boundary-value tests and code inspection.
- Integration tests — Manual integration test of input validation, task creation, local storage save/load, refresh persistence, and rendering.
- Contract tests — Manual/file-inspection validation of the localStorage record contract containing `id`, `text`, and `createdAt`.
- End-to-end tests — Manual browser acceptance tests in current Chrome, Edge, Firefox, and Safari covering add, refresh, newest-first display, and local-save failure behavior where feasible.
- Security tests — File inspection and browser developer tools verification that there are no external resources, no external network requests, and no user task rendering through `innerHTML`.
- Performance tests, where applicable — Manual validation that at least 10 tasks can be added and persisted without visible failure in supported browsers.

Minimum quality expectations:

- **Coverage Threshold:** No automated coverage threshold for this version; required coverage is manual acceptance of all Must-priority PRD requirements.
- **Blocking Security Severity:** Acceptance is blocked by any finding that transmits task data externally, includes external dependencies/resources, uses `innerHTML` for task text, stores secrets, or silently ignores local storage save failure.
- **Required Build/Lint Checks:** No build or lint gate is required; required checks are manual browser acceptance, file inspection, browser developer tools network verification, and code inspection for storage error handling and text-only rendering.

## 15. Architecture Constraints

- ARC-CON-001 — The deliverable must be exactly one self-contained HTML file required to run the app.
- ARC-CON-002 — The file must be `index.html` at the repository root for this architecture.
- ARC-CON-003 — CSS and JavaScript must be embedded in `index.html`.
- ARC-CON-004 — No external JavaScript libraries, CSS files, fonts, images, APIs, analytics, CDNs, backend services, package managers, build outputs, or remote resources may be required.
- ARC-CON-005 — Tasks must remain local to browser `localStorage` and must not be transmitted externally.
- ARC-CON-006 — The first version must not include completion, deletion, editing, collaboration, due dates, priorities, categories, reminders, search, accounts, or synchronization.
- ARC-CON-007 — The app must work in current Chrome, Edge, Firefox, and Safari with keyboard-operable input and add button.
- ARC-CON-008 — Task text must be validated to 1–120 trimmed characters before add.
- ARC-CON-009 — Newly added tasks must appear at the top of the task list and preserve newest-first order after refresh.
- ARC-CON-010 — Save failures from unavailable or full local storage must produce a visible local-save error.
- ARC-CON-011 — No secrets, credentials, tokens, or environment-specific configuration may be introduced.
- ARC-CON-012 — No data migration is required for the first version because there is no existing app data source.

## 16. Risks and Trade-offs

| ID | Risk/Trade-off | Impact | Mitigation |
|---|---|---|---|
| ARC-RISK-001 | Browser `localStorage` is unavailable, disabled, full, or cleared by the user/browser | Tasks may not persist or may be lost | Catch write failures, show a clear error, and validate persistence manually across supported browsers |
| ARC-RISK-002 | The user expects tasks to sync across devices or browsers | User may think tasks are missing in another browser or device | Keep local-only behavior explicit in architecture and avoid any UI claim of cross-device availability |
| ARC-RISK-003 | Single-file architecture can become hard to maintain if scope grows | Future enhancements may make embedded CSS/JS unwieldy | Keep version 1 scope minimal and revisit architecture if completion, deletion, editing, or collaboration is approved |
| ARC-RISK-004 | No automated test or lint gate means regressions are caught manually | Manual acceptance may miss edge cases | Use a focused manual checklist mapped to PRD requirements and revisit automation if the app grows |
| ARC-RISK-005 | No application-managed encryption for local storage | Anyone with access to the browser profile or device may access tasks | Classify task data as Internal and rely on local device/browser controls; do not store secrets in tasks |
| ARC-RISK-006 | Invalid stored JSON or future schema mismatch may prevent loading existing tasks | Tasks may not render correctly after local storage corruption or incompatible changes | Catch parse errors; define detailed recovery behavior before introducing schema changes |

## 17. Open Architecture Questions

| ID | Question | Owner | Status | Decision/Due Date |
|---|---|---|---|---|
| ARC-OQ-001 | Exact recovery behavior for corrupted or invalid existing `localStorage` JSON is not settled; assumption: catch parse errors and show an empty task list without external recovery. | Individual user/product owner | Open | TBD |
| ARC-OQ-002 | Whether a visible app version label is needed inside `index.html` is not settled; assumption: ARD/PRD metadata and file replacement are sufficient for first release. | Individual user/product owner | Open | TBD |

## 18. Feature Mapping

| Feature | Relevant Components | Decision IDs | Interface IDs | Security IDs |
|---|---|---|---|---|
| FEAT-001 — Local Todo Capture | COMP-001, COMP-002, COMP-003, COMP-004, COMP-005 | ARD-DEC-001, ARD-DEC-002, ARD-DEC-003, ARD-DEC-004, ARD-DEC-005, ARD-DEC-006, ARD-DEC-007 | ARD-API-001, ARD-API-002, ARD-API-003 | ARD-SEC-001, ARD-SEC-002, ARD-SEC-003 |

## 19. Approval

| Role | Name | Decision | Date |
|---|---|---|---|
| Solution Architect | Solution Architect | Pending | TBD |
| Security Reviewer | Individual user/product owner | Pending | TBD |
| Engineering Lead | Individual user/product owner | Pending | TBD |