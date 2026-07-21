# Enterprise Vendor Management Platform

An Airtable-based vendor operations and compliance management system designed to centralize vendor onboarding, document tracking, compliance review, expiration monitoring, and operational follow-up.

The platform demonstrates how Airtable can be used as an operational system rather than simply a spreadsheet or database. It combines relational data modeling, workflow automation, business rules, forms, action queues, and management dashboards into a connected vendor lifecycle workflow.

---

## Business Problem

Vendor management often involves information spread across spreadsheets, email threads, shared folders, and disconnected systems.

This creates several operational problems:

- Vendor information must be entered and maintained manually
- Required documents may be missing or overlooked
- Compliance documents can expire without timely follow-up
- Operations teams have limited visibility into vendor readiness
- Review tasks are difficult to prioritize
- Vendor and document statuses can become inconsistent
- Manual follow-up increases administrative workload

The goal of this project was to design a centralized system that allows operations teams to manage the vendor lifecycle from initial submission through document collection, compliance review, approval, and ongoing monitoring.

---

## Solution Overview

The Enterprise Vendor Management Platform centralizes vendor information and connects it with contacts, compliance documents, and operational tasks.

The system supports:

- Vendor onboarding
- Vendor classification
- Contact management
- Required-document generation
- W-9 driven onboarding workflows
- Document review and approval
- Document expiration tracking
- Exception and action management
- Automated task creation and closure
- Vendor status monitoring
- Operational dashboards and reporting

The platform is built around four connected tables:

**Vendors → Contacts → Documents → Action Queue**

Airtable Interfaces and Forms provide simplified views for operational users and vendor intake.

---

## System Architecture

### Vendors

The Vendors table serves as the central vendor record.

It stores information such as:

- Vendor name
- Vendor type
- Vendor lifecycle status
- Associated contacts
- Associated documents
- Open operational tasks
- Document counts
- Compliance indicators

Vendor-level rollups and formulas summarize activity from related records so operations teams can quickly identify vendors requiring attention.

---

### Contacts

The Contacts table stores individuals associated with each vendor.

Typical information includes:

- First name
- Last name
- Email
- Phone
- Vendor relationship

Contacts are linked back to the appropriate vendor record.

---

### Documents

The Documents table manages vendor compliance and onboarding documentation.

Examples include:

- W-9
- Business License
- Insurance Certificate
- Master Service Agreement
- NDA

Each document can track:

- Vendor
- Document type
- Required status
- Review status
- Uploaded date
- Expiration date
- Approval state
- Related operational tasks

Document lifecycle statuses include:

**Requested → Under Review → Approved / Rejected → Expired**

---

### Action Queue

The Action Queue provides an operational worklist for items requiring follow-up.

Tasks are linked to both the relevant vendor and document.

Examples include:

- Missing required document
- Expired compliance document
- Document requiring follow-up
- Vendor documentation exception

Each task includes:

- Task description
- Vendor
- Document
- Status
- Priority
- Open-task indicator

This creates a centralized exception-management workflow instead of requiring operations teams to manually search the Documents table.

---

## Vendor Onboarding Workflow

The vendor onboarding process begins with a vendor intake form.

The form captures core vendor information including:

- Vendor name
- Vendor type
- Contact first name
- Contact last name
- Website
- Contact phone
- Contact email

After submission, the information is used to create and connect the appropriate vendor and contact records.

The onboarding workflow then begins document collection.

---

## W-9 Driven Document Workflow

The W-9 acts as an important onboarding checkpoint.

Once a required W-9 reaches **Approved** status, the system can automatically create the additional documents required for the vendor onboarding process.

For example:

- Business License
- Insurance Certificate
- Master Service Agreement

These records are created with an initial status of **Requested**, allowing the operations team to immediately see which documents still need to be collected.

### Duplicate Prevention

A key workflow challenge was preventing duplicate required-document records.

A status-based automation alone can fire again if a W-9 moves away from Approved and later returns to Approved.

To make the workflow idempotent, the Documents table includes a control field:

**Post-W9 Documents Created?**

The automation checks this field before generating downstream documents.

The workflow becomes:

```text
W-9 Approved
      ↓
Post-W9 Documents Created? = No
      ↓
Create required document records
      ↓
Set Post-W9 Documents Created? = Yes
      ↓
Future status changes do not recreate the documents
```

This prevents duplicate Business License, Insurance Certificate, and Master Service Agreement records from being generated if the W-9 is reprocessed.

---

## Document Review Workflow

Submitted documents move through a controlled review lifecycle.

```text
Requested
    ↓
Under Review
    ↓
Approved
   ↙   ↘
Rejected  Expired
```

The workflow allows operations teams to distinguish between:

- Documents that have not yet been received
- Documents awaiting review
- Approved documents
- Rejected documents
- Documents requiring renewal because they have expired

---

## Expiration Monitoring

Documents containing expiration dates can be evaluated for expiration.

When an applicable document passes its expiration date, the workflow can:

1. Identify the expired document
2. Update the document status to **Expired**
3. Create an operational follow-up task
4. Link the task to the correct vendor and document
5. Surface the issue through the Action Queue and operational reporting

This converts document expiration from a passive date field into an actionable compliance workflow.

---

## Automated Action Queue

The Action Queue is designed around exception-based operations.

Instead of asking users to continuously inspect every document record, the system creates operational tasks when intervention is required.

Example:

```text
Document Expires
      ↓
Status = Expired
      ↓
Create Action Queue Record
      ↓
Status = Open
      ↓
Operations Team Takes Action
```

When the underlying document is subsequently approved, the related open Action Queue item can be automatically closed.

```text
Document Approved
      ↓
Find related open Action Queue record
      ↓
Update task status
      ↓
Closed
```

This keeps operational tasks synchronized with the actual state of the underlying document.

---

## Operations Dashboard

An Airtable Interface provides an executive and operational view of vendor activity.

The dashboard includes KPI cards such as:

- Total Vendors
- Approved Vendors
- Vendors Under Review
- Documents Awaiting Review

Additional visualizations provide insight into:

### Vendor Status Distribution

Shows the distribution of vendors across operational states such as:

- Approved
- Under Review
- Action Required
- Inactive

### Vendor Type

Provides visibility into vendor classification, such as:

- Company
- Individual

### Documents by Status

Displays the current document pipeline across:

- Approved
- Under Review
- Requested
- Expired
- Rejected

### Action Required

Provides a focused operational view of vendors requiring intervention because of incomplete, expired, rejected, or pending documentation.

---

## Automation Architecture

The platform uses multiple Airtable automations to manage different parts of the vendor lifecycle.

Key automation patterns include:

| Automation | Purpose |
|---|---|
| Vendor Intake | Creates and connects vendor/contact information from form submissions |
| Document Submitted for Review | Moves submitted documents into the review workflow |
| W-9 Approved – Start Vendor Review | Generates downstream required documents after W-9 approval |
| Duplicate Prevention | Prevents downstream documents from being recreated |
| Expiration Check | Identifies documents that have passed their expiration date |
| Expired Document → Action Queue | Creates an operational task for expired documents |
| Approved Document → Close Action Queue | Closes related open tasks when the document is resolved |
| Vendor Status Sync | Keeps vendor-level operational status aligned with document activity |

Separating the workflows into focused automations makes the system easier to troubleshoot and maintain than placing the entire lifecycle inside one large automation.

---

## Business Rules and Controls

Several calculated fields and control fields support the workflow.

### Required?

Identifies documents that are mandatory for the vendor onboarding or compliance process.

### Post-W9 Documents Created?

Acts as an automation control flag to prevent duplicate downstream document creation.

### Approved Flag

Identifies documents currently in an approved state for rollups and reporting.

### Under Review Flag

Supports vendor-level calculations for documents currently awaiting review.

### Action Required Flag

Identifies documentation requiring operational intervention.

### Open Task Flag

Allows open Action Queue items to be counted and summarized at the vendor level.

These fields allow record-level activity to roll up into vendor-level operational health indicators.

---

## Relational Data Model

The system uses linked records rather than duplicating information across tables.

```text
VENDORS
   │
   ├──────── CONTACTS
   │
   └──────── DOCUMENTS
                  │
                  └──────── ACTION QUEUE
```

This structure allows the system to answer operational questions such as:

- Which documents belong to this vendor?
- Which required documents are still outstanding?
- Which vendors currently have documents under review?
- Which documents have expired?
- Which vendors have unresolved operational tasks?
- Which action item belongs to which document?
- Has an issue already been resolved?

---

## Key Design Challenge: Automation Idempotency

One of the most important technical lessons from the project involved automation idempotency.

Initially, approving a W-9 correctly generated the required downstream documents.

However, if the W-9 status was changed and later returned to Approved, the automation could trigger again and create duplicate document records.

The solution was not simply another status condition.

Instead, the workflow introduced persistent state through:

**Post-W9 Documents Created?**

This allows the automation to distinguish between:

> "The W-9 is approved"

and:

> "The W-9 is approved AND the downstream onboarding workflow has never been generated."

This pattern is useful in many operational systems where automations must safely handle records moving backward and forward through workflow states.

---

## Operational Design Principles

This project was built around several principles.

### Automate exceptions, not just data movement

Automations create actionable operational work rather than simply copying values between fields.

### Maintain one source of truth

Vendor, document, and task information is connected through relational records rather than duplicated manually.

### Make workflows state-aware

Control fields prevent repeated automation runs from producing duplicate business records.

### Surface problems

Expired, missing, rejected, or pending documents are surfaced through operational queues and dashboards.

### Separate data from user experience

The underlying relational tables maintain the system of record, while Airtable Interfaces provide simplified operational views.

---

## Screenshots

### Operations Dashboard
![Operations Dashboard](assets/operations_dashboard_kpi.png)
The operations dashboard provides real-time visibility into vendor volume, approval status, documents awaiting review, vendor status distribution, and vendor classification.

### Documents by Status

![Documents by Status](assets/documents_by_status.png)
Document-level reporting provides visibility into Approved, Under Review, Requested, Expired, and Rejected documentation.

### Action Required View

![Action Required](assets/operations_dashboard_action_required.png)
The Action Required interface surfaces vendors requiring operational intervention based on documentation and compliance status.

### Document Workflow

![Document Workflow](assets/document-workflow.png)
The Documents table tracks each vendor document through its lifecycle from Requested through review, approval, rejection, or expiration.

### Action Queue

![Action Queue](assets/action-queue.png)
The Action Queue provides a centralized worklist for unresolved documentation issues and automatically closes tasks when the underlying issue is resolved.
---

## Skills Demonstrated

This project demonstrates practical experience with:

- Airtable relational database design
- Linked records
- Lookup fields
- Rollups
- Formula fields
- Forms
- Airtable Interfaces
- Dashboard design
- Workflow automation
- Conditional automation triggers
- Record creation and updates
- Find-record automation patterns
- Repeating automation actions
- Cross-table workflow management
- Exception management
- Document lifecycle management
- Automation debugging
- Duplicate prevention
- Idempotent workflow design
- Operational reporting
- Business-process modeling

---

## Testing and Validation

The workflows were functionally tested using sample vendor and document records.

Testing included:

- Vendor onboarding
- W-9 approval
- Required-document creation
- Reprocessing an approved W-9
- Duplicate prevention
- Document review status changes
- Document expiration
- Action Queue creation
- Action Queue closure
- Vendor-level status updates
- Dashboard reporting

During testing, automation behavior was also validated through Airtable automation run history.

The project is currently implemented in an Airtable Free workspace, where monthly automation-run limits apply. High-volume production deployment would require appropriate workspace capacity and additional production testing.

---

## Future Enhancements

Potential extensions include:

- Automated email reminders for missing documents
- Expiration notifications before documents expire
- Vendor self-service document upload
- Escalation rules for overdue compliance items
- SLA tracking
- Approval timestamps and audit history
- Role-based operational interfaces
- Automated renewal workflows
- Integration with CRM or procurement platforms
- Slack or Microsoft Teams notifications
- External workflow orchestration through Make, Zapier, or APIs
- Higher-volume automation architecture for production environments

---

## Project Outcome

The final platform demonstrates a connected vendor operations workflow rather than a collection of independent Airtable tables.

Vendor onboarding, document collection, compliance review, expiration monitoring, operational task management, and management reporting are connected through a shared relational data model.

The result is a system designed to help operations teams answer three questions quickly:

**What is the current state of each vendor?**

**What documentation or compliance issue needs attention?**

**What action needs to happen next?**

---

## Technology

**Platform:** Airtable  
**Components:** Tables, Forms, Interfaces, Automations, Formulas, Linked Records, Lookups, Rollups  
**Use Case:** Vendor Operations & Compliance Management
- Vendor Performance Scorecards
- SLA Tracking
- Approval Workflows
- Executive Reporting
- Vendor Analytics

  ---

## Project Roadmap

### Version 1.0 — Core Vendor Management System

**Completed**

- Relational Database Design
- Vendor Management
- Contact Management
- Document Management
- Linked Record Architecture
- Lookup Fields
- Rollup Fields
- Formula Fields
- Operations Dashboard
- Vendor Management Console
- Interactive Airtable Interfaces

### Version 2.0 — Workflow Automation & Compliance Management

**Completed**

- Vendor Intake Form
- Automated Vendor Onboarding
- Automated Vendor Status Updates
- W-9 Approval Workflow
- Automated Required Document Creation
- Document Review Workflow
- Document Expiration Monitoring
- Automated Action Queue Creation
- Automated Action Queue Closure
- Open Task Tracking
- Vendor-Level Compliance Indicators
- Duplicate Document Prevention
- Idempotent W-9 Workflow Controls
- Action Required Dashboard
- Document Status Reporting

### Version 2.1 — Planned

- Compliance Reminder Emails
- Pre-Expiration Document Notifications
- Automated Renewal Reminders
- Escalation Rules for Overdue Documents
- Approval and Review Audit Timestamps

### Version 3.0 — Future

- Contract Lifecycle Management
- Vendor Self-Service Document Portal
- Role-Based Approval Workflows
- Procurement / CRM Integration
- Slack or Microsoft Teams Notifications
- External Workflow Integration via API, Make, or Zapier
- SLA and Compliance Performance Reporting
