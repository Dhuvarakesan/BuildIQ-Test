# Product Requirements Document (PRD)

> Requirement IDs must remain stable after approval so downstream ARD and feature specifications retain traceability.

## Document Metadata

- **Product/Application:** Online Patient Scheduling and Reminders
- **Document ID:** PRD-OPSR
- **Version:** 0.1
- **Status:** Draft
- **Owner:** TBD
- **Contributors:** TBD
- **Last Updated:** 2026-11-23

---

## 1. Executive Summary

The clinic needs an online scheduling experience because patients can currently book only by phone during office hours, causing lost bookings when patients try to schedule outside those hours. The first release will let new and existing patients book appointments online, reschedule eligible appointments, and receive automated reminders.

The product will support direct booking for clinic-selected low-risk appointment types, while appointment types that require review will create pending requests for staff confirmation. Staff will also get a single daily schedule view that shows appointments and pending requests grouped by provider and time.

Because this is a healthcare product for a HIPAA-covered clinic, the release must protect patient information, limit reminder message content, support identity verification for rescheduling, and integrate with the existing EHR/practice management scheduling system.

## 2. Problem Statement

Patients who want to book outside office hours cannot complete scheduling online today. This creates missed booking opportunities for the clinic and forces staff to handle scheduling manually by phone. Staff also need one consolidated place to view the day's schedule and pending scheduling work.

The current evidence baseline will be established by tracking after-hours missed calls and voicemail booking requests before launch.

## 3. Product Vision

Patients can book and reschedule eligible appointments online at any time, receive compliant reminders, and avoid unnecessary phone calls. Staff can manage the day's schedule and pending appointment requests from one reliable view, while the clinic maintains HIPAA-compliant handling of scheduling data.

## 4. Goals and Success Measures

### 4.1 Goals

- Reduce missed booking opportunities outside office hours.
- Enable new and existing patients to book online with basic identity and contact capture.
- Enable verified patients to reschedule eligible appointments online before a configurable cutoff.
- Reduce no-shows through automated SMS and email reminders.
- Give staff a single daily schedule view with appointments and pending requests grouped by provider and time.

### 4.2 Success Measures

| ID | Measure | Baseline | Target | Measurement Method |
|---|---|---:|---:|---|
| KPI-001 | After-hours missed booking opportunities | TBD | TBD | Count after-hours missed calls and voicemail booking requests during pre-launch baseline and compare to pilot period |
| KPI-002 | Online booking adoption | TBD | TBD | Count completed online bookings as a share of total bookings during the pilot |
| KPI-003 | No-show rate | TBD | TBD | Compare no-show rate before and after automated reminders launch |
| KPI-004 | Staff scheduling workload | TBD | TBD | Track scheduling-related phone calls and staff-confirmed pending requests during baseline and pilot |

## 5. Scope

### 5.1 In Scope

- Online appointment booking for new and existing patients.
- Direct online booking for clinic-selected low-risk appointment types.
- Online request submission for appointment types that require staff confirmation.
- Online rescheduling for eligible appointments after patient identity verification.
- Automated SMS and email appointment reminders.
- Staff daily schedule view showing appointments and pending requests grouped by provider and time.
- Integration with the existing EHR/practice management scheduling system.
- Pilot rollout within one clinic location.

### 5.2 Out of Scope

- Replacing the full patient portal.
- Clinical documentation features.
- Medical advice, triage, or diagnosis.
- Billing, insurance eligibility, or payment collection unless later approved.
- Multi-location scheduling optimization for the first release.

## 6. Personas and Users

### PERSONA-001 — Patient

- **Description:** New or existing patient who wants to book or reschedule a clinic appointment online.
- **Primary Need:** Complete scheduling tasks without calling during office hours.
- **Key Pain Point:** Phone-only scheduling prevents booking when the office is closed or staff are unavailable.

### PERSONA-002 — Scheduling Staff

- **Description:** Clinic staff responsible for reviewing appointment requests and managing the daily schedule.
- **Primary Need:** See appointments and pending requests in one daily view.
- **Key Pain Point:** Scheduling work is fragmented across phone calls, voicemails, and existing systems.

### PERSONA-003 — Clinic Administrator

- **Description:** Clinic operator responsible for configuring which appointment types can be booked directly and ensuring compliant operations.
- **Primary Need:** Control online booking rules for low-risk appointment types.
- **Key Pain Point:** Direct booking must be useful without allowing inappropriate appointment types to bypass staff review.

## 7. Assumptions and Dependencies

### Assumptions

- ASM-001 — The clinic will define which appointment types are considered low-risk before pilot launch (assumption).
- ASM-002 — Basic identity and contact capture is sufficient for new patient online booking (assumption).
- ASM-003 — One-time passcodes sent by SMS or email are acceptable for rescheduling identity verification (assumption).
- ASM-004 — Reminder messages will avoid PHI beyond clinic name and will use a secure link for appointment details (assumption).
- ASM-005 — Success targets will be set after pre-launch baseline measurement and before pilot approval (assumption).

### Dependencies

- DEP-001 — Existing EHR/practice management scheduling system integration.
- DEP-002 — SMS and email delivery services that support healthcare privacy and compliance needs.
- DEP-003 — Staff availability to review and confirm pending appointment requests.
- DEP-004 — Clinic approval of appointment types, rescheduling cutoff rules, and reminder templates.

## 8. Functional Requirements

### PRD-FR-001 — Online Booking for Low-Risk Appointment Types

- **Priority:** Must
- **Feature:** FEAT-001
- **Persona:** PERSONA-001

**Requirement**

The system shall allow new and existing patients to directly book available appointment slots online for clinic-selected low-risk appointment types.

**Business Rationale**

Direct online booking reduces missed booking opportunities outside office hours while limiting self-service to appointment types the clinic considers safe for direct scheduling.

**Acceptance Criteria**

1. A patient can select a clinic-approved low-risk appointment type, view available slots, enter required identity and contact information, and receive a booking confirmation.
2. Direct booking is available only for appointment types marked by the clinic as eligible for direct online booking.
3. Confirmed online bookings are reflected in the staff daily schedule view.
4. If no slots are available for an eligible appointment type, the system shows that no online slots are available and does not create a booking.

---

### PRD-FR-002 — Appointment Request for Staff-Confirmed Types

- **Priority:** Must
- **Feature:** FEAT-002
- **Persona:** PERSONA-001

**Requirement**

The system shall allow patients to submit appointment requests for appointment types that require staff confirmation and show the request as pending until staff confirms or declines it.

**Business Rationale**

Some appointment types should not be directly booked online, but patients should still be able to start scheduling outside office hours instead of being forced to call.

**Acceptance Criteria**

1. A patient can submit a request for an appointment type that requires staff confirmation.
2. The patient sees a pending status after submitting the request.
3. Pending requests appear in the staff daily schedule view grouped by provider and time when applicable.
4. Staff can distinguish pending requests from confirmed appointments.

---

### PRD-FR-003 — Online Rescheduling

- **Priority:** Must
- **Feature:** FEAT-003
- **Persona:** PERSONA-001

**Requirement**

The system shall allow patients to reschedule eligible appointments online after verifying their identity with a one-time passcode sent by SMS or email, subject to a configurable cutoff before the appointment time.

**Business Rationale**

Self-service rescheduling reduces phone calls and improves patient flexibility while preserving control over late changes.

**Acceptance Criteria**

1. A patient must complete one-time passcode verification before accessing rescheduling for an existing appointment.
2. A patient can reschedule an eligible appointment only before the clinic-configured cutoff.
3. The system prevents online rescheduling after the cutoff and directs the patient to contact the clinic.
4. A completed reschedule updates the staff daily schedule view.

---

### PRD-FR-004 — Automated Appointment Reminders

- **Priority:** Must
- **Feature:** FEAT-004
- **Persona:** PERSONA-001

**Requirement**

The system shall send automated appointment reminders by SMS and email using message content that excludes PHI beyond the clinic name and provides a secure link to view appointment details.

**Business Rationale**

Automated reminders are intended to reduce no-shows while limiting privacy exposure through SMS and email.

**Acceptance Criteria**

1. The system sends reminders through both SMS and email for appointments with valid patient contact details and required consent where applicable.
2. Reminder message bodies do not include appointment type, diagnosis, treatment details, or other PHI beyond the clinic name.
3. Reminder messages include a secure link for viewing appointment details.
4. Failed reminder delivery attempts are visible to staff or administrators for follow-up.

---

### PRD-FR-005 — Staff Daily Schedule View

- **Priority:** Must
- **Feature:** FEAT-005
- **Persona:** PERSONA-002

**Requirement**

The system shall provide staff with a daily schedule view that shows appointments and pending requests grouped by provider and time.

**Business Rationale**

Staff need one place to understand the day's confirmed schedule and pending scheduling work.

**Acceptance Criteria**

1. Staff can view the day's confirmed appointments grouped by provider and time.
2. Staff can view pending appointment requests in the same daily schedule experience.
3. Staff can visually distinguish confirmed appointments from pending requests.
4. Schedule changes from online booking or rescheduling appear in the staff view after synchronization with the scheduling system.

---

### PRD-FR-006 — Appointment Type Configuration

- **Priority:** Must
- **Feature:** FEAT-006
- **Persona:** PERSONA-003

**Requirement**

The system shall allow authorized clinic administrators or staff to configure which appointment types are eligible for direct online booking and which require staff confirmation.

**Business Rationale**

The clinic must control scheduling risk by deciding which appointment types patients may book directly.

**Acceptance Criteria**

1. An authorized user can mark an appointment type as direct-bookable or staff-confirmed.
2. Patient booking behavior changes according to the configured appointment type setting.
3. Unauthorized users cannot change appointment type booking settings.
4. Configuration changes are retained and applied to future booking sessions.

## 9. Non-Functional Requirements

### PRD-NFR-001 — HIPAA-Compliant PHI Handling

- **Category:** Compliance
- **Priority:** Must

**Requirement**

The system shall handle patient scheduling data in a manner consistent with HIPAA requirements for protected health information.

**Acceptance Criteria**

1. Patient scheduling data is protected in transit and at rest.
2. Access to patient scheduling data is limited to authorized users and verified patients.
3. Audit evidence is available for access to and modification of scheduling records.

### PRD-NFR-002 — Privacy-Safe Reminder Content

- **Category:** Privacy
- **Priority:** Must

**Requirement**

The system shall ensure SMS and email reminder content excludes PHI beyond the clinic name and routes appointment details through a secure link.

**Acceptance Criteria**

1. Reminder templates cannot include appointment type, diagnosis, treatment, or other detailed appointment information in the message body.
2. Appointment details are accessible only after the patient opens the secure link and completes required verification.

### PRD-NFR-003 — Identity Verification for Rescheduling

- **Category:** Security
- **Priority:** Must

**Requirement**

The system shall require one-time passcode verification by SMS or email before allowing a patient to reschedule an existing appointment.

**Acceptance Criteria**

1. A rescheduling session cannot proceed until a valid one-time passcode is entered.
2. Invalid or expired passcodes do not grant access to rescheduling.
3. The system does not expose existing appointment details before successful verification.

### PRD-NFR-004 — Accessibility and Responsive Design

- **Category:** Accessibility
- **Priority:** Must

**Requirement**

The patient-facing online scheduling experience shall be mobile-first, responsive, and meet WCAG 2.1 AA accessibility requirements.

**Acceptance Criteria**

1. Patient-facing booking, rescheduling, and reminder-link pages pass WCAG 2.1 AA accessibility validation before pilot launch.
2. Patient-facing flows are usable on mobile and desktop web browsers.

## 10. User Journeys

### JOURNEY-001 — Patient Books a Directly Bookable Appointment

1. Patient opens the online scheduling experience outside office hours.
2. Patient selects a clinic-approved low-risk appointment type.
3. Patient chooses an available slot and enters basic identity and contact information.
4. System confirms the appointment and updates the staff daily schedule.

**Related Requirements:** PRD-FR-001, PRD-FR-005, PRD-NFR-001, PRD-NFR-004

### JOURNEY-002 — Patient Requests a Staff-Confirmed Appointment

1. Patient selects an appointment type that requires staff confirmation.
2. Patient enters required identity, contact, and scheduling preference information.
3. System creates a pending request and shows pending status to the patient.
4. Staff sees the pending request in the daily schedule view.

**Related Requirements:** PRD-FR-002, PRD-FR-005, PRD-NFR-001, PRD-NFR-004

### JOURNEY-003 — Patient Reschedules an Eligible Appointment

1. Patient starts the rescheduling flow from the online experience or secure reminder link.
2. Patient verifies identity using a one-time passcode sent by SMS or email.
3. Patient selects a new eligible appointment slot before the configurable cutoff.
4. System confirms the reschedule and updates the staff daily schedule.

**Related Requirements:** PRD-FR-003, PRD-FR-005, PRD-NFR-003

### JOURNEY-004 — Patient Receives a Reminder

1. System identifies an upcoming appointment eligible for reminders.
2. System sends SMS and email reminders with clinic name and a secure link only.
3. Patient opens the secure link to view appointment details.
4. Staff can see failed reminder delivery attempts for follow-up.

**Related Requirements:** PRD-FR-004, PRD-NFR-001, PRD-NFR-002

### JOURNEY-005 — Staff Reviews the Day's Schedule

1. Staff opens the daily schedule view.
2. Staff sees confirmed appointments grouped by provider and time.
3. Staff sees pending requests in the same view.
4. Staff uses the view to understand confirmed and pending scheduling work for the day.

**Related Requirements:** PRD-FR-005, PRD-FR-006, PRD-NFR-001

## 11. Feature Breakdown

### FEAT-001 — Direct Online Booking

- **Summary:** Enables patients to book available slots online for clinic-selected low-risk appointment types.
- **Priority:** Must
- **Related Requirements:** PRD-FR-001, PRD-NFR-001, PRD-NFR-004
- **Expected Spec File:** `specs/FEAT-001-direct-online-booking.spec.md`

### FEAT-002 — Staff-Confirmed Appointment Requests

- **Summary:** Enables patients to submit pending appointment requests for appointment types that require staff confirmation.
- **Priority:** Must
- **Related Requirements:** PRD-FR-002, PRD-FR-005, PRD-NFR-001, PRD-NFR-004
- **Expected Spec File:** `specs/FEAT-002-staff-confirmed-appointment-requests.spec.md`

### FEAT-003 — Online Rescheduling

- **Summary:** Enables verified patients to reschedule eligible appointments before a configurable cutoff.
- **Priority:** Must
- **Related Requirements:** PRD-FR-003, PRD-NFR-003, PRD-NFR-004
- **Expected Spec File:** `specs/FEAT-003-online-rescheduling.spec.md`

### FEAT-004 — Automated Reminders

- **Summary:** Sends HIPAA-conscious SMS and email reminders with secure links to appointment details.
- **Priority:** Must
- **Related Requirements:** PRD-FR-004, PRD-NFR-001, PRD-NFR-002
- **Expected Spec File:** `specs/FEAT-004-automated-reminders.spec.md`

### FEAT-005 — Staff Daily Schedule View

- **Summary:** Gives staff one view of daily appointments and pending requests grouped by provider and time.
- **Priority:** Must
- **Related Requirements:** PRD-FR-005, PRD-NFR-001
- **Expected Spec File:** `specs/FEAT-005-staff-daily-schedule-view.spec.md`

### FEAT-006 — Scheduling Configuration

- **Summary:** Lets authorized clinic users configure direct-bookable and staff-confirmed appointment types.
- **Priority:** Must
- **Related Requirements:** PRD-FR-006, PRD-NFR-001
- **Expected Spec File:** `specs/FEAT-006-scheduling-configuration.spec.md`

## 12. Constraints

- CON-001 — The clinic operates in healthcare and must satisfy HIPAA requirements for protected health information.
- CON-002 — The company is a small business, so the first release should focus on the minimum viable scheduling and reminder workflows.
- CON-003 — The first release must integrate with the existing EHR/practice management scheduling system.
- CON-004 — Reminder messages must not expose PHI beyond clinic name and must use secure links for appointment details.
- CON-005 — Launch approach is a pilot within one clinic location; pilot date is TBD.

## 13. Risks

| ID | Risk | Impact | Likelihood | Mitigation |
|---|---|---|---|---|
| RISK-001 | EHR/practice management integration is more complex than expected | High | Medium | Confirm integration capabilities and data sync requirements before implementation commitment |
| RISK-002 | Reminder messages accidentally expose PHI | High | Medium | Enforce locked reminder templates, secure links, and compliance review before pilot |
| RISK-003 | Appointment type configuration is too broad and allows inappropriate direct booking | High | Medium | Require clinic approval of direct-bookable appointment types before launch |
| RISK-004 | Staff do not adopt pending request workflow | Medium | Medium | Include staff in pilot testing and ensure pending requests appear in the daily schedule view |
| RISK-005 | Success targets cannot be set without baseline data | Medium | Medium | Complete pre-launch baseline measurement before pilot approval |

## 14. Open Questions

| ID | Question | Owner | Status | Decision/Due Date |
|---|---|---|---|---|
| OQ-001 | What is the pre-launch baseline and pilot target for after-hours missed booking opportunities? | Product Owner | Open | Before pilot approval |
| OQ-002 | What is the pre-launch baseline and pilot target for online booking adoption? | Product Owner | Open | Before pilot approval |
| OQ-003 | What is the pre-launch baseline and pilot target for no-show rate reduction? | Product Owner | Open | Before pilot approval |
| OQ-004 | What is the pre-launch baseline and pilot target for staff scheduling workload reduction? | Product Owner | Open | Before pilot approval |
| OQ-005 | Which exact appointment types are direct-bookable versus staff-confirmed at pilot launch? | Clinic Administrator | Open | Before pilot approval |
| OQ-006 | What is the configurable cutoff for online rescheduling before appointment time? | Clinic Administrator | Open | Before pilot approval |
| OQ-007 | What existing EHR/practice management system will be integrated and what integration method is available? | Technical Owner | Open | TBD |
| OQ-008 | What is the pilot launch date for the first clinic location? | Business Sponsor | Open | TBD |
| OQ-009 | Who are the named Product Owner, Business Sponsor, and contributors for approval? | Business Sponsor | Open | TBD |

## 15. Release and Rollout Considerations

- **Target Release:** Pilot launch date TBD for one clinic location.
- **Rollout Approach:** Pilot within one clinic location first.
- **Migration Considerations:** Existing scheduling data must remain synchronized with the EHR/practice management scheduling system; exact migration or sync approach is TBD.
- **Training/Support Considerations:** Staff will need training on daily schedule view usage, pending request handling, appointment type configuration, and failed reminder follow-up.

## 16. Definition of Product Success

The product is successful when:

- The clinic reduces missed booking opportunities outside office hours against the pre-launch baseline.
- Patients can complete online booking, rescheduling, and reminder-link flows without calling the clinic for eligible scenarios.
- Staff can see the day's appointments and pending requests in one view grouped by provider and time.
- No-show rate improves against the pre-launch baseline after automated reminders launch.
- All Must-priority requirements are accepted.
- Required quality, security, and compliance outcomes are evidenced.

## 17. Approval

| Role | Name | Decision | Date |
|---|---|---|---|
| Product Owner | TBD | Pending | TBD |
| Business Sponsor | TBD | Pending | TBD |