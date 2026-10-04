# RideFlow — Ride-Sharing Database & Operations Platform

RideFlow is a relational database engineering project that models the core
operations of a modern ride-sharing platform.

The project is being built using Microsoft SQL Server and focuses on designing
and implementing a reliable database for managing riders, drivers, vehicles,
ride requests, completed rides, payments, refunds, ratings, promotions,
support activity, and operational history.

Rather than functioning only as a collection of SQL queries, RideFlow is
designed as a complete database project covering requirements analysis,
relational modelling, normalization, data integrity, transactional workflows,
database programmability, security, auditing, performance optimization, and
backup/recovery concepts.

---

## Project Objectives

The project is intended to demonstrate practical database engineering skills
through a realistic operational system.

The main objectives are to:

- Translate business requirements into a relational database design
- Model entities, relationships, and cardinalities
- Apply primary keys, foreign keys, constraints, and normalization
- Maintain data integrity across ride and payment workflows
- Implement reusable database logic with views, procedures, and functions
- Handle transactional operations safely
- Preserve important operational and status history
- Introduce auditing for sensitive database changes
- Investigate indexing and SQL query performance
- Apply role-based database security concepts
- Explore concurrency, locking, and transaction isolation
- Document backup and recovery considerations
- Build realistic operational and support queries
- Maintain clear technical documentation and design decisions

---

## Technology

### Primary Platform

- Microsoft SQL Server
- SQL Server Management Studio (SSMS)
- T-SQL

### Project / Documentation Tools

- Git
- GitHub
- Markdown
- ER modelling tools

Additional technologies will only be introduced where they have a clear role
in the project.

---

# System Scope

RideFlow represents the database behind a fictional ride-sharing platform.

The system must support several major operational areas.

## Rider Management

The database should support:

- Rider account information
- Ride request activity
- Ride history
- Payment history
- Ratings
- Promotion usage
- Customer support activity

## Driver Management

The database should support:

- Driver profiles
- Driver status
- Driver documentation
- Driver availability
- Vehicle relationships
- Ride assignments
- Driver ratings
- Operational history

## Vehicle Management

Vehicle information is maintained independently from driver and ride records.

The database should eventually support:

- Vehicle identification
- Registration information
- Vehicle status
- Driver/vehicle relationships
- Vehicle inspections
- Historical ride usage

A ride should reference the specific vehicle used to perform that ride rather
than copying vehicle attributes into the ride record.

The detailed driver-to-vehicle assignment model has not yet been finalized.

## Ride Operations

The database should distinguish between a customer's request for transportation
and the actual ride that may result from that request.

This allows the system to represent scenarios such as:

- A rider submits a request
- No driver accepts the request
- The rider cancels before matching
- A driver accepts the request
- An accepted ride is later cancelled
- A ride progresses through multiple operational statuses
- A ride is successfully completed

Ride status changes should be capable of being preserved historically rather
than only storing the latest state.

## Payments and Refunds

The payment model should eventually support:

- Ride charges
- Payment status
- Multiple payment attempts where appropriate
- Failed payments
- Successful payments
- Refunds
- Payment reconciliation

Financial workflows must be designed with transaction integrity in mind.

## Ratings

After eligible rides, the platform should be able to store ratings associated
with the ride.

Detailed rating rules and constraints will be finalized during database design.

## Promotions

The system should support promotional offers and track their usage.

The design must prevent invalid or inconsistent promotion usage through
appropriate relational rules and constraints.

## Customer Support

Support cases should be capable of being associated with relevant operational
records such as riders, drivers, rides, or payments where appropriate.

This portion of the project will also be used to explore database-support and
troubleshooting scenarios.

## Auditing

Important database changes may require audit records.

Auditing will be introduced selectively where a genuine operational or security
reason exists rather than logging every database operation.

---

# Preliminary Entity Model

The following entities have been identified during initial requirements
analysis.

They are NOT yet considered the final database schema.

- Users
- Riders
- Drivers
- Vehicles
- DriverDocuments
- DriverAvailability
- RideRequests
- Rides
- RideStatusHistory
- Payments
- Refunds
- Ratings
- Promotions
- PromotionUsage
- SupportTickets
- VehicleInspections
- ServiceAreas
- AuditLogs

Entities may be added, removed, combined, or separated as the relational model
is developed.

No entity in this list should be treated as finalized until relationships,
business rules, keys, cardinalities, and normalization have been reviewed.

---

# Important Design Decisions

## Ride Request vs Ride

RideFlow separates a ride request from an actual ride.

A RideRequest represents a rider asking for transportation.

A Ride represents the operational trip created when the request progresses
into an actual driver assignment / ride workflow.

This prevents unsuccessful or cancelled requests from being incorrectly treated
as completed ride records.

---

## Vehicle as an Independent Entity

Vehicle details should not be duplicated inside every ride.

Vehicles are maintained separately and rides reference the vehicle involved in
the trip.

This provides stronger normalization and allows RideFlow to determine which
vehicle was used for a historical ride.

The complete model for drivers using different vehicles over time remains a
pending design decision.

---

## Historical Status Information

Important operational status changes should not necessarily overwrite previous
information.

RideFlow plans to investigate a history model that allows the database to
answer questions such as:

- When was a driver assigned?
- When did the driver arrive?
- When did the ride begin?
- When was the ride completed?
- Was the ride cancelled?
- How long did each stage take?

---

## Payment Attempt History

The project should not assume every payment succeeds on the first attempt.

The final payment model should be capable of representing operational scenarios
such as failed attempts followed by successful payment.

The exact schema will be determined during detailed design.

---

# Database Engineering Areas

RideFlow is planned to demonstrate the following areas.

## Relational Design

- Entities
- Relationships
- Cardinality
- Primary keys
- Foreign keys
- Candidate/unique keys
- 1NF
- 2NF
- 3NF

## Data Integrity

Where appropriate, the implementation will use:

- PRIMARY KEY
- FOREIGN KEY
- NOT NULL
- UNIQUE
- CHECK
- DEFAULT

Constraints should enforce real business rules rather than being added only for
demonstration.

## Database Programmability

Planned areas include:

- Views
- Stored procedures
- User-defined functions
- Triggers where justified

## Transaction Management

The project will explore:

- Transactions
- COMMIT
- ROLLBACK
- TRY/CATCH
- ACID properties
- Multi-step operations
- Concurrency
- Locking
- Isolation levels

## Performance

Performance work will include:

- Execution plan investigation
- Index selection
- Composite indexes
- Query optimization
- SARGable filtering
- Avoiding unnecessary indexes
- Comparing performance before and after optimization

## Security

The security phase is planned to explore:

- Database users
- Database roles
- Permissions
- Least privilege
- Restricted access to sensitive operations

## Backup and Recovery

The project will document and test appropriate SQL Server backup/recovery
concepts in a development environment.

## Auditing

Selected operations may be audited to preserve information about important
changes and support investigation.

---

# Planned Operational Scenarios

RideFlow should eventually be able to investigate scenarios such as:

- Finding available drivers
- Matching ride requests to drivers
- Tracking ride lifecycle events
- Detecting unusually long ride stages
- Investigating cancelled rides
- Investigating failed payments
- Processing refunds safely
- Identifying driver or vehicle operational issues
- Reviewing rider and driver activity
- Investigating support cases
- Reviewing promotion usage
- Identifying frequently executed queries that require optimization
- Auditing selected sensitive changes

These scenarios will be refined as the database model develops.

---

# Repository Structure

```text
RideFlow-Ride-Sharing-Database/
│
├── README.md
│
├── database/
├── queries/
├── views/
├── stored-procedures/
├── functions/
├── triggers/
├── transactions/
├── indexes/
├── security/
├── backup-recovery/
├── docs/
└── screenshots/