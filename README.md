# RideFlow — Ride-Sharing Database & Operations Platform

A SQL Server database engineering project that models the core operations of a modern ride-sharing platform, including riders, drivers, vehicles, ride requests, trips, payments, ratings, support operations, and auditing.

## Project Status

**Status:** 🟡 In Development  
**Current Phase:** Phase 1 — Requirements & Database Design  
**Database:** Microsoft SQL Server  
**Primary Development Tool:** SQL Server Management Studio (SSMS)

---

## 1. Project Overview

RideFlow is a fictional ride-sharing platform designed as a production-style relational database project.

The project models the database and operational workflows required to support a ride-sharing service where riders request trips, drivers accept ride requests, vehicles are assigned to trips, rides progress through multiple operational states, payments are processed, ratings are submitted, and support teams investigate customer or driver issues.

The goal of RideFlow is not to reproduce the complete architecture of companies such as Uber or Lyft. Instead, it focuses on designing a realistic relational database that demonstrates practical database engineering, SQL development, transaction processing, data integrity, performance optimization, security, and operational reporting.

---

## 2. Project Objectives

RideFlow is being built to demonstrate practical knowledge of:

- Relational database design
- Entity-relationship modelling
- Primary and foreign keys
- One-to-one, one-to-many, and many-to-many relationships
- Database normalization through 3NF
- Data integrity and constraints
- Advanced SQL querying
- JOINs and aggregations
- Subqueries
- Common Table Expressions (CTEs)
- Window functions
- Views
- Stored procedures
- User-defined functions
- Transactions
- ACID principles
- Error handling and rollback
- Concurrency and locking
- Triggers
- Audit logging
- Index design
- Query optimization
- Execution-plan analysis
- Database security
- Role-based permissions
- Backup and recovery
- Operational troubleshooting
- Business reporting

---

## 3. Business Scenario

RideFlow operates a digital ride-sharing platform.

A rider creates an account and submits a ride request containing pickup and destination information.

An eligible driver can be matched with the request and perform the ride using an approved vehicle.

The ride progresses through operational states such as:

```text
REQUESTED
    ↓
MATCHED
    ↓
DRIVER_ARRIVING
    ↓
DRIVER_ARRIVED
    ↓
IN_PROGRESS
    ↓
COMPLETED
```

Alternative outcomes may include cancellation, expiration, payment failure, or other operational exceptions.

RideFlow must preserve enough historical information for operational and support teams to investigate situations such as:

- A rider being charged for a cancelled ride
- A payment failing after a completed ride
- A driver receiving multiple ride assignments simultaneously
- A rider repeatedly cancelling requests
- A driver experiencing unusually high cancellation rates
- A refund not matching its original payment
- An unauthorized or expired vehicle being used
- Delayed pickups
- Incorrect ride status transitions
- Suspicious payment or account activity

---

## 4. High-Level System Flow

```text
User
 │
 ├───────────────┐
 ▼               ▼
Rider           Driver
 │               │
 │               ├── Vehicle
 │               ├── Documents
 │               └── Availability
 │
 ▼
Ride Request
 │
 ▼
Ride
 │
 ├── Ride Status History
 ├── Payment
 │      └── Refund
 ├── Rating
 └── Support / Operational Investigation
```

The database will also contain supporting entities for promotions, service areas, vehicle inspections, support cases, and auditing.

---

## 5. Planned Core Entities

The current conceptual design contains the following planned entities.

| Entity | Purpose |
|---|---|
| Users | Stores common account information |
| Riders | Stores rider-specific information |
| Drivers | Stores driver-specific information |
| Vehicles | Stores registered vehicles |
| DriverVehicleAssignments | Preserves driver-to-vehicle assignment history |
| DriverDocuments | Stores driver licence, insurance, and compliance document information |
| DriverAvailability | Tracks driver online/offline availability |
| RideRequests | Stores ride requests before a trip is successfully created |
| Rides | Stores accepted and active/completed trips |
| RideStatusHistory | Preserves ride lifecycle changes |
| Payments | Stores payment attempts and transactions |
| Refunds | Stores refunds against payments |
| Ratings | Stores rider/driver ratings associated with rides |
| Promotions | Defines promotional offers |
| PromotionUsage | Tracks promotion usage |
| SupportTickets | Stores operational/customer support cases |
| VehicleInspections | Stores vehicle inspection history |
| ServiceAreas | Defines supported operating areas |
| AuditLogs | Records selected important database/system changes |

**Important:** This entity list is not yet considered final. Tables may be added, removed, split, or redesigned during normalization and detailed schema design.

---

## 6. Preliminary Relationships

The following relationships are currently planned:

```text
Users          1 ─── 0..1 Riders
Users          1 ─── 0..1 Drivers

Drivers        1 ─── M DriverVehicleAssignments
Vehicles       1 ─── M DriverVehicleAssignments

Drivers        1 ─── M DriverDocuments
Drivers        1 ─── M DriverAvailability

Riders         1 ─── M RideRequests

RideRequests   1 ─── 0..1 Rides

Drivers        1 ─── M Rides
Vehicles       1 ─── M Rides

Rides          1 ─── M RideStatusHistory

Rides          1 ─── M Payments
Payments       1 ─── M Refunds

Rides          1 ─── M Ratings

Promotions     1 ─── M PromotionUsage
Riders         1 ─── M PromotionUsage

Users          1 ─── M SupportTickets

Vehicles       1 ─── M VehicleInspections

ServiceAreas   1 ─── M RideRequests
```

These relationships will be reviewed during conceptual and logical database design before implementation.

---

## 7. Important Initial Design Decisions

### Users, Riders, and Drivers

Common account information will be separated from role-specific information.

Conceptually:

```text
            Users
           /     \
       Riders   Drivers
```

This avoids unnecessarily duplicating attributes such as names, email addresses, phone numbers, account status, and account creation information.

---

### Ride Requests vs. Rides

`RideRequests` and `Rides` are separate concepts.

A rider may request a trip without a ride ever being created.

Examples include:

- No driver is available
- The request expires
- The rider cancels before matching

Conceptually:

```text
RideRequest
     │
     │ 0..1
     ▼
    Ride
```

This allows RideFlow to analyze request-to-match conversion rates and unsuccessful requests.

---

### Vehicles

Vehicle information will be stored separately from rides.

A ride references the specific vehicle used rather than copying vehicle information into every ride record.

Conceptually:

```text
Vehicles

vehicle_id
make
model
plate
...

        ↓

Rides

ride_id
vehicle_id FK
...
```

This reduces duplicated vehicle information while preserving which vehicle was used for each ride.

---

### Driver-Vehicle History

A vehicle may be associated with different drivers over time.

Instead of permanently storing a single `driver_id` inside `Vehicles`, RideFlow plans to preserve assignment history through a relationship such as:

```text
DriverVehicleAssignments

assignment_id
driver_id
vehicle_id
assigned_from
assigned_until
```

The detailed business rules for vehicle ownership and assignment are still under design.

---

### Ride Status History

The `Rides` table may contain the current ride status for efficient operational access, while `RideStatusHistory` preserves historical status changes.

Example:

```text
Ride 1052

MATCHED          08:31
DRIVER_ARRIVING  08:32
DRIVER_ARRIVED   08:38
IN_PROGRESS      08:40
COMPLETED        09:07
```

This makes operational metrics and investigations possible without losing historical state changes.

---

### Multiple Payment Attempts

A ride is not automatically restricted to exactly one payment record.

For example:

```text
Ride #5001

Payment Attempt 1 → FAILED
Payment Attempt 2 → FAILED
Payment Attempt 3 → SUCCESS
```

Therefore, the initial design allows:

```text
Rides 1 ─── M Payments
```

Detailed payment rules will be finalized later.

---

## 8. Planned Database Features

### Database Design

- ER diagram
- Entity identification
- Relationship modelling
- Cardinality
- PK/FK design
- Normalization to 3NF
- Data dictionary
- Business rules
- Appropriate SQL Server data types
- Constraints

### SQL Development

The project will include approximately **40–50 meaningful business and operational queries** rather than disconnected syntax exercises.

Planned query topics include:

- Rider activity
- Driver performance
- Driver earnings
- Ride demand
- Ride cancellations
- Pickup wait times
- Payment reconciliation
- Failed payments
- Refund analysis
- Revenue analysis
- Service-area performance
- Customer retention
- Driver utilization
- Ratings analysis
- Operational exceptions
- Support investigations

Queries will demonstrate techniques including:

- INNER JOIN
- LEFT JOIN
- Multi-table joins
- GROUP BY
- HAVING
- CASE
- Subqueries
- Correlated subqueries
- CTEs
- Multiple CTEs
- EXISTS / NOT EXISTS
- UNION / UNION ALL where appropriate
- Date functions
- Conditional aggregation
- ROW_NUMBER
- RANK / DENSE_RANK
- LAG / LEAD
- Running totals
- Percentage calculations

### Views

Approximately 5–7 useful reporting/operational views are planned.

Possible examples:

```text
vw_ActiveRides
vw_DriverPerformance
vw_RiderHistory
vw_PaymentReconciliation
vw_ServiceAreaPerformance
```

Final views will be determined from actual business requirements.

### Stored Procedures

Approximately 5–7 meaningful procedures are planned.

Potential workflows include:

- Creating a ride request
- Assigning a driver
- Starting a ride
- Completing a ride
- Processing payment
- Processing refund
- Retrieving ride/customer history

Stored procedures will only be introduced where they provide a reasonable database-level implementation.

### Functions

User-defined functions may be used where a reusable calculation provides genuine value.

Possible examples include fare calculations or reusable operational calculations.

Functions will not be added solely to increase feature count.

### Transactions

Important workflows will demonstrate transaction handling.

A possible ride-completion workflow:

```text
Begin Transaction
       ↓
Validate ride
       ↓
Update ride status
       ↓
Calculate/finalize fare
       ↓
Create payment attempt
       ↓
Update driver availability
       ↓
Commit
```

If an important operation fails:

```text
ROLLBACK
```

Transaction implementations will demonstrate:

- BEGIN TRANSACTION
- COMMIT
- ROLLBACK
- TRY/CATCH
- ACID principles
- Error handling

### Concurrency and Locking

RideFlow will include scenarios demonstrating why concurrency control matters.

Example:

Two requests attempt to assign the same available driver simultaneously.

The project will investigate appropriate SQL Server transaction/isolation/locking behaviour to protect data integrity.

### Triggers and Auditing

Triggers will only be implemented where they provide a reasonable auditing or integrity benefit.

Possible examples:

- Ride status audit
- Driver status changes
- Payment status changes

The project will avoid unnecessary trigger usage.

### Indexing and Performance

Performance work will include:

1. Selecting an important query
2. Reviewing its execution plan
3. Identifying inefficient access
4. Creating an appropriate index
5. Running the query again
6. Comparing the result

Potential indexes may involve:

- Ride timestamps
- Ride status
- Driver references
- Rider references
- Payment status
- Service area
- Frequently used composite search conditions

Indexes will be justified rather than added indiscriminately.

### Security

RideFlow will demonstrate SQL Server security principles using database roles and least-privilege access.

Potential roles:

```text
db_rideflow_admin
db_operations
db_finance
db_support
db_analyst
```

Permissions will be designed according to operational responsibilities.

### Backup and Recovery

The project will demonstrate:

- Full database backup
- Restore process
- Recovery verification
- Documentation of recovery steps

---

## 9. Planned Business Questions

Examples of questions the completed database should be capable of answering include:

1. Which drivers completed the most rides during the previous month?
2. Which riders have the highest lifetime spending?
3. What is the average rider wait time by hour of day?
4. Which drivers have an unusually high cancellation rate?
5. Which riders cancel more than 25% of their ride requests?
6. Which service areas generate the most revenue?
7. Which service areas have the longest pickup wait times?
8. Which completed rides have unresolved payment failures?
9. Which refunds do not correctly reconcile with original payments?
10. Which drivers maintain high ratings across a meaningful number of rides?
11. Which drivers experienced declining earnings across consecutive months?
12. What percentage of ride requests successfully become rides?
13. What percentage of rides result in successful payment?
14. Which riders have become inactive after previously frequent usage?
15. Which time periods experience the highest ride demand?

The final query set will contain approximately 40–50 business and operational questions.

---

## 10. Planned Repository Structure

```text
RideFlow-Ride-Sharing-Database/
│
├── README.md
│
├── database/
│   ├── 01-create-database.sql
│   ├── 02-create-schema.sql
│   ├── 03-seed-data.sql
│   └── 04-verify-database.sql
│
├── queries/
│
├── views/
│
├── stored-procedures/
│
├── functions/
│
├── triggers/
│
├── transactions/
│
├── indexes/
│
├── security/
│
├── backup-recovery/
│
├── docs/
│   ├── database-design.md
│   ├── data-dictionary.md
│   ├── normalization.md
│   ├── business-rules.md
│   └── er-diagram/
│
└── screenshots/
```

The repository structure may evolve as the project develops.

---

## 11. Development Roadmap

### Phase 1 — Requirements & Business Rules

- [x] Define project concept
- [x] Define high-level business scenario
- [x] Identify initial entities
- [ ] Finalize business rules
- [ ] Finalize entity responsibilities

### Phase 2 — Conceptual ER Design

- [ ] Finalize entities
- [ ] Define relationships
- [ ] Define cardinalities
- [ ] Resolve many-to-many relationships
- [ ] Create conceptual ER diagram

### Phase 3 — Logical Schema & Normalization

- [ ] Define attributes
- [ ] Select primary keys
- [ ] Define foreign keys
- [ ] Normalize tables through 3NF
- [ ] Define constraints
- [ ] Create data dictionary
- [ ] Document normalization decisions

### Phase 4 — SQL Server Implementation

- [ ] Create database
- [ ] Create schemas if required
- [ ] Create tables
- [ ] Create PK constraints
- [ ] Create FK constraints
- [ ] Create UNIQUE constraints
- [ ] Create CHECK constraints
- [ ] Create DEFAULT constraints
- [ ] Verify schema

### Phase 5 — Data Generation & Validation

- [ ] Create realistic seed data
- [ ] Populate reference data
- [ ] Populate users/riders/drivers
- [ ] Populate vehicles
- [ ] Populate ride requests/rides
- [ ] Populate payments/refunds
- [ ] Populate ratings
- [ ] Validate referential integrity

### Phase 6 — Business & Operational SQL

- [ ] Create query requirements
- [ ] Implement 40–50 meaningful queries
- [ ] Include beginner/intermediate/advanced SQL
- [ ] Test query results
- [ ] Document business purpose

### Phase 7 — Views

- [ ] Identify useful reporting views
- [ ] Implement approximately 5–7 views
- [ ] Test views
- [ ] Document intended consumers

### Phase 8 — Stored Procedures & Functions

- [ ] Identify appropriate workflows
- [ ] Implement stored procedures
- [ ] Implement functions where justified
- [ ] Add parameter validation
- [ ] Add error handling
- [ ] Test procedures/functions

### Phase 9 — Transactions & Concurrency

- [ ] Implement transaction scenarios
- [ ] Demonstrate COMMIT
- [ ] Demonstrate ROLLBACK
- [ ] Demonstrate TRY/CATCH
- [ ] Document ACID
- [ ] Create concurrency scenario
- [ ] Investigate SQL Server isolation/locking behaviour

### Phase 10 — Triggers & Audit

- [ ] Identify appropriate trigger use cases
- [ ] Implement selected triggers
- [ ] Create auditing where appropriate
- [ ] Test audit behaviour

### Phase 11 — Indexing & Performance

- [ ] Establish baseline queries
- [ ] Review execution plans
- [ ] Identify performance opportunities
- [ ] Create justified indexes
- [ ] Compare before/after execution
- [ ] Document findings

### Phase 12 — Security

- [ ] Define database roles
- [ ] Apply least-privilege permissions
- [ ] Test permitted operations
- [ ] Test denied operations
- [ ] Document security model

### Phase 13 — Backup & Recovery

- [ ] Create database backup
- [ ] Document backup process
- [ ] Restore database
- [ ] Verify restored data
- [ ] Document recovery process

### Phase 14 — Testing

- [ ] Test constraints
- [ ] Test invalid inserts
- [ ] Test FK protection
- [ ] Test transaction rollback
- [ ] Test procedures
- [ ] Test triggers
- [ ] Test permissions
- [ ] Test important business workflows

### Phase 15 — Portfolio Documentation

- [ ] Finalize ER diagram
- [ ] Finalize data dictionary
- [ ] Add screenshots
- [ ] Document major design decisions
- [ ] Document performance improvements
- [ ] Document security implementation
- [ ] Document backup/recovery
- [ ] Add setup instructions
- [ ] Add example outputs
- [ ] Rewrite README for final portfolio presentation
- [ ] Final repository review

---

## 12. Project Completion Criteria

RideFlow will be considered complete when the repository demonstrates:

- A normalized relational database
- A documented ER model
- Realistic business rules
- Appropriate PK/FK relationships
- Data-integrity constraints
- Realistic test data
- 40–50 meaningful SQL queries
- Advanced SQL techniques
- Views
- Stored procedures
- Appropriate functions
- Transactions
- Error handling
- Concurrency concepts
- Appropriate triggers
- Audit capability
- Indexing and performance analysis
- Security roles and permissions
- Backup and restore
- Testing
- Professional technical documentation

---

## 13. Design Principles

The following principles should guide development:

### Business requirements before SQL

Tables and database features should exist because they support a business requirement, not simply because a technology needs to be demonstrated.

### Avoid unnecessary complexity

RideFlow is intended to demonstrate strong junior-level database engineering knowledge.

It should not imitate enterprise-scale architecture unnecessarily.

### Preserve important history

Operational information such as ride status changes, driver availability, vehicle assignments, payments, and refunds should retain appropriate historical records.

### Protect data integrity

Constraints, relationships, transactions, and application/database rules should prevent invalid states where reasonably possible.

### Design for explainability

Every major table, relationship, index, trigger, procedure, and transaction should be understandable and defensible during a technical interview.

### Optimize based on evidence

Indexes and performance changes should be based on actual query patterns and execution-plan observations rather than speculative optimization.

---

## 14. Technology

**Database**

- Microsoft SQL Server

**Development / Administration**

- SQL Server Management Studio (SSMS)

**Version Control**

- Git
- GitHub

**Documentation**

- Markdown
- ER diagrams
- SQL Server execution plans
- Screenshots

Potential future integration with Power BI may be considered after the core database project is complete.

---

## 15. Project Continuation Context

This section exists so development can continue across different work sessions without losing important architectural context.

### Current State

The repository structure and initial project specification have been defined.

The project is currently in:

**Phase 1 — Requirements & Business Rules**

No production schema should be considered finalized yet.

### Decisions Already Made

1. The project will use Microsoft SQL Server.
2. RideFlow is a fictional ride-sharing platform.
3. `Users` will hold common account information.
4. `Riders` and `Drivers` will represent role-specific information.
5. `RideRequests` and `Rides` will be separate entities.
6. Vehicles will exist independently from rides.
7. Each ride will reference the vehicle actually used.
8. Driver/vehicle historical assignments should be preserved.
9. Ride lifecycle history should be preserved.
10. Multiple payment attempts may exist for a ride.
11. Database features should be introduced because of business requirements rather than simply to demonstrate syntax.
12. The finished project should remain understandable and defensible by a final-year student applying for junior database, SQL, reporting, or support positions.

### Still To Be Designed

The following decisions have **not** yet been finalized:

- Complete entity list
- Complete table attributes
- Driver/vehicle assignment rules
- Driver document structure
- Ride-request matching model
- Location representation
- Fare structure
- Cancellation model
- Payment rules
- Refund rules
- Rating rules
- Promotion rules
- Support-ticket relationships
- Audit strategy
- Exact PK/FK structure
- SQL Server data types
- Constraints
- Indexes
- Security roles and permissions

### Next Recommended Task

Continue **Phase 1 — Requirements & Business Rules**.

The next design topic should be:

**Driver ↔ Vehicle relationship and assignment history**

Determine whether multiple drivers may use the same vehicle over time and define the business rules for `DriverVehicleAssignments`.

After that, continue through the major entities one at a time before creating SQL Server tables.

---

## 16. Current Development Rule

Do **not** jump directly into creating all SQL tables.

The intended development sequence is:

```text
Requirements
    ↓
Business Rules
    ↓
Entities
    ↓
Relationships
    ↓
Cardinality
    ↓
ER Diagram
    ↓
Attributes
    ↓
Normalization
    ↓
PK / FK Design
    ↓
Constraints
    ↓
SQL Server Implementation
    ↓
Seed Data
    ↓
Queries
    ↓
Database Programming
    ↓
Transactions / Concurrency
    ↓
Performance
    ↓
Security
    ↓
Backup / Recovery
    ↓
Testing
    ↓
Portfolio Documentation
```

This sequence is intentional so that RideFlow demonstrates database design and engineering rather than only SQL syntax.

---

## Author

Developed as a database engineering and SQL portfolio project using Microsoft SQL Server.

## License

No license has currently been selected.
