# Neo CQI API Design Notes

## Current CQI Capabilities

```text

Current CQI Capabilities
├── Report discovery and filter setup
├── Drug library / care area / drug reference data
├── Compliance analysis
├── Limit-event analysis
├── Dose-rate-change analysis
├── Device usage analysis
├── Infusion story reconstruction
├── Guardian event reporting (limited)
├── Export and audit
├── Pump Event History Log files Collection, Aggregation and Download
└── Data quality / processing observability

```

## Future CQI Capabilities (Short Term)

```text

Future CQI Capabilities (Short Term)
├── Alarms Analysis
├── Minimum Required Flow Rate Analysis
├── Data push to BCLP for Replication
├── PeerVue capabilities in CQI (functionality merge)
├── DoseIQ Integration
├── DeviceVue Integration
├── Usage Analytics
└── New Guardian Report

```

## Future CQI Capabilities (Long Term)

```text

Future CQI Capabilities (Long Term)
├── Predictive Maintianence for the Pumps
├── Reliability Metrics Collection and Aggregation for Pumps
├── AutoDocumentation capabilities
├── Analysis of Pumps Events Data and EHL Data to determine usage patterns
└── Alarm Burden/Fatigue Analytics

```


# Neo CQI Domain Concepts

## Current CQI Domain Concepts
```text

Customer / Site / Tenant
Enterprise hierarchy
Device
Pump type
Drug library
Drug library version
Care area
Drug
Infusion
Infusion event
Programming event
Limit event
Guardian event
Pump Event History Log Event
Dose rate change
Report
Filter
Export
User preference
Data processing status

```

## Future CQI Domain Concepts (Short Term)
```text

Alarm
Flow Rate
Replication Config
Replication Data Payload
Comparator Data For PeerVue
Seed Data For PeerVue
Care Area Mapping Data
PeerVue Configurations
Drug Library Feedback
DoseIQ Configurations
Pump Utilization Details
DeviceVue Configurations
CQI Usage Details
New Guardian Report Details
Guardian Summary Details
Guardian Dose Rate Details
Guardian Flow Rate Details
Guardian Concentration Details
Guardian Dose Rate Step Change Details
Guardian Patient Weight Details
Guardian Uncommon Drug Details
Guardian Limit Analysis
Guardian Deep Dive Details
Guardian Graph Details
Guardian Notes Details

```

## Future CQI Domain Concepts (Long Term)
```text

TBD

```



---
---

# Bounded Contexts

## For Current CQI

| Bounded Context | Owns | Why it should be separate |
| --- | --- | --- |
| Reference Data Context | Drug library, drug library version, care area, drug, pump type, device metadata, enterprise hierarchy | These are filters and dimensions used across many reports. Recent Neo-CQI work added drug library, care area, and drug details through ingestion, bronze-to-gold movement, and CQIQUERY API updates. |
| Infusion Journey Context | Infusion, infusion lifecycle, infusion story, event sequence | Infusion story is a user-facing CQI concept already present in the current CQI API/UI design. | 
| Compliance Reporting Context | Compliance summaries, compliance infusions, compliance details | Compliance report is an explicit CQI report area in current UI/API documentation. |
| Limit & Alert Context | Soft limit, hard limit, limit type, limit event, Guardian event, alert summary | Internal sources mention Guardian integration, limit-event leaderboard/report requirements, and Guardian events as part of CQI reporting. | 
| Dose Rate Change Context | Dose rate changes, dose change details, dose rate change report filters | Existing API signatures include dose rate change care area and dose rate change infusion detail operations. | 
| Device Usage Context | Device activity, utilization, pump availability, device usage report | CQI v3.3 requirements include Device Usage Report filters by drug library, device type, and enterprise group, plus reset and retained selections. |
| Report Experience Context | Saved filters, preferences, report state, export requests | Current CQI UI documentation includes user preference APIs, saved filters, sorting, paging, and export-oriented workflows. |
| Data Operations Context | Processing status, data quality, late messages, duplicates, invalid messages, lineage | CQI FMEA and known failure-mode notes call out schema drift, retry, observability, rollback, duplicate handling, dead-letter/invalid messages, and out-of-order message risks. |


## For Future CQI (Short Term)
| Bounded Context | Owns | Why it should be separate |
| --- | --- | --- |
| TBD | TBD | TDB |


## For Future CQI (Long Term)
| Bounded Context | Owns | Why it should be separate |
| --- | --- | --- |
| TBD | TBD | TDB |

# Recommended API Structure

## Suggested Implementetion Plan

```text

        Pump / IQE / Guardian events
                    ↓
            Raw event ingestion
                    ↓
        Canonical event validation
                    ↓
        Domain event normalization
                    ↓
           Lakehouse storage
                    ↓
CQI semantic projections / materialized views
                    ↓
           Domain query services
                    ↓
           Neo CQI API contracts
                    ↓
                 CQI UI

```

## For Current CQI
* [reference-data-api](neo-cqi-api-contracts/cqi-reference-data-api.yaml)
* [compliance-api](neo-cqi-api-contracts/cqi-compliance-api.yaml)
* limits-api
* dose-rate-change-api
* device-usage-api
* infusion-story-api
* guardian-alert-api
* report-preferences-api
* data-quality-api

## For Future CQI (Short Term)
TBD

## For Future CQI (Long Term)
TBD

## API Detailed Design Notes

### reference-data-api

#### Responsibilities
##### Owns:
```text
Drug Library
Drug Library Version
Care Area
Drug
Pump Type
Device Metadata
Enterprise Hierarchy
Location
Report Filter Metadata
```
##### Does NOT own:
```text
Infusions
Limit Events
Guardian Events
Reports
Compliance Calculations
```

#### Domain Model
```text
DrugLibrary
 └─ DrugLibraryVersion
      ├─ CareArea
      ├─ Drug
      └─ PumpType

EnterpriseHierarchy
 └─ EnterpriseGroup
      └─ Hospital
           └─ Location

Device
 ├─ Serial Number
 ├─ Pump Type
 └─ Current Metadata
```

#### Events Owned by Reference Data Context
Since Neo CQI is event-driven, this context should also publish domain events:

```text
DrugLibraryCreated
DrugLibraryVersionActivated
CareAreaAdded
CareAreaUpdated
DrugAdded
DrugUpdated
EnterpriseHierarchyChanged
DeviceRegistered
DeviceMetadataUpdated
```

#### Final Reference Data Context API Portfolio
```text
GET /reference/filter-model

GET /reference/drug-libraries
GET /reference/drug-libraries/{id}
GET /reference/drug-libraries/{id}/versions

GET /reference/drug-library-versions/{id}/care-areas

GET /reference/care-areas/{id}

GET /reference/drugs
GET /reference/drugs/{id}

GET /reference/enterprise-hierarchy
GET /reference/enterprise-hierarchy/{id}/children

GET /reference/devices
GET /reference/devices/{id}

GET /reference/report-filters
```

This would be the first bounded context implemented, because every other context (Compliance, Limits, Guardian, Infusion Story, Device Usage) depends on it but it has almost no dependency on them. It is the cleanest place to establish ubiquitous language and API governance before the report-centric APIs are designed.


---
---

### compliance-api
#### Responsibilities
##### Owns:
```text
Compliance Evaluation
Compliance Status
Compliance Summary
Compliance Breakdown
Non-Compliant Infusions
Compliance Trends
Compliance Metrics
```
##### Does NOT own:

```text
Reference Data Context
Infusion Journey Context
Limit & Alert Context
Guardian Context
Device Usage Context
Report Experience Context
Export Context
Data Operations / Data Quality Context
Lakehouse / Storage Context
```

##### Consumes:

From Reference Context:

```text
DrugLibrary
DrugLibraryVersion
CareArea
Drug
Enterprise Hierarchy

```

From Reference Context:
```text
Infusion
Infusion Events
Programming Events

```

#### Domain Model
```text

Core Model

ComplianceEvaluation
 ├─ infusionId
 ├─ complianceStatus
 ├─ evaluatedAt
 ├─ evaluatedAgainst
 │   ├─ drugLibraryVersionId
 │   ├─ careAreaId
 │   ├─ drugId
 │   └─ complianceDefinitionVersion
 └─ findings[]
      ├─ findingCode
      ├─ severity
      ├─ description
      ├─ occurredAt
      └─ relatedEventReference
```

```text
Analytical Model

ComplianceSummary
 ├─ totalInfusions
 ├─ compliantInfusions
 ├─ nonCompliantInfusions
 ├─ unknownComplianceInfusions
 └─ complianceRate

ComplianceBreakdown
 ├─ groupBy
 └─ items[]
      ├─ id
      ├─ name
      ├─ totalInfusions
      ├─ compliantInfusions
      ├─ nonCompliantInfusions
      ├─ unknownComplianceInfusions
      └─ complianceRate

ComplianceTrend
 ├─ granularity
 └─ points[]
      ├─ periodStart
      ├─ periodEnd
      ├─ totalInfusions
      ├─ compliantInfusions
      ├─ nonCompliantInfusions
      └─ complianceRate

```
```text
Infusion compliance view

ComplianceInfusion
 ├─ infusionId
 ├─ deviceId
 ├─ deviceSerialNumber
 ├─ pumpType
 ├─ drugLibraryVersionId
 ├─ careAreaId
 ├─ careAreaName
 ├─ drugId
 ├─ drugName
 ├─ complianceStatus
 ├─ startedAt
 └─ endedAt

```

#### Events Owned by Reference Data Context
Since Neo CQI is event-driven, this context should also publish domain events:

```text
ComplianceEvaluationCreated
ComplianceEvaluationUpdated
ComplianceSummaryCalculated
ComplianceBreakdownCalculated
ComplianceTrendCalculated
NonCompliantInfusionDetected
ComplianceFindingCreated
ComplianceFindingResolved
ComplianceDefinitionChanged
ComplianceDataRefreshed
```

#### Final Reference Data Context API Portfolio
```text
POST /api/cqi/v1/compliance/summary-query
POST /api/cqi/v1/compliance/breakdown-query
POST /api/cqi/v1/compliance/trends-query
POST /api/cqi/v1/compliance/infusions-query

GET  /api/cqi/v1/compliance/infusions/{infusionId}
GET  /api/cqi/v1/compliance/infusions/{infusionId}/evaluation

POST /api/cqi/v1/compliance/exports
GET  /api/cqi/v1/compliance/exports/{exportId}
```

This would be the first bounded context implemented, because every other context (Compliance, Limits, Guardian, Infusion Story, Device Usage) depends on it but it has almost no dependency on them. It is the cleanest place to establish ubiquitous language and API governance before the report-centric APIs are designed.

---
---

### limits-api
(TBD)

### dose-rate-change-api
(TBD)

### device-usage-api
(TBD)

### infusion-story-api
(TBD)

### guardian-alert-api
(TBD)

### report-preferences-api
(TBD)

### data-quality-api
(TBD)

# API Contract

## Contract Meta Data

```text

Each API Contract will cover the following elements:
API name
Business purpose
Consumer
Bounded context
Request parameters
Response DTO
Sort / pagination rules
Security / authorization rule
Data freshness expectation
Traceability to requirement / report / UI workflow
Backing read model
Contract version

```

## API Contract Structure

1. Purpose
2. Scope
3. Design goals
   - Contract-first
   - Storage-agnostic
   - UI-friendly
   - Report-capability aligned
   - Versioned and backward compatible
4. Non-goals
   - No direct SQL exposure
   - No bronze/silver/gold exposure
   - No table-shaped public APIs
5. Ubiquitous language
6. Bounded context map
7. API portfolio
8. Contract definitions
9. Request/response standards
10. Pagination/filter/sort standards
11. Error model
12. Security and authorization
13. Data freshness and lineage
14. Schema evolution strategy
15. Traceability to CQI requirements/reports
16. Migration approach from current CQI API
17. Open questions

---
---
