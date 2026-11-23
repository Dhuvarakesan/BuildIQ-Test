# Scheduling Configuration Acceptance Tests

This directory contains the pilot launch acceptance test pack for appointment type configuration behavior and accessibility.

The scenarios are designed for execution against the HIPAA-ready vendor-hosted scheduling SaaS using clinic pilot tenant test accounts. They verify that appointment type booking settings persist in vendor-managed configuration, are protected by MFA-backed RBAC, drive the correct patient routing behavior, and have launch evidence for audit reporting and WCAG 2.1 AA accessibility.

Test accounts, clinic-approved appointment type names, vendor report export paths, and vendor accessibility scanner output locations must be populated by the clinic/vendor implementation team before formal pilot sign-off. Prototype appointment type examples may be used only while the clinic-approved list is pending.
