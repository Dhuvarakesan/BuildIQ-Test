# Architecture Requirements Document (ARD)

> Replace all `<PLACEHOLDER>` values. Every material architecture decision or constraint should have a stable ID that feature specifications can reference.

## Document Metadata

- **Product/Application:** Online Patient Scheduling and Reminders
- **Document ID:** ARD-OPSR
- **Version:** 0.1
- **Status:** Draft
- **Architecture Owner:** TBD
- **Contributors:** TBD
- **Last Updated:** 2026-11-23
- **Reference PRD:** `PRD.md`

---

## 1. Architecture Summary

Online Patient Scheduling and Reminders will be implemented by configuring and integrating a HIPAA-ready scheduling SaaS with minimal custom code. The vendor-hosted SaaS will provide the patient-facing mobile-first scheduling experience, staff/admin schedule and configuration views, OTP-based patient verification, and SMS/email reminder delivery under a signed BAA.

The existing EHR/practice management scheduling system remains the system of record for appointments and availability. The scheduling SaaS stores only operational copies needed for active scheduling workflows and synchronizes availability, booking, rescheduling, pending requests, and staff schedule view data with the EHR/practice management system, preferably through real-time APIs where available.

Key drivers are HIPAA-compliant PHI handling, privacy-safe reminder content, a small-business-appropriate implementation footprint, staff visibility into appointments and pending requests, and a pilot rollout for one clinic location.

## 2. Architecture Drivers

- PRD-FR-001 — Patients must be able to directly book available slots online for clinic-selected low-risk appointment types.
- PRD-FR-002 — Patients must be able to submit pending appointment requests for appointment types requiring staff confirmation.
- PRD-FR-003 — Patients must verify identity with an SMS or email OTP before rescheduling eligible appointments.
- PRD-FR-004 — The platform must send SMS and email reminders without exposing PHI beyond clinic name in message bodies.
- PRD-FR-005 — Staff need one daily schedule view containing confirmed appointments and pending requests grouped by provider and time.
- PRD-FR-006 — Authorized clinic users must configure direct-bookable versus staff-confirmed appointment types.
- PRD-NFR-001 — Patient scheduling data must be handled consistently with HIPAA requirements for PHI.
- PRD-NFR-002 — Reminder content must avoid PHI and route details through secure links.
- PRD-NFR-003 — Existing appointment details and rescheduling must be protected by OTP verification.
- PRD-NFR-004 — Patient-facing scheduling flows must be mobile-first, responsive, and meet WCAG 2.1 AA.
- CON-001 — The clinic operates in healthcare and must satisfy HIPAA requirements for protected health information.
- CON-002 — The small-business context requires a minimum viable architecture with low operational burden.
- CON-003 — The first release must integrate with the existing EHR/practice management scheduling system.
- CON-004 — Reminder messages must not expose PHI beyond clinic name and must use secure links for appointment details.
- CON-005 — Launch is a pilot within one clinic location; pilot date is TBD.

## 3. Architecture Principles

1. Keep the EHR/practice management system as the appointment and availability system of record.
2. Prefer HIPAA-ready vendor-managed SaaS capabilities over custom build to reduce small-business operational burden.
3. Reuse established repository and platform patterns where appropriate.
4. Keep presentation, orchestration, domain, integration, and persistence concerns separated.
5. Minimize PHI in SMS, email, logs, operational copies, and integrations.
6. Require auditable access and modification trails for scheduling records and configuration changes.

## 4. System Context

### 4.1 Actors

- Patient — Uses the patient-facing scheduling web experience to book appointments, submit pending requests, reschedule eligible appointments after OTP verification, and open secure reminder links.
- Scheduling Staff — Uses the staff daily schedule view to see confirmed appointments, pending requests, and failed reminder delivery attempts.
- Clinic Administrator — Configures appointment types as direct-bookable or staff-confirmed and helps approve pilot configuration.
- Scheduling SaaS Vendor — Operates the HIPAA-ready scheduling platform, reminder service, hosting, rollback, infrastructure, and vendor-owned observability.
- Clinic Technical/Operations Owner — Owns the EHR/practice management system relationship, integration configuration coordination, acceptance testing, and clinic-side operational review.

### 4.2 External Systems

| ID | System | Purpose | Interface | Owner |
|---|---|---|---|---|
| EXT-001 | Existing EHR/practice management scheduling system | System of record for appointments and availability | REST/API preferred; exact method TBD | Clinic |
| EXT-002 | HIPAA-ready scheduling SaaS | Patient scheduling, staff schedule view, appointment type configuration, OTP verification, reminders, audit reports | Web/REST/API | SaaS Vendor |
| EXT-003 | SaaS reminder delivery service | SMS and email appointment reminders and OTP delivery | Vendor-managed messaging interface | SaaS Vendor |
| EXT-004 | Clinic identity/staff roster process | Staff account provisioning, MFA enrollment, and role assignment in SaaS native accounts | Manual/admin configuration or vendor-supported import TBD | Clinic |

## 5. Technology Stack

| Layer | Technology | Version/Constraint | Rationale |
|---|---|---|---|
| Frontend | Vendor-hosted responsive web UI from HIPAA-ready scheduling SaaS | Must support mobile-first patient flows and WCAG 2.1 AA validation | Meets PRD-NFR-004 while avoiding custom frontend build burden |
| Backend | Vendor-hosted scheduling SaaS application services | Must support BAA, HIPAA security controls, OTP, scheduling rules, reminders, audit logs, and EHR integration | Delivers core scheduling capabilities with minimal custom code |
| Data | Existing EHR/practice management system plus SaaS operational data store | EHR remains system of record; SaaS stores operational copies only while needed for active scheduling | Preserves existing clinical scheduling source of truth and limits duplicated PHI |
| Hosting | Vendor-hosted HIPAA-ready SaaS under signed BAA | Vendor owns hosting, releases, rollback, and infrastructure controls | Reduces clinic operations burden and supports HIPAA requirements |
| Integration | Real-time API integration where available | Exact EHR integration method is TBD pending EHR capability confirmation | Supports timely availability, booking, rescheduling, and schedule view sync |
| Messaging | SaaS reminder service under BAA | SMS and email content must exclude PHI beyond clinic name and use secure links | Supports PRD-FR-004 and PRD-NFR-002 |

## 6. Logical Architecture

### 6.1 Components

#### COMP-001 — Patient Scheduling Web Experience

- **Responsibility:** Provides mobile-first booking, appointment request, rescheduling, and secure reminder-link flows for patients.
- **Inputs:** Appointment type selections, desired time slots, patient identity/contact information, OTP submissions, secure reminder-link tokens.
- **Outputs:** Booking confirmations, pending request statuses, reschedule confirmations, secure appointment detail views after verification.
- **Dependencies:** COMP-002, COMP-003, COMP-004, COMP-005, EXT-002.
- **Repository Area:** `vendor-managed/TBD`

#### COMP-002 — Scheduling Orchestration Service

- **Responsibility:** Applies booking rules, appointment type configuration, cutoff rules, OTP verification status, and EHR synchronization workflow.
- **Inputs:** Patient scheduling actions, staff/admin configuration, EHR availability and appointment data, reminder workflow triggers.
- **Outputs:** Confirmed bookings, pending requests, rescheduled appointments, sync updates, reminder jobs, audit events.
- **Dependencies:** COMP-003, COMP-004, COMP-005, COMP-006, EXT-001, EXT-002.
- **Repository Area:** `vendor-managed/TBD`

#### COMP-003 — EHR Integration Adapter

- **Responsibility:** Integrates with the EHR/practice management scheduling system for availability, booking, rescheduling, and schedule synchronization.
- **Inputs:** Availability queries, booking/rescheduling commands, appointment updates, reconciliation requests.
- **Outputs:** Availability results, appointment IDs/statuses, sync results, integration failure events.
- **Dependencies:** EXT-001, EXT-002.
- **Repository Area:** `vendor-managed/TBD`

#### COMP-004 — Staff Schedule and Administration Experience

- **Responsibility:** Gives staff a daily schedule view and allows authorized configuration of direct-bookable versus staff-confirmed appointment types.
- **Inputs:** Staff credentials, role assignments, schedule date/provider filters, appointment type configuration changes.
- **Outputs:** Daily schedule view, pending request visibility, failed reminder delivery visibility, configuration audit events.
- **Dependencies:** COMP-002, COMP-005, COMP-006, EXT-002, EXT-004.
- **Repository Area:** `vendor-managed/TBD`

#### COMP-005 — Reminder and OTP Service

- **Responsibility:** Sends SMS/email reminders and OTPs, enforces privacy-safe templates, and routes appointment details through secure links.
- **Inputs:** Upcoming appointment events, patient contact information, consent/eligibility indicators where applicable, reminder templates, OTP requests.
- **Outputs:** SMS/email reminders, OTP messages, delivery statuses, failed delivery events, secure link tokens.
- **Dependencies:** COMP-002, EXT-003.
- **Repository Area:** `vendor-managed/TBD`

#### COMP-006 — Audit, Reporting, and Observability Dashboard

- **Responsibility:** Provides audit evidence, operational reports, sync failure visibility, reminder failure visibility, and access event exports.
- **Inputs:** Access events, scheduling modifications, configuration changes, integration results, reminder delivery statuses.
- **Outputs:** Audit logs, exportable reports, vendor dashboard views, operational alerts or notifications.
- **Dependencies:** EXT-002, EXT-003.
- **Repository Area:** `vendor-managed/TBD`

### 6.2 Data Flow

1. Patient sends appointment type, identity/contact data, and desired slot information to COMP-001.
2. COMP-001 sends the scheduling request to COMP-002.
3. COMP-002 checks appointment type rules from COMP-004 configuration.
4. For direct-bookable appointment types, COMP-002 queries availability through COMP-003 against EXT-001.
5. EXT-001 returns availability to COMP-003, which returns normalized availability to COMP-002 and COMP-001.
6. Patient confirms a selected slot through COMP-001, and COMP-002 writes the booking through COMP-003 to EXT-001.
7. EXT-001 returns booking status; COMP-002 updates SaaS operational copies and emits audit events to COMP-006.
8. For staff-confirmed appointment types, COMP-002 creates a pending request and exposes it in COMP-004 until staff confirms or declines it.
9. For rescheduling, COMP-001 requests OTP verification through COMP-005 before COMP-002 exposes eligible appointment actions.
10. COMP-005 sends OTP by SMS or email through EXT-003 and returns verification status to COMP-002.
11. COMP-002 applies the configurable cutoff, performs rescheduling through COMP-003 and EXT-001, and updates COMP-004.
12. COMP-005 sends SMS/email reminders with clinic name and secure links only, records delivery status, and exposes failures through COMP-006 and COMP-004.
13. Staff opens COMP-004 to view confirmed appointments and pending requests grouped by provider and time, using data synchronized from EXT-001 and SaaS operational state.

## 7. Architecture Decisions

### ARD-DEC-001 — Use HIPAA-Ready Scheduling SaaS With Minimal Custom Code

- **Status:** Accepted
- **Related Requirements:** PRD-FR-001, PRD-FR-002, PRD-FR-003, PRD-FR-004, PRD-FR-005, PRD-FR-006, PRD-NFR-001, PRD-NFR-004

**Decision**

The first release will configure and integrate a HIPAA-ready scheduling SaaS rather than building a fully custom scheduling application.

**Rationale**

The clinic is a small business and needs online booking, rescheduling, reminders, staff schedule visibility, HIPAA controls, and EHR integration with a low operational burden.

**Consequences**

- Reduces custom development, hosting, release, and infrastructure responsibilities for the clinic.
- Makes vendor BAA, security evidence, integration capabilities, and configurability critical selection criteria.
- Limits architecture flexibility to vendor-supported workflow and integration patterns.

**Alternatives Considered**

- Custom web application integrated with EHR — Not selected because it increases delivery, security, compliance, and operations burden.
- Standalone scheduling tool without EHR integration — Not selected because CON-003 requires EHR/practice management integration.
- Full patient portal replacement — Not selected because the PRD explicitly excludes replacing the full patient portal.

### ARD-DEC-002 — Keep EHR/Practice Management System as System of Record

- **Status:** Accepted
- **Related Requirements:** PRD-FR-001, PRD-FR-003, PRD-FR-005, PRD-NFR-001

**Decision**

The existing EHR/practice management scheduling system remains the system of record for appointments and availability. The scheduling SaaS stores operational copies only as needed for active scheduling workflows and staff visibility.

**Rationale**

The clinic already relies on the EHR/practice management system for scheduling records, and the PRD requires integration rather than replacement.

**Consequences**

- Reduces risk of conflicting appointment records.
- Requires robust EHR integration and reconciliation behavior.
- Availability and performance may depend on EHR API capabilities.

**Alternatives Considered**

- Scheduling SaaS as system of record — Not selected because it would shift core scheduling authority away from the existing EHR/practice management system.
- Dual-write between SaaS and EHR — Not selected because it increases conflict and reconciliation complexity.

### ARD-DEC-003 — Prefer Real-Time EHR Integration

- **Status:** Accepted
- **Related Requirements:** PRD-FR-001, PRD-FR-003, PRD-FR-005

**Decision**

If the EHR/practice management system supports multiple integration methods, the architecture will use real-time API integration for availability, booking, rescheduling, and schedule view synchronization.

**Rationale**

Direct online booking and timely staff schedule visibility require current availability and prompt updates after patient and staff actions.

**Consequences**

- Improves booking accuracy and staff schedule freshness.
- Requires API capability, authentication setup, error handling, and integration monitoring.
- If real-time APIs are unavailable, the integration approach remains an open architecture question.

**Alternatives Considered**

- Batch file import/export — Not selected as primary because it may not support timely availability and schedule updates.
- Manual reconciliation — Not selected because it undermines online booking and staff workload reduction goals.

### ARD-DEC-004 — Use Vendor-Hosted SaaS Under Signed BAA

- **Status:** Accepted
- **Related Requirements:** PRD-NFR-001, PRD-NFR-002, PRD-NFR-003

**Decision**

The scheduling platform will be vendor-hosted as a HIPAA-ready SaaS under a signed BAA. The vendor owns SaaS hosting, release, rollback, and infrastructure operations.

**Rationale**

Vendor-hosted SaaS aligns with the small-business constraint and allows the clinic to rely on vendor-managed infrastructure controls, provided HIPAA security evidence is acceptable.

**Consequences**

- Vendor due diligence becomes a launch gate.
- Clinic must manage configuration change control and acceptance testing.
- Infrastructure details may be abstracted by vendor documentation rather than directly controlled by the clinic.

**Alternatives Considered**

- Clinic-owned cloud hosting — Not selected because it increases operational complexity.
- On-premises hosting — Not selected because it is inconsistent with the desired low-burden SaaS model.

### ARD-DEC-005 — Use SaaS Native Staff Accounts With MFA and RBAC

- **Status:** Accepted
- **Related Requirements:** PRD-FR-005, PRD-FR-006, PRD-NFR-001

**Decision**

Staff and administrator access will use scheduling SaaS native accounts with MFA and role-based access control.

**Rationale**

This approach avoids SSO implementation complexity during the first pilot while still requiring individual authentication, MFA, and role separation.

**Consequences**

- Staff provisioning and deprovisioning must be operationally controlled by the clinic.
- Shared staff accounts are prohibited.
- Future SSO may be considered after pilot if supported by vendor and clinic identity tooling.

**Alternatives Considered**

- SSO with clinic identity provider — Not selected for pilot because identity provider availability is not established.
- Shared staff login — Not selected because it weakens auditability and HIPAA access controls.

### ARD-DEC-006 — Deliver Reminders Through SaaS Reminder Service

- **Status:** Accepted
- **Related Requirements:** PRD-FR-004, PRD-NFR-001, PRD-NFR-002, PRD-NFR-003

**Decision**

SMS and email reminders and OTPs will be delivered through the HIPAA-ready scheduling SaaS reminder service under BAA.

**Rationale**

Using the SaaS reminder service reduces integration complexity and centralizes privacy-safe templates, secure links, delivery status, and audit evidence.

**Consequences**

- Vendor reminder content controls and delivery reporting must be validated.
- Reminder messages must exclude PHI beyond clinic name and include secure links only.
- Failed delivery visibility is dependent on vendor dashboard/reporting capabilities.

**Alternatives Considered**

- Separate SMS and email providers — Not selected for first release because it adds vendor and integration complexity.
- Clinic existing messaging tools — Not selected because BAA, audit logging, and privacy-safe template enforcement are not confirmed.

### ARD-DEC-007 — No Bulk Data Migration for Pilot

- **Status:** Accepted
- **Related Requirements:** PRD-FR-001, PRD-FR-003, PRD-FR-005, PRD-NFR-001

**Decision**

The pilot will not perform a bulk migration of scheduling history or patient records into the SaaS. The SaaS will read/sync appointments and availability from the EHR as needed for pilot workflows.

**Rationale**

The EHR remains the system of record and the pilot scope should minimize duplicated PHI and migration risk.

**Consequences**

- Reduces migration complexity and PHI duplication.
- Requires integration availability for active workflows.
- Historical scheduling analytics in the SaaS may be limited during pilot.

**Alternatives Considered**

- Full scheduling history migration — Not selected because it is unnecessary for pilot workflows and increases PHI exposure.
- One-time import of scheduling data — Not selected unless later required by the selected vendor/EHR integration approach.

## 8. API and Integration Architecture

### ARD-API-001 — EHR Availability Query

- **Type:** REST
- **Operation/Topic:** `GET /availability` or EHR/vendor equivalent TBD
- **Provider:** EXT-001
- **Consumers:** COMP-003, COMP-002
- **Authentication:** EHR/vendor-supported secure service authentication TBD
- **Versioning:** EHR/vendor API versioning TBD; changes must be reviewed before pilot release

**Request/Input Contract**

Appointment type, provider/location where applicable, date/time range, and eligibility filters needed to return available slots. Exact EHR contract is TBD pending EHR integration confirmation.

**Response/Output Contract**

Available appointment slots, provider/time metadata where available, and status/error details. Exact EHR contract is TBD pending EHR integration confirmation.

**Error Handling**

Transient API failures must be retried according to vendor-supported policy. Patient-facing flows must not create bookings when availability cannot be confirmed. Failures must be logged and visible through operational reporting.

### ARD-API-002 — EHR Booking and Rescheduling Command

- **Type:** REST
- **Operation/Topic:** `POST /appointments`, `PATCH /appointments/{id}` or EHR/vendor equivalent TBD
- **Provider:** EXT-001
- **Consumers:** COMP-003, COMP-002
- **Authentication:** EHR/vendor-supported secure service authentication TBD
- **Versioning:** EHR/vendor API versioning TBD; changes must be reviewed before pilot release

**Request/Input Contract**

Patient identity/contact data required for scheduling, appointment type, selected slot, booking/rescheduling action, source channel, and idempotency/correlation identifier where supported. Exact EHR contract is TBD pending EHR integration confirmation.

**Response/Output Contract**

Appointment identifier, confirmation or pending status, updated appointment time, failure reason where applicable, and correlation identifier where supported. Exact EHR contract is TBD pending EHR integration confirmation.

**Error Handling**

Booking and rescheduling commands must be idempotent where supported or protected from duplicate submission by the SaaS workflow. Conflicts must result in no confirmed booking and must prompt the patient to select another available slot or contact the clinic. Integration failures must be logged for staff/vendor follow-up.

### ARD-API-003 — Schedule Synchronization and Staff View Feed

- **Type:** REST
- **Operation/Topic:** `GET /schedule?date={date}` or EHR/vendor equivalent TBD
- **Provider:** EXT-001 and COMP-002
- **Consumers:** COMP-004
- **Authentication:** SaaS staff authentication with backend service authentication to EHR
- **Versioning:** Vendor/EHR versioning TBD

**Request/Input Contract**

Date, provider, location, schedule status filters, and staff role context.

**Response/Output Contract**

Confirmed appointments and pending requests grouped or groupable by provider and time, with status metadata sufficient to distinguish confirmed versus pending items.

**Error Handling**

If synchronization fails, staff must see an operational indication or report of sync failure. The system must avoid displaying stale data as current without a freshness indicator where vendor capability supports it.

### ARD-API-004 — Reminder and OTP Delivery

- **Type:** REST
- **Operation/Topic:** `POST /messages` or vendor-managed reminder workflow TBD
- **Provider:** COMP-005 / EXT-003
- **Consumers:** COMP-001, COMP-002
- **Authentication:** Vendor-managed service authentication
- **Versioning:** Vendor-managed API/workflow versioning TBD

**Request/Input Contract**

Recipient contact channel, purpose (reminder or OTP), approved template identifier, secure link token where applicable, and appointment reference without PHI in message body.

**Response/Output Contract**

Delivery accepted/rejected status, delivery attempt status where available, failure reason where available, and audit/correlation identifier.

**Error Handling**

Failed delivery attempts must be visible to staff or administrators. OTP failures must not expose appointment details or allow rescheduling. Reminder retries follow vendor-supported policy and must avoid duplicate or misleading reminders where possible.

### ARD-API-005 — Audit and Operational Report Export

- **Type:** File
- **Operation/Topic:** `audit/report export` vendor capability TBD
- **Provider:** COMP-006
- **Consumers:** Clinic Technical/Operations Owner, Scheduling Staff, Security/Compliance Reviewer
- **Authentication:** SaaS staff/admin MFA and RBAC
- **Versioning:** Vendor report schema/version TBD

**Request/Input Contract**

Report type, date/time range, event types, and authorized user role.

**Response/Output Contract**

Exportable audit and operational reports for access events, scheduling changes, configuration changes, sync failures, and reminder failures.

**Error Handling**

Failed exports must be retryable or requestable from vendor support. Audit/report export limitations must be documented before pilot approval.

## 9. Data Architecture

### DATA-001 — Appointment

- **Purpose:** Represents confirmed clinic appointments used for booking, rescheduling, reminders, and staff schedule visibility.
- **System of Record:** Existing EHR/practice management scheduling system.
- **Classification:** Restricted
- **Retention:** EHR retention policy TBD; SaaS operational copies retained only while needed for active scheduling, with exact retention TBD under compliance/security policy.
- **Key Relationships:** Patient, provider, appointment type, location, reminder events, audit events.

### DATA-002 — Pending Appointment Request

- **Purpose:** Represents staff-confirmed appointment requests submitted online before confirmation or decline.
- **System of Record:** Scheduling SaaS operational store until confirmed/declined, then synchronized to EHR where applicable.
- **Classification:** Restricted
- **Retention:** Retained only while needed for active scheduling workflow and audit evidence; exact retention TBD under compliance/security policy.
- **Key Relationships:** Patient identity/contact data, requested appointment type, preferred time/provider where applicable, staff action, audit events.

### DATA-003 — Patient Identity and Contact Data

- **Purpose:** Supports online booking, OTP verification, reminders, and clinic follow-up.
- **System of Record:** Existing EHR/practice management system where matched or created; SaaS stores operational copies needed for active workflows.
- **Classification:** Restricted
- **Retention:** EHR retention policy TBD; SaaS operational copies retained only while needed for active scheduling and reminder workflows; exact retention TBD.
- **Key Relationships:** Appointment, pending request, OTP verification session, reminder delivery events.

### DATA-004 — Appointment Type Configuration

- **Purpose:** Defines whether each appointment type is direct-bookable or staff-confirmed.
- **System of Record:** Scheduling SaaS configuration for online scheduling behavior, based on clinic-approved appointment type list.
- **Classification:** Internal
- **Retention:** Retained while configuration is active and as required for configuration audit evidence; exact audit retention TBD.
- **Key Relationships:** Appointment type, scheduling rule, admin user, audit event.

### DATA-005 — Reminder and OTP Event

- **Purpose:** Tracks reminder and OTP delivery attempts, verification status, failures, and secure-link access controls.
- **System of Record:** Scheduling SaaS reminder service.
- **Classification:** Restricted
- **Retention:** Retained according to vendor-supported audit and operational retention policy; exact retention TBD.
- **Key Relationships:** Appointment, patient contact channel, secure link token, delivery status, audit event.

### DATA-006 — Audit Event

- **Purpose:** Provides evidence of access, modification, configuration changes, sync failures, and reminder failures.
- **System of Record:** Scheduling SaaS audit/reporting subsystem.
- **Classification:** Confidential
- **Retention:** Retained per HIPAA/security policy TBD and vendor capability; exact duration TBD.
- **Key Relationships:** Staff/admin user, patient verification session, appointment, pending request, configuration change, integration event.

### Data Migration

- **Required:** No
- **Approach:** No bulk migration for the pilot. The SaaS will read/sync appointments and availability from the EHR/practice management system as needed for pilot workflows. Any vendor-required initial configuration import for appointment types, providers, or availability rules is treated as configuration setup, not bulk clinical data migration.

## 10. Security Architecture

### ARD-SEC-001 — HIPAA-Compliant PHI Handling

- **Related Requirements:** PRD-NFR-001, PRD-NFR-002, PRD-NFR-003

The implementation must:

- Operate under a signed BAA with the scheduling SaaS vendor before any PHI is processed.
- Protect patient scheduling data in transit and at rest.
- Limit staff access to patient scheduling data through individual accounts, MFA, and role-based access control.
- Require OTP verification before exposing existing appointment details through patient rescheduling or secure reminder-link flows.
- Maintain audit evidence for access to and modification of scheduling records.
- Minimize PHI in reminder content, logs, operational copies, and exports.

The implementation must not:

- Use shared staff accounts for access to PHI or scheduling configuration.
- Include appointment type, diagnosis, treatment details, or other detailed PHI in SMS or email reminder message bodies.
- Expose existing appointment details before successful OTP verification.
- Store bulk scheduling history in the SaaS for the pilot unless separately approved.

### ARD-SEC-002 — Vendor Security Assurance Gate

- **Related Requirements:** PRD-NFR-001

The implementation must:

- Obtain signed BAA, HIPAA security documentation, annual third-party security assessment evidence, CVE management process evidence, and SBOM or equivalent component inventory before pilot approval.
- Verify that vendor-hosted services support encryption, audit logging, access controls, secure deletion/retention controls, and operational reporting appropriate for HIPAA-covered scheduling data.
- Block pilot launch when known Critical or High security findings affect in-scope production workflows unless formally risk accepted by the clinic.

The implementation must not:

- Process PHI through vendors or sub-processors without acceptable compliance and security documentation.
- Launch the pilot with unresolved Critical or High security findings unless formally risk accepted.

### Security Controls

- **Authentication:** Patient rescheduling and secure reminder-link detail views require OTP verification by SMS or email; staff use SaaS native individual accounts with MFA.
- **Authorization:** SaaS RBAC limits staff schedule access, failed reminder visibility, and appointment type configuration to authorized roles.
- **Secrets Management:** Vendor manages SaaS platform secrets; EHR integration credentials must be stored in vendor-supported secure secret storage and must not be exposed to clinic users or logs.
- **Encryption:** TLS is required for data in transit; vendor-managed encryption at rest is required for PHI and audit data.
- **Audit Logging:** Access to scheduling records, booking/rescheduling actions, pending request actions, reminder failures, OTP/security events, and configuration changes must be auditable and exportable where vendor supports export.
- **Dependency/CVE Scanning:** Vendor must provide evidence of CVE management and must address Critical and High findings affecting in-scope production workflows before pilot launch or obtain formal risk acceptance.
- **SBOM:** Required as an SBOM or equivalent component inventory from the SaaS vendor.

## 11. Reliability, Performance, and Scalability

### ARD-OPS-001 — SaaS Availability

- **Target:** Vendor-provided uptime commitment is TBD and required before pilot approval.
- **Validation Method:** Review vendor SLA, status history where available, incident process documentation, and pilot operational reports.

### ARD-OPS-002 — Patient Page Performance

- **Target:** Vendor-provided patient-facing response time commitment is TBD and required before pilot approval.
- **Validation Method:** Vendor performance documentation plus pilot validation of booking, rescheduling, and secure-link flows on mobile and desktop browsers.

### ARD-OPS-003 — EHR Synchronization Latency

- **Target:** Vendor/EHR sync latency commitment is TBD and required before pilot approval.
- **Validation Method:** Integration testing for availability lookup, booking confirmation, rescheduling update, and staff schedule refresh flows.

### ARD-OPS-004 — Reminder Delivery Reliability

- **Target:** Vendor-provided SMS/email delivery reporting and delivery commitment are TBD and required before pilot approval.
- **Validation Method:** Reminder delivery tests, failed delivery report review, and pilot monitoring of SMS/email delivery statuses.

### ARD-OPS-005 — Operational Recovery and Rollback

- **Target:** Vendor rollback and incident recovery commitments are TBD and required before pilot approval.
- **Validation Method:** Review vendor rollback process, incident response procedure, and support escalation path before launch.

## 12. Observability

- **Logging:** Vendor must provide audit and operational logging for access events, scheduling changes, configuration changes, EHR sync failures, OTP events, and reminder delivery failures.
- **Metrics:** Vendor dashboard or reports must expose sync failures, reminder failures, access/security events, and scheduling workflow outcomes needed for pilot monitoring; exact metrics are TBD pending vendor selection.
- **Tracing:** End-to-end technical tracing across SaaS and EHR integration is vendor-dependent and TBD; at minimum, correlation identifiers or reportable event references are required for support investigation where supported.
- **Alerting:** Vendor dashboard and exportable operational reports are required for sync failures, reminder failures, and access events; real-time alerting method is TBD pending vendor capability.
- **Sensitive Data Restrictions:** Logs, metrics, traces, alerts, exports, SMS bodies, and email bodies must minimize PHI and must not include appointment type, diagnosis, treatment details, or other detailed PHI in reminder content.

## 13. Deployment Architecture

- **Environments:** Vendor-provided environments are TBD; pilot requires at least a production environment and a non-production/test or vendor-supported validation path before production configuration changes.
- **Deployment Model:** Vendor-hosted HIPAA-ready SaaS under signed BAA.
- **CI/CD:** Vendor owns SaaS release pipelines; clinic owns configuration change control and acceptance testing for scheduling rules, reminder templates, EHR integration behavior, and pilot readiness.
- **Configuration Management:** Appointment type settings, rescheduling cutoff rules, reminder templates, staff roles, and EHR integration configuration must be controlled through authorized SaaS administration with auditable changes.
- **Rollback:** Vendor owns SaaS platform rollback; clinic configuration rollback must use vendor-supported configuration history, backup/export, or documented manual rollback steps TBD.
- **Infrastructure as Code:** Vendor-owned and not directly managed by the clinic; vendor infrastructure control evidence is required through security/compliance documentation.

## 14. Testing Architecture

Required test levels:

- Unit tests
- Integration tests
- Contract tests
- End-to-end tests
- Security tests
- Performance tests, where applicable

Minimum quality expectations:

- **Coverage Threshold:** Policy-based threshold: 100% acceptance coverage for Must-priority PRD requirements in clinic/vendor acceptance testing; vendor internal unit/code coverage percentage is TBD and may be evidenced through vendor quality documentation where available.
- **Blocking Security Severity:** Known Critical or High security findings affecting in-scope production workflows block pilot launch unless formally risk accepted by the clinic.
- **Required Build/Lint Checks:** Vendor standard build, lint, automated test, dependency/CVE scanning, and release checks must be evidenced through vendor assurance documentation; clinic configuration changes require peer review or authorized approval plus successful acceptance tests.

Pilot launch is blocked until acceptance tests for all Must requirements, EHR integration tests, reminder delivery tests, WCAG 2.1 AA checks, HIPAA/privacy review, and Critical/High security finding review are complete.

## 15. Architecture Constraints

- ARC-CON-001 — The architecture must satisfy HIPAA obligations for PHI handled in patient scheduling workflows.
- ARC-CON-002 — The architecture must use a small-business-appropriate operating model with minimal custom code and vendor-managed SaaS operations where feasible.
- ARC-CON-003 — The first release must integrate with the existing EHR/practice management scheduling system.
- ARC-CON-004 — The existing EHR/practice management system remains the system of record for appointments and availability.
- ARC-CON-005 — SMS and email reminder bodies must not expose PHI beyond clinic name and must route appointment details through secure links.
- ARC-CON-006 — Patient rescheduling and secure reminder-link detail access require OTP verification by SMS or email.
- ARC-CON-007 — The first launch is a pilot within one clinic location; pilot date is TBD.
- ARC-CON-008 — The vendor-hosted SaaS must operate under a signed BAA before processing PHI.
- ARC-CON-009 — Staff access must use individual accounts, MFA, RBAC, and auditable activity.
- ARC-CON-010 — No bulk scheduling history migration is planned for the pilot.

## 16. Risks and Trade-offs

| ID | Risk/Trade-off | Impact | Mitigation |
|---|---|---|---|
| ARC-RISK-001 | EHR/practice management integration capabilities may not support real-time availability, booking, and rescheduling | Could force batch/manual workarounds and weaken patient/staff experience | Confirm EHR integration method before implementation commitment and document fallback before pilot approval |
| ARC-RISK-002 | Vendor SaaS workflow may not exactly match clinic scheduling rules | Could require process changes or scope reduction | Validate appointment type configuration, cutoff rules, pending requests, and staff view during vendor selection and acceptance testing |
| ARC-RISK-003 | Reminder or secure-link configuration could accidentally expose PHI | High compliance/privacy impact | Lock templates, require HIPAA/privacy review, validate messages, and route details only after OTP verification |
| ARC-RISK-004 | Vendor security evidence may be insufficient for HIPAA risk expectations | Pilot may be delayed or vendor rejected | Require BAA, HIPAA documentation, third-party assessment, CVE process, and SBOM/equivalent inventory before approval |
| ARC-RISK-005 | SaaS operational copies introduce duplicated PHI outside the EHR | Increases privacy, retention, and breach scope | Minimize operational copies, avoid bulk migration, define retention/deletion policy, and audit vendor storage controls |
| ARC-RISK-006 | Vendor-hosted model limits clinic control over CI/CD, infrastructure, and detailed observability | Some controls depend on vendor transparency | Use contractual/security review gates, vendor reports, operational dashboards, and acceptance testing |
| ARC-RISK-007 | Staff provisioning/deprovisioning in native SaaS accounts may be missed | Former staff could retain inappropriate access | Establish clinic-owned account lifecycle process with MFA/RBAC review before pilot |
| ARC-RISK-008 | Success and reliability targets remain TBD until baseline/vendor commitments are available | Pilot readiness criteria may be incomplete | Track open questions with owners and require targets/commitments before pilot approval |

## 17. Open Architecture Questions

| ID | Question | Owner | Status | Decision/Due Date |
|---|---|---|---|---|
| ARC-OQ-001 | Who is the named Architecture Owner for this ARD? | Business Sponsor | Open | TBD |
| ARC-OQ-002 | Who are the named ARD contributors and approval reviewers? | Business Sponsor | Open | TBD |
| ARC-OQ-003 | What existing EHR/practice management system will be integrated and what integration method is available? | Technical Owner | Open | TBD |
| ARC-OQ-004 | What exact EHR API/file contracts, authentication method, versioning strategy, and rate limits apply? | Technical Owner | Open | Before implementation commitment |
| ARC-OQ-005 | What is the vendor-provided uptime SLA for the pilot and production service? | SaaS Vendor | Open | Before pilot approval |
| ARC-OQ-006 | What is the vendor-provided patient-facing response time target? | SaaS Vendor | Open | Before pilot approval |
| ARC-OQ-007 | What is the vendor/EHR synchronization latency target for booking, rescheduling, and staff schedule view updates? | SaaS Vendor / Technical Owner | Open | Before pilot approval |
| ARC-OQ-008 | What is the exact retention/deletion policy for SaaS operational copies, reminder/OTP events, and audit logs? | Security/Compliance Reviewer | Open | Before pilot approval |
| ARC-OQ-009 | What vendor environments are available for non-production testing and acceptance validation? | SaaS Vendor | Open | Before pilot approval |
| ARC-OQ-010 | What vendor-supported rollback approach exists for platform releases and clinic configuration changes? | SaaS Vendor | Open | Before pilot approval |
| ARC-OQ-011 | What exact real-time alerting, reporting exports, and correlation identifiers does the SaaS provide? | SaaS Vendor | Open | Before pilot approval |
| ARC-OQ-012 | What pilot launch date and clinic location will be used? | Business Sponsor | Open | TBD |
| ARC-OQ-013 | What exact appointment types are direct-bookable versus staff-confirmed at pilot launch? | Clinic Administrator | Open | Before pilot approval |
| ARC-OQ-014 | What is the configurable cutoff for online rescheduling before appointment time? | Clinic Administrator | Open | Before pilot approval |
| ARC-OQ-015 | What baseline and pilot targets will be set for KPI-001 through KPI-004? | Product Owner | Open | Before pilot approval |

## 18. Feature Mapping

| Feature | Relevant Components | Decision IDs | Interface IDs | Security IDs |
|---|---|---|---|---|
| FEAT-001 | COMP-001, COMP-002, COMP-003, COMP-004, COMP-006 | ARD-DEC-001, ARD-DEC-002, ARD-DEC-003, ARD-DEC-004 | ARD-API-001, ARD-API-002, ARD-API-003, ARD-API-005 | ARD-SEC-001, ARD-SEC-002 |
| FEAT-002 | COMP-001, COMP-002, COMP-004, COMP-006 | ARD-DEC-001, ARD-DEC-002, ARD-DEC-004 | ARD-API-003, ARD-API-005 | ARD-SEC-001, ARD-SEC-002 |
| FEAT-003 | COMP-001, COMP-002, COMP-003, COMP-005, COMP-006 | ARD-DEC-001, ARD-DEC-002, ARD-DEC-003, ARD-DEC-004, ARD-DEC-006 | ARD-API-001, ARD-API-002, ARD-API-004, ARD-API-005 | ARD-SEC-001, ARD-SEC-002 |
| FEAT-004 | COMP-001, COMP-002, COMP-005, COMP-006 | ARD-DEC-001, ARD-DEC-004, ARD-DEC-006 | ARD-API-004, ARD-API-005 | ARD-SEC-001, ARD-SEC-002 |
| FEAT-005 | COMP-002, COMP-003, COMP-004, COMP-006 | ARD-DEC-001, ARD-DEC-002, ARD-DEC-003, ARD-DEC-005 | ARD-API-003, ARD-API-005 | ARD-SEC-001, ARD-SEC-002 |
| FEAT-006 | COMP-002, COMP-004, COMP-006 | ARD-DEC-001, ARD-DEC-004, ARD-DEC-005 | ARD-API-005 | ARD-SEC-001, ARD-SEC-002 |

## 19. Approval

| Role | Name | Decision | Date |
|---|---|---|---|
| Solution Architect | TBD | Pending | TBD |
| Security Reviewer | TBD | Pending | TBD |
| Engineering Lead | TBD | Pending | TBD |