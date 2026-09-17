# AYP Digital Health, Response & Intelligence Platform

## Complete System Architecture Specification

**Version:** 1.0\
**Date:** 17 September 2026\
**Architecture direction:** Java / Spring Boot, modular, event-driven,
interoperability-first\
**Interoperability foundation:** Kenya Core FHIR R4 + DHA Health
Information Exchange (HIE)\
**AI/ML:** Not used in the core platform

------------------------------------------------------------------------

## 1. Executive Summary

The AYP platform is proposed as a **privacy-preserving digital support,
referral, coordination and population-health intelligence platform for
adolescents and young people (AYP)**.

It is not intended to replace existing government systems such as
M-Dharura, eCHIS, hospital HMIS platforms, or the DHA national
interoperability ecosystem. Instead, it provides a coordinated
AYP-specific layer that can receive signals from young people, Community
Health Promoters (CHPs), health facilities and youth officers;
coordinate human responses; manage referrals and escalation; and convert
operational events into aggregated intelligence for planning and service
improvement.

The platform will use **Java and Spring Boot** for its core backend. The
choice supports alignment with the Java/FHIR ecosystem used by DHA while
keeping the platform independent at the domain level.

The architecture will use:

-   Spring Boot and Java for application services
-   PostgreSQL for transactional data
-   An event-driven architecture
-   A modular monolith initially, with controlled extraction of services
    later
-   REST/OpenAPI for application APIs
-   FHIR R4 at interoperability boundaries
-   Kenya Core FHIR profiles and terminology
-   DHA HIE for national health-data interoperability
-   Role-, attribute-, geography- and relationship-based authorization
-   Separate handling of identity, case data and population intelligence
-   Deterministic rules instead of AI/ML
-   Immutable audit/event records
-   Offline-tolerant workflows where required
-   Versioned integration and event contracts

The central architectural idea is:

> **Collect signals from multiple trusted entry points, coordinate the
> minimum people required to respond, preserve privacy through
> architectural separation, and transform response events into useful
> population-level intelligence.**

------------------------------------------------------------------------

# 2. Architectural Vision

The platform should evolve from a normal case-management application
into an **AYP Signal → Response → Intelligence Network**.

``` text
Young Person
     │
CHP ─┼─ Facility ─ Youth Officer
     │
     ▼
  AYP SIGNAL
     │
     ▼
 RESPONSE ENGINE
     │
 ┌───┼────────┐
 ▼   ▼        ▼
Support Referral Escalation
     │
     ▼
   OUTCOME
     │
     ▼
AYP INTELLIGENCE
     │
     ▼
Government Planning & Action
```

The platform therefore has three major capabilities:

1.  **Response Network** --- handles individual support, referral,
    verification, follow-up and resolution.
2.  **Intelligence Network** --- identifies aggregated trends, service
    gaps, bottlenecks and demand.
3.  **Interoperability Network** --- connects AYP workflows with DHA
    HIE, eCHIS, hospital HMIS and future systems.

------------------------------------------------------------------------

# 3. Design Principles

## 3.1 Interoperability First

The platform shall not invent a separate national health-data language.

FHIR R4, Kenya Core FHIR profiles, national identifiers, terminology and
DHA HIE interfaces shall be used at health-system boundaries.

Kenya Core FHIR 1.0.0 is the national foundational layer for profiles,
extensions, terminologies and identifier patterns, and domain-specific
IGs are designed to build on it.

## 3.2 Privacy by Architecture

Privacy shall not depend only on policy.

The architecture shall separate:

-   identity
-   case information
-   referral information
-   operational events
-   aggregated intelligence

A national or county analytics user should not need a young person's
name to understand a population-level service problem.

## 3.3 Minimum Necessary Access

A user receives only the information required for their role, location,
organization and relationship to a case.

## 3.4 Human-in-the-Loop

The platform supports people; it does not replace authorized government
or health decisions.

The system can:

-   identify workflow requirements
-   assign
-   notify
-   calculate deadlines
-   route according to deterministic rules
-   detect thresholds
-   escalate workflow

Authorized people make substantive decisions.

## 3.5 No AI/ML in the Core Platform

No machine-learning or generative-AI model is required for:

-   triage
-   referral routing
-   case resolution
-   escalation
-   trend detection
-   population analytics

The platform uses deterministic rules, coded data, statistical
aggregation, workflow engines and human decisions.

## 3.6 DHA-Centric Health Interoperability

The AYP platform should integrate with the national DHA ecosystem rather
than become an independent health-data silo.

## 3.7 Modular Evolution

The first implementation should be a modular Spring Boot application.
Modules must have clear boundaries so that high-scale components can
later become independent services without redesigning the domain.

------------------------------------------------------------------------

# 4. High-Level Architecture

``` text
┌───────────────────────────────────────────────────────────────────┐
│                         EXPERIENCE LAYER                           │
│                                                                   │
│ Young Person Web/PWA │ CHP Interface │ Facility Portal           │
│ Youth Officer Portal │ County/National Dashboard │ Future USSD    │
└───────────────────────────────┬───────────────────────────────────┘
                                │
                                ▼
┌───────────────────────────────────────────────────────────────────┐
│                         ACCESS LAYER                               │
│                                                                   │
│ API Gateway │ Authentication │ Authorization │ Rate Limiting      │
│ Session Management │ Device Trust │ API Security                  │
└───────────────────────────────┬───────────────────────────────────┘
                                │
                                ▼
┌───────────────────────────────────────────────────────────────────┐
│                      AYP SPRING BOOT CORE                          │
│                                                                   │
│ Signal │ Case │ Case Team │ Referral │ Workflow │ Escalation      │
│ CHP │ Facility │ Youth Officer │ Consent │ Notification           │
│ Resolution │ Audit │ Configuration │ Service Directory            │
└───────────────────────────────┬───────────────────────────────────┘
                                │
                                ▼
┌───────────────────────────────────────────────────────────────────┐
│                           EVENT BUS                                │
│                                                                   │
│ Case Events │ Referral Events │ Response Events │ Escalation      │
│ Signal Events │ Outcome Events │ Integration Events               │
└───────────────┬───────────────────────────────┬───────────────────┘
                │                               │
                ▼                               ▼
┌──────────────────────────────┐    ┌──────────────────────────────┐
│     INTELLIGENCE LAYER       │    │   INTEROPERABILITY LAYER     │
│                              │    │                              │
│ Trend Engine                 │    │ FHIR Adapter                 │
│ Service Gap Engine           │    │ DHA HIE Adapter              │
│ Referral Analytics           │    │ eCHIS Adapter                │
│ Response Analytics           │    │ Hospital HMIS Adapters       │
│ Geographic Aggregation       │    │ M-Dharura Integration        │
│ Bottleneck Detection         │    │ Future System Adapters       │
└──────────────┬───────────────┘    └──────────────┬───────────────┘
               │                                   │
               ▼                                   ▼
      ┌──────────────────┐              ┌─────────────────────────┐
      │ Analytics Store  │              │ DHA HIE / External      │
      │ / Warehouse      │              │ Government Systems      │
      └──────────────────┘              └─────────────────────────┘
```

------------------------------------------------------------------------

# 5. Technology Stack

## Backend

-   Java
-   Spring Boot
-   Spring Security
-   Spring Data JPA
-   Spring Validation
-   Spring Integration
-   Spring Kafka or equivalent event infrastructure
-   Spring Boot Actuator

## Database

-   PostgreSQL
-   Redis for caching and short-lived operational state where
    appropriate

## Interoperability

-   HL7 FHIR R4
-   Kenya Core FHIR IG
-   DHA HIE APIs
-   REST
-   OpenAPI
-   OAuth 2.0 / appropriate DHA authentication mechanisms
-   FHIR Bundles where required

## FHIR implementation

Use HAPI FHIR libraries for FHIR parsing, validation and
interoperability functions rather than implementing FHIR serialization
and validation from scratch.

## Frontend

-   React / Next.js for web and PWA interfaces
-   A mobile application may be introduced for CHPs where device/offline
    requirements justify it

## Infrastructure

-   Docker
-   Managed PostgreSQL
-   Managed object storage
-   Containerized Spring Boot deployment
-   CI/CD pipeline
-   Centralized monitoring and logging

Kubernetes should not be mandatory for the first deployment. It can be
introduced when scale and operational requirements justify it.

------------------------------------------------------------------------

# 6. Application Architecture

The core application should initially be a **modular monolith**.

``` text
ayp-platform
│
├── identity
├── authorization
├── young-person
├── signal
├── case-management
├── case-team
├── chp
├── facility
├── youth-officer
├── service-directory
├── referral
├── workflow
├── escalation
├── consent
├── notification
├── resolution
├── audit
├── analytics
└── interoperability
```

Each module owns its domain logic and exposes explicit interfaces.

The modules should not directly manipulate each other's database
internals.

------------------------------------------------------------------------

# 7. Why Modular Monolith First

Starting immediately with many microservices would increase:

-   deployment complexity
-   debugging complexity
-   infrastructure cost
-   distributed transaction problems
-   operational overhead

The system should instead begin as:

``` text
Spring Boot Application
       │
       ├── Domain Modules
       ├── Event Publisher
       └── Integration Layer
```

When a module needs independent scaling, it can become a service.

Potential future extraction:

``` text
AYP Core
  │
  ├── Case Service
  ├── Referral Service
  ├── Notification Service
  ├── Intelligence Service
  └── Interoperability Service
```

------------------------------------------------------------------------

# 8. Domain Model

The core AYP domain should not simply be a collection of CRUD entities.

The important domain objects are:

### YoungPerson

Represents the person receiving support.

### AYP Signal

A structured observation or request entering the system.

Sources may include:

-   young person
-   CHP
-   health facility
-   youth officer
-   approved future source

### AYP Case

A managed response process created from a signal or operational
interaction.

### Case Team

The minimum authorized group of people who need to participate in a
case.

### Service Need

The service or intervention required.

### Referral

A controlled transfer/request for service.

### Response

An action taken by an authorized actor.

### Resolution

The outcome recorded by an authorized actor.

### Escalation

Movement of responsibility to a higher or designated level.

### Service Provider

An organization/facility capable of providing a defined service.

### Event

An immutable record that something happened in the system.

### Population Signal

An aggregated analytical observation derived from multiple operational
events.

------------------------------------------------------------------------

# 9. Signal Architecture

Not everything should immediately become a full case.

``` text
Incoming Observation
        │
        ▼
     SIGNAL
        │
        ▼
Determine:
- Is support required?
- Is verification required?
- Is referral required?
- Is this a case?
- Is this only population intelligence?
```

This allows the system to distinguish between:

-   individual support
-   operational cases
-   service demand
-   population-level patterns

This is one of the main differences between the platform and a
conventional reporting system.

------------------------------------------------------------------------

# 10. Multi-Entry Reporting

The system supports multiple entry points.

## Young Person

``` text
Young Person
   ↓
Submit request/signal
   ↓
AYP Platform
```

## CHP

``` text
CHP identifies issue
   ↓
Create signal/case
   ↓
Verify if required
   ↓
Resolve or refer
```

## Facility

``` text
Facility / Youth Officer
   ↓
Create case
   ↓
Assess
   ↓
Provide support / refer / escalate
```

## Youth Officer

``` text
Youth Officer
   ↓
Create / receive / coordinate case
   ↓
Assign / refer / resolve / escalate
```

------------------------------------------------------------------------

# 11. Case Workflow

A recommended state model:

``` text
NEW
 │
 ▼
ASSIGNED
 │
 ▼
ACKNOWLEDGED
 │
 ▼
IN_PROGRESS
 │
 ├───────────────┐
 │               │
 ▼               ▼
REFERRED      RESOLVED
 │
 ▼
REFERRAL_ACCEPTED
 │
 ▼
IN_PROGRESS
 │
 ├───────────────┐
 ▼               ▼
RESOLVED      ESCALATED
                  │
                  ▼
             HIGHER LEVEL
```

Additional states may include:

-   VERIFIED
-   AWAITING_ACTION
-   TRANSFERRED
-   UNABLE_TO_RESOLVE
-   CLOSED

The final state machine should be configurable by case type.

------------------------------------------------------------------------

# 12. CHP Architecture

CHPs are first-class actors.

They can:

-   report cases
-   receive assigned cases
-   verify cases where authorized
-   provide permitted support
-   refer cases
-   follow up
-   record outcomes
-   resolve cases within their mandate

They should not automatically have access to every young person's record
in their geographical area.

A CHP should normally access:

-   assigned cases
-   cases for which they have an authorized relationship
-   minimum information needed for the task
-   approved community-level information

------------------------------------------------------------------------

# 13. Facility Architecture

Facilities provide both an entry point and a resolution point.

A facility user may:

-   report a case
-   receive a CHP referral
-   accept/reject a referral
-   provide service
-   record outcome
-   refer onward
-   escalate
-   close a case where authorized

The facility should receive only the information required for the
referral/service.

The complete private case history should not automatically be
transmitted.

------------------------------------------------------------------------

# 14. Youth Officer Architecture

Youth officers operate across:

-   sub-county
-   county
-   national

Their visibility should correspond to their administrative scope and
function.

### Sub-county

-   assigned cases
-   referrals requiring their action
-   unresolved cases
-   local trends
-   escalation monitoring

### County

-   sub-county performance
-   aggregated county trends
-   unresolved/escalated cases
-   service gaps
-   referral performance

### National

-   national trends
-   service demand
-   county/sub-county performance
-   system-wide bottlenecks
-   aggregated population intelligence

------------------------------------------------------------------------

# 15. Hierarchical Escalation

The hierarchy should identify handled and unhandled cases.

``` text
Case
 │
 ▼
Assigned actor
 │
 ├── handled → resolved
 │
 └── not handled within rule
          │
          ▼
     next responsible level
          │
          ▼
       escalation
```

Escalation should be rule-based.

Example:

``` text
Case assigned
     ↓
Acknowledgement deadline
     ↓
Action deadline
     ↓
Reminder
     ↓
Supervisor notification
     ↓
Sub-county escalation
     ↓
County escalation
```

Deadlines must be configurable by case type and policy rather than
hard-coded.

------------------------------------------------------------------------

# 16. M-Dharura Relationship

AYP should not reproduce M-Dharura's existing incident-accountability
functionality.

Instead:

``` text
AYP
 │
 ├── AYP support workflow
 ├── referral
 ├── youth coordination
 ├── service intelligence
 └── population intelligence
       │
       ▼
M-Dharura where applicable
```

Where a case qualifies for an existing government operational pathway,
the AYP integration layer can create or exchange the required
information subject to authorization and integration agreements.

------------------------------------------------------------------------

# 17. Referral Engine

The referral engine should be deterministic.

``` text
Service Need
     │
     ▼
Required Service
     │
     ▼
Eligible Providers
     │
     ├── Location
     ├── Facility capability
     ├── Service availability
     ├── Operating status
     └── Other configured constraints
     │
     ▼
Referral Destination
```

No AI is necessary.

The system should record:

-   referring actor
-   destination
-   reason
-   service requested
-   referral date
-   acceptance
-   appointment/action where applicable
-   completion
-   outcome
-   onward referral

------------------------------------------------------------------------

# 18. Interoperability Architecture

The platform must separate internal domain logic from external
representations.

``` text
AYP Domain Model
      │
      ▼
Integration Adapter
      │
      ▼
FHIR Mapping
      │
      ▼
DHA HIE
```

Do not make the internal database a direct copy of FHIR resources.

This protects the AYP domain from changes in external implementation
details.

------------------------------------------------------------------------

# 19. Kenya Core FHIR Alignment

The current Kenya Core FHIR Implementation Guide v1.0.0 is active and
based on FHIR R4. It defines the national foundational layer of
profiles, extensions, terminology and identifier patterns that
domain-specific implementation guides build upon.

AYP should therefore:

1.  Use FHIR R4.
2.  Reuse Kenya Core profiles.
3.  Reuse Kenya terminology where applicable.
4.  Use national identifiers/registry references where authorized.
5.  Follow Kenya Core security/provenance conventions.
6.  Build an AYP-specific domain implementation layer where existing
    resources do not fully express AYP workflows.

The Kenya Core architecture explicitly supports modular domain IGs,
including Referral.

------------------------------------------------------------------------

# 20. FHIR Mapping

Potential mappings:

  AYP concept              FHIR representation
  ------------------------ ---------------------------------
  Young person             Patient
  CHP                      Practitioner / PractitionerRole
  Youth officer            Practitioner / PractitionerRole
  Facility                 Organization / Location
  Service provider         Organization / PractitionerRole
  Service need             ServiceRequest
  Referral task            Task
  Encounter                Encounter
  Consent                  Consent
  Communication            Communication
  Condition                Condition
  Observation              Observation
  Supporting document      DocumentReference
  Provenance               Provenance
  Population measurement   Measure / MeasureReport

These mappings must be validated against the applicable current DHA IGs
before production implementation.

------------------------------------------------------------------------

# 21. DHA HIE Integration

The DHA HIE provides national infrastructure for secure and authorized
health-data exchange and exposes services around authentication,
consent, registries and standardized exchange.

AYP should use the HIE rather than establish independent national
registries.

Potential integration areas:

``` text
AYP
 │
 ├── Authentication
 ├── Client Registry
 ├── Facility Registry
 ├── Practitioner/Worker information
 ├── Terminology
 ├── Consent
 ├── Referral
 └── Shared Health Record where authorized
```

Access must be based on actual DHA onboarding, authorization and
production integration requirements. Public documentation does not by
itself grant production access.

------------------------------------------------------------------------

# 22. Integration Adapter Pattern

Every external system should have its own adapter.

``` text
AYP Integration Layer
│
├── DHA-HIE Adapter
├── eCHIS Adapter
├── HMIS Adapter
├── M-Dharura Adapter
└── Future-System Adapter
```

The internal application should not contain vendor-specific code.

For example:

``` text
AYPReferral
      ↓
ReferralIntegrationService
      ↓
DHAReferralAdapter
      ↓
FHIR ServiceRequest + Task
```

A future system can then receive the same domain event through another
adapter.

------------------------------------------------------------------------

# 23. Hospital HMIS Integration

Hospital systems such as Tiberbu/Taifacare, KenyaEMR and other approved
HMIS platforms should be treated as external systems.

The DHA Digital Health Superhighway currently identifies Tiberbu (public
HMIS -- Taifacare), KenyaEMR, national registries,
KHISL/interoperability infrastructure and analytics as ecosystem
components.

AYP should therefore avoid creating direct point-to-point dependencies
where the DHA interoperability layer can provide the appropriate route.

Where a direct integration is required and authorized, use an adapter.

------------------------------------------------------------------------

# 24. Event-Driven Architecture

Important events include:

``` text
ayp.signal.created.v1
ayp.case.created.v1
ayp.case.assigned.v1
ayp.case.acknowledged.v1
ayp.case.verified.v1
ayp.case.referred.v1
ayp.referral.accepted.v1
ayp.referral.completed.v1
ayp.response.recorded.v1
ayp.case.escalated.v1
ayp.case.resolved.v1
ayp.case.closed.v1
ayp.consent.updated.v1
ayp.integration.failed.v1
```

Events should be versioned.

Example:

``` text
ayp.case.resolved.v1
ayp.case.resolved.v2
```

This protects future integrations from breaking when the event schema
evolves.

------------------------------------------------------------------------

# 25. Event Flow Example

``` text
CHP reports case
      │
      ▼
CaseCreated
      │
      ├── Case Service
      ├── Notification Service
      ├── Audit Service
      ├── Analytics
      └── Escalation Monitor
```

Later:

``` text
ReferralCreated
      │
      ├── Referral workflow
      ├── Facility notification
      ├── Analytics
      ├── Audit
      └── DHA integration
```

The event bus becomes the platform's operational nervous system.

------------------------------------------------------------------------

# 26. Privacy Architecture

Data should be classified into four major levels.

## Level 1 --- Identity

Examples:

-   name
-   phone
-   date of birth
-   identifiers
-   contact information

Highly restricted.

## Level 2 --- Case

Examples:

-   concern
-   assessment
-   verification
-   actions
-   follow-up

Restricted to authorized case participants.

## Level 3 --- Referral

Examples:

-   service required
-   referral reason
-   destination
-   relevant information
-   referral status

Shared only with the appropriate destination/service.

## Level 4 --- Intelligence

Examples:

-   county
-   sub-county
-   age group
-   issue category
-   service demand
-   referral completion
-   resolution statistics

Primarily aggregated/de-identified.

------------------------------------------------------------------------

# 27. Identity Separation

A conceptual architecture:

``` text
              IDENTITY STORE
           name / phone / identifiers
                    │
                    │ protected reference
                    ▼
               CASE STORE
                    │
                    ▼
               EVENT STORE
                    │
                    ▼
             ANALYTICS STORE
                    │
                    ▼
        Aggregated intelligence
```

Analytics should not require direct access to the identity store.

------------------------------------------------------------------------

# 28. Authorization Architecture

Use a combination of:

### RBAC

Role-based access:

-   CHP
-   facility officer
-   youth officer
-   supervisor
-   county officer
-   national officer
-   administrator

### ABAC

Attribute-based constraints:

-   organization
-   facility
-   county
-   sub-county
-   program
-   case type

### Relationship-based authorization

For example:

> Is this user actually assigned to this case?

A permission decision can therefore be:

``` text
Role
+
Organization
+
Geographic scope
+
Case relationship
+
Requested action
=
Authorization decision
```

------------------------------------------------------------------------

# 29. Consent

Consent should be handled explicitly.

The platform should distinguish:

-   consent for collection
-   consent for a particular use
-   consent for referral/data sharing
-   withdrawal/update
-   legally required or safeguarding-related exceptions

The exact rules must follow applicable Kenyan law, policy and DHA
requirements.

The architecture should integrate with DHA consent services where
applicable rather than inventing a separate national consent mechanism.

------------------------------------------------------------------------

# 30. Safeguarding and Emergency Workflows

The platform must not promise absolute confidentiality.

Some cases may require authorized intervention or disclosure according
to applicable law, policy and safeguarding procedures.

The architecture should therefore have configurable:

``` text
Emergency
Safeguarding
Mandatory escalation
Standard support
```

pathways.

These should be reviewed and defined with the relevant government
stakeholders before implementation.

------------------------------------------------------------------------

# 31. Analytics Architecture

Operational data should not be used directly for heavy analytics.

``` text
Operational PostgreSQL
        │
        ▼
       Events
        │
        ▼
Aggregation / ETL
        │
        ▼
Analytics Store
        │
        ▼
Dashboards
```

Potential analytics:

-   case volumes
-   service demand
-   resolution rates
-   referral completion
-   time to action
-   time to resolution
-   escalation rates
-   geographic patterns
-   service gaps
-   referral bottlenecks
-   repeated demand
-   facility/service utilization

------------------------------------------------------------------------

# 32. Population Signal Detection

No AI is required.

Example:

``` text
Daily service requests
       │
       ▼
7-day baseline
       │
       ▼
Current count
       │
       ▼
Configured statistical threshold
       │
       ▼
Population Signal
```

The system can use deterministic statistical methods such as:

-   moving averages
-   percentage changes
-   thresholds
-   confidence intervals where appropriate
-   rate calculations
-   geographic aggregation
-   time-series comparisons

The output is a signal for human review, not an automated diagnosis or
policy decision.

------------------------------------------------------------------------

# 33. Service Gap Detection

The platform should relate:

``` text
Demand
  +
Available Services
  +
Referral Outcomes
  +
Geography
```

Example:

``` text
High demand
     +
Low referral completion
     +
Few eligible providers
     =
Potential service gap
```

The platform flags the pattern.

An authorized officer determines the appropriate response.

------------------------------------------------------------------------

# 34. Response Bottleneck Detection

The system should monitor the full pathway:

``` text
Reported
   ↓
Assigned
   ↓
Acknowledged
   ↓
Action started
   ↓
Referred
   ↓
Referral accepted
   ↓
Service delivered
   ↓
Resolved
```

If a large number of cases stop at one stage, the system identifies a
potential bottleneck.

This provides more useful intelligence than simply counting "open
cases."

------------------------------------------------------------------------

# 35. Government Dashboards

## CHP

``` text
My cases
New
Assigned
In progress
Awaiting action
Referred
Resolved
```

## Facility

``` text
Incoming referrals
Accepted
Awaiting action
Completed
Escalated
```

## Sub-county

``` text
Cases
Handled
Unresolved
Escalated
Referral completion
Response time
Service demand
```

## County

``` text
Sub-county comparison
Aggregated trends
Service gaps
Referral bottlenecks
Escalation patterns
```

## National

``` text
National AYP trends
Regional/service demand
System bottlenecks
Referral performance
Aggregated population intelligence
```

The dashboards should respect the same authorization boundaries as the
operational system.

------------------------------------------------------------------------

# 36. Audit Architecture

Every sensitive operation should generate an audit event.

Examples:

``` text
Case viewed
Case created
Case assigned
Case referred
Case modified
Consent changed
Record exported
Record accessed
Case resolved
Case escalated
```

Audit data should include:

-   actor
-   timestamp
-   action
-   resource
-   purpose/context where required
-   source/device metadata as appropriate
-   outcome

Audit records should be protected from ordinary users modifying their
own history.

------------------------------------------------------------------------

# 37. Security Architecture

Core controls:

-   TLS for data in transit
-   encryption at rest
-   strong authentication
-   least-privilege authorization
-   token-based API security
-   secret management
-   audit logging
-   secure session management
-   rate limiting
-   input validation
-   API gateway controls
-   vulnerability scanning
-   dependency management
-   database access controls
-   backup and recovery
-   environment separation

Sensitive data should never be placed in normal application logs.

------------------------------------------------------------------------

# 38. Offline and Low-Connectivity Strategy

CHP and community workflows may operate in environments with unreliable
connectivity.

The client should support:

``` text
Capture locally
     ↓
Encrypt
     ↓
Queue
     ↓
Connectivity returns
     ↓
Synchronize
     ↓
Server validates
     ↓
Event created
```

Conflict resolution and duplicate prevention must be designed
explicitly.

Offline storage should be minimized and protected.

------------------------------------------------------------------------

# 39. Notification Architecture

Notifications should be decoupled from the core case service.

``` text
Case Event
    │
    ▼
Notification Service
    │
 ┌──┼───────────┐
 ↓  ↓           ↓
Push SMS      Email
```

Notifications should contain the minimum necessary information.

For example, a generic CHP notification may indicate:

> New assigned support request requiring action.

It should not expose the entire sensitive case in a notification
channel.

------------------------------------------------------------------------

# 40. Data Architecture

Primary transactional database:

**PostgreSQL**

Logical domains:

``` text
identity
cases
signals
case_teams
referrals
services
facilities
organizations
workflows
escalations
consents
notifications
audit
configuration
```

Analytical data:

``` text
facts
dimensions
aggregates
time-series
geographic summaries
service summaries
```

Identity data should have additional logical and physical security
boundaries where appropriate.

------------------------------------------------------------------------

# 41. Resilience

External systems may become unavailable.

AYP should not stop functioning because DHA HIE or an HMIS is
temporarily unavailable.

Example:

``` text
AYP
 │
 ▼
Integration Request
 │
 ├── Success → external system
 │
 └── Failure
       ↓
   Secure Queue
       ↓
     Retry
       ↓
   External system
```

Integration failures should be visible to system administrators and
relevant operators.

------------------------------------------------------------------------

# 42. Integration Reliability

Use:

-   retry policies
-   exponential backoff
-   dead-letter queues
-   idempotency keys
-   correlation IDs
-   timeout controls
-   circuit breakers where appropriate
-   reconciliation jobs
-   integration monitoring

Every external transaction should be traceable.

------------------------------------------------------------------------

# 43. Correlation and Traceability

A case should have:

``` text
AYP Case ID
```

and where external systems are involved:

``` text
AYP Case ID
External Reference ID
FHIR Resource ID
Referral ID
```

The mapping should allow the platform to trace a transaction without
duplicating an external system's source of truth.

------------------------------------------------------------------------

# 44. API Architecture

Internal APIs:

``` text
/api/v1/signals
/api/v1/cases
/api/v1/referrals
/api/v1/facilities
/api/v1/services
/api/v1/chps
/api/v1/officers
/api/v1/workflows
/api/v1/escalations
```

External interoperability APIs should be separated:

``` text
/api/v1/integration/dha/*
/api/v1/integration/echis/*
/api/v1/integration/hmis/*
```

FHIR endpoints should follow the applicable FHIR server/API conventions
rather than being mixed with ordinary application endpoints.

------------------------------------------------------------------------

# 45. API Versioning

APIs should be versioned.

``` text
/v1
/v2
```

Breaking changes should never silently alter an existing production
contract.

------------------------------------------------------------------------

# 46. Observability

Use:

-   structured logs
-   metrics
-   distributed tracing
-   health checks
-   integration monitoring
-   event processing monitoring

Recommended standards:

-   OpenTelemetry
-   Spring Boot Actuator
-   centralized log management
-   metrics dashboards

Important metrics:

``` text
API latency
Event processing delay
Failed integrations
Referral processing time
Case acknowledgement time
Case resolution time
Queue depth
Notification failures
Database health
```

------------------------------------------------------------------------

# 47. Deployment Architecture

Initial deployment:

``` text
Internet
   │
   ▼
Load Balancer / Reverse Proxy
   │
   ▼
Spring Boot Application
   │
   ├── PostgreSQL
   ├── Redis
   ├── Event Broker
   └── Object Storage
```

Production should have:

-   multiple application instances
-   automated backups
-   database monitoring
-   encrypted storage
-   secrets management
-   CI/CD
-   disaster recovery procedures

The hosting provider should be selected based on security, availability,
data residency/governance requirements, operational cost and
compatibility with government integration requirements.

------------------------------------------------------------------------

# 48. Development Environments

Separate:

``` text
Development
     ↓
Testing
     ↓
Integration/Sandbox
     ↓
Staging
     ↓
Production
```

DHA integration should use the appropriate sandbox/test environment
before production onboarding.

No production health data should be used in development.

------------------------------------------------------------------------

# 49. Testing Strategy

## Unit Testing

Domain rules and services.

## Integration Testing

Database, event broker, FHIR services and adapters.

## Contract Testing

Ensure external integration contracts remain compatible.

## FHIR Validation

Validate generated resources against the relevant Kenya Core/domain
profiles.

## Security Testing

-   authentication
-   authorization
-   privilege escalation
-   API security
-   data leakage
-   audit integrity

## Performance Testing

-   concurrent case reporting
-   event throughput
-   dashboard queries
-   referral processing

## Offline Testing

-   synchronization
-   duplicate submission
-   conflict resolution
-   interrupted transfers

------------------------------------------------------------------------

# 50. Governance

The platform should establish governance for:

-   data ownership
-   data stewardship
-   access approval
-   integration approval
-   terminology management
-   FHIR profile governance
-   retention
-   audit
-   security
-   incident response
-   change management

The final production governance model must be agreed with the relevant
government authorities.

------------------------------------------------------------------------

# 51. AYP FHIR Domain Implementation Guide

A future AYP FHIR IG should be considered.

It should:

1.  Build on Kenya Core.
2.  Reuse existing profiles wherever possible.
3.  Define AYP-specific profiles only where necessary.
4.  Define AYP value sets.
5.  Define AYP extensions sparingly.
6.  Define referral and response workflows.
7.  Define examples.
8.  Define validation rules.
9.  Define security/provenance requirements.
10. Define interoperability use cases.

Potential AYP-specific concepts include:

``` text
AYP Signal
AYP Service Need
AYP Case
AYP Response
AYP Escalation
AYP Outcome
```

These should be represented using existing FHIR resources where
appropriate rather than creating unnecessary custom resources.

------------------------------------------------------------------------

# 52. Future System Integration

A future government system should be able to connect through:

``` text
API
FHIR
Events
Approved integration adapter
```

without changing the AYP core.

``` text
Future System
     │
     ▼
Integration Adapter
     │
     ▼
AYP Integration Layer
     │
     ▼
AYP Domain
```

This is the main mechanism for future-proofing.

------------------------------------------------------------------------

# 53. Architectural Differentiator

The system's uniqueness should not be claimed to come from using Spring
Boot or FHIR.

Those provide the foundation.

The innovation is the combination of:

### AYP Signal Network

Multiple trusted actors can contribute signals.

### Response Network

The minimum required people coordinate the response.

### Referral Network

Service needs are connected to appropriate service providers.

### Event Network

Every operational action becomes a structured event.

### Intelligence Network

Events become aggregated service and population intelligence.

### Interoperability Network

The platform connects to the national ecosystem through standards.

### Privacy Architecture

Identity, case information and population intelligence are deliberately
separated.

Together:

``` text
SIGNAL
  ↓
RESPONSE
  ↓
REFERRAL
  ↓
OUTCOME
  ↓
EVENT
  ↓
INTELLIGENCE
  ↓
ACTION
  ↓
SERVICE IMPROVEMENT
```

------------------------------------------------------------------------

# 54. Recommended Implementation Phases

## Phase 1 --- Foundation

-   Spring Boot project
-   PostgreSQL
-   authentication
-   authorization
-   users/roles
-   organizations
-   facilities
-   service directory
-   audit
-   base event infrastructure

## Phase 2 --- AYP Response

-   signal creation
-   case management
-   CHP workflow
-   facility workflow
-   youth officer workflow
-   case teams
-   resolution
-   escalation

## Phase 3 --- Referral

-   service matching
-   facility routing
-   referral lifecycle
-   referral tracking
-   outcome tracking

## Phase 4 --- Interoperability

-   FHIR implementation
-   Kenya Core validation
-   DHA sandbox
-   registries
-   consent integration
-   referral interoperability
-   approved HMIS/eCHIS integrations

## Phase 5 --- Intelligence

-   analytics pipeline
-   trend detection
-   service gaps
-   referral bottlenecks
-   response performance
-   geographic aggregation

## Phase 6 --- National Scale

-   performance optimization
-   high availability
-   additional integrations
-   formal AYP FHIR IG
-   advanced analytics
-   multi-county deployment

------------------------------------------------------------------------

# 55. What We Should NOT Build

The architecture explicitly avoids:

-   a replacement for DHA
-   a replacement for M-Dharura
-   a replacement for eCHIS
-   a replacement for hospital HMIS
-   a national duplicate facility registry
-   a separate national patient registry
-   an AI diagnosis engine
-   an AI case-decision engine
-   unrestricted government access to youth records
-   a single giant database shared by every ministry
-   direct point-to-point integration with every external system where
    HIE/adapters are appropriate

------------------------------------------------------------------------

# 56. Core Architectural Decisions

  Decision                  Choice
  ------------------------- -----------------------------------------
  Backend                   Java + Spring Boot
  Architecture              Modular monolith initially
  Evolution                 Event-driven/service extraction
  Database                  PostgreSQL
  Cache                     Redis
  Messaging                 Event broker
  API                       REST/OpenAPI
  Health interoperability   FHIR R4
  National foundation       Kenya Core FHIR
  National integration      DHA HIE
  FHIR library              HAPI FHIR
  Authorization             RBAC + ABAC + relationship
  Analytics                 Separate analytical layer
  Privacy                   Identity/case/intelligence separation
  Referral                  Rules-based
  Escalation                Rules-based
  Intelligence              Statistical/deterministic
  AI/ML                     None
  CHP                       First-class actor
  Facility                  First-class actor
  Youth Officer             First-class actor
  M-Dharura                 Complement/integration, not replacement
  eCHIS                     Integration
  HMIS                      Integration
  Future systems            Adapter/event/API model

------------------------------------------------------------------------

# 57. Final Architecture Statement

The AYP platform should be implemented as a:

> **Java/Spring Boot, modular, event-driven, privacy-preserving,
> interoperability-first digital platform that sits above the AYP
> response workflow and connects to Kenya's national digital-health
> ecosystem through Kenya Core FHIR and DHA HIE.**

The platform's internal architecture should remain domain-oriented and
independent of external system representations.

Its external health interoperability should follow Kenya's current FHIR
direction.

Its operational intelligence should be generated from structured events
rather than AI.

Its privacy model should separate identity, case management, referral
information and population intelligence.

Its government accountability model should follow:

``` text
Reported
   ↓
Assigned
   ↓
Acknowledged
   ↓
Action
   ↓
Referral if required
   ↓
Resolution
   ↓
Escalation if unresolved
   ↓
Aggregated intelligence
   ↓
Government action
```

This creates a platform that is not merely another reporting
application. It becomes an **AYP response and intelligence layer within
the wider Kenyan digital-health ecosystem**.

------------------------------------------------------------------------

## 58. External Architecture References

The architecture should be implemented against the current official DHA
specifications rather than assumptions.

Key references:

-   Kenya Core FHIR Implementation Guide --- current release 1.0.0, FHIR
    R4
-   DHA HIE API documentation
-   Kenya Patient Summary FHIR Implementation Guide
-   Relevant Kenya Referral/domain implementation guides
-   DHA terminology and registry services
-   Applicable Kenyan digital-health legislation and regulations

All production integration points, permissions, consent requirements,
data-sharing arrangements and government responsibilities must be
confirmed with DHA and the relevant authorities before deployment.
