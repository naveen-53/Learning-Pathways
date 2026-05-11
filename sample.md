Pricing Engine Complete Flow Description

Execution starts from:

PricingExecutorService.execute()

COMPLETE END-TO-END FLOW
```
Client Request
    ↓
PricingExecutorService.execute()
    ↓
Validate sourceData
    ↓
Validate Customer CRN
    ↓
Identify OFFER_TYPE
    ↓
 ┌─────────────────────────────┐
 │         FLAT FLOW           │
 └─────────────────────────────┘
                OR
 ┌─────────────────────────────┐
 │      COMPOSITE FLOW         │
 └─────────────────────────────┘
    ↓
Validate pricingType list
    ↓
Execute R1/R2/R4 in Parallel
    ↓
Each Pricing Type Calculates Independently
    ↓
Collect Results
    ↓
Calculate Final ROI
    ↓
ROI Validation
    ↓
Product Validation
    ↓
Validation Service
    ↓
Final Response
```
DETAILED FLOW DESCRIPTION
STEP 1 — REQUEST RECEIVED

The system receives pricing request.

Main request contains:

sourceData
pricingType
R3
STEP 2 — EXECUTION STARTS

Method:

execute(request, apiToken)

Functionality:

Creates response object
Starts validation process
Initializes pricing execution
STEP 3 — sourceData VALIDATION

System extracts:

sourceData

Validation:

sourceData must not be null
sourceData must not be empty

Failure:

Returns validation error immediately

Purpose:

Ensures request contains core pricing information
STEP 4 — CUSTOMER CRN VALIDATION

System validates:

ORG_CIF_CODE

Purpose:

Customer identification
Required for external integrations
Required for pricing lookup

Failure:

Stops execution
Returns error response
STEP 5 — OFFER TYPE IDENTIFICATION

System reads:

OFFER_TYPE

Possible values:

Offer Type	Flow
FLAT	Simple pricing
COMPOSITE	Multi-component pricing
FLAT OFFER FLOW

If offer type = FLAT:

calculateFlatOffer()

Functionality:

Direct pricing calculation
No parallel processing
No R1/R2/R4 orchestration

Flow ends after:

ROI calculation
Validation
Response generation
COMPOSITE OFFER FLOW

If offer type = COMPOSITE:

System starts composite pricing orchestration.

STEP 6 — pricingType VALIDATION

System validates:

pricingType

Purpose:

Defines which pricing calculations to execute

Each pricingType contains:

type
value

Example:

R1 → FTP
R2 → CORPORATE
R4 → MATRIX

Failure:

Returns pricing type validation error
STEP 7 — PARALLEL EXECUTION STARTS

Method:

executeCalculationInParallel()

Functionality:

Creates CompletableFuture tasks
Uses thread pool executor
Executes pricing calculations simultaneously

Purpose:

Improve performance
Reduce external API wait time
Independent pricing execution
STEP 8 — INDIVIDUAL PRICING EXECUTION

Each pricing config executes independently.

Method:

processPricingAsync()

Functionality:

Reads pricing type
Finds pricing implementation
Builds PricingRequest
Executes calculate()
R1 FLOW

If pricing type = R1:

R1RateOfInterestCalculation.calculate()
R1 TYPE IDENTIFICATION

System checks R1 value.

Possible values:

Value	Functionality
FTP	FTP table pricing
CC	Finacle pricing
MCLR	Floating benchmark pricing
REPO	Repo benchmark pricing
NOT_APPLICABLE	Return 0
R1 → FTP FLOW

Purpose:

Calculate funding transfer pricing

Flow:

Read FTP DB Data
    ↓
Filter by:
    - maturity date
    - request tenor
    - request date
    ↓
Select matching effective date
    ↓
Extract RATE_OF_INTEREST
    ↓
Return R1

Special Handling:

Multiple record validation
Latest effective date selection
R1 → CC FLOW

Purpose:

Fetch pricing from Finacle-linked facilities

Flow:

Call Finacle Facility API
    ↓
Fetch Customer Accounts
    ↓
Identify Scheme Types
    ↓
Parallel Interest Fetch
    ↓
Extract Interest Rates
    ↓
Determine R1

Special Functionality:

Parallel account processing
Scheme-type-based logic
R1 → MCLR FLOW

Purpose:

Benchmark-linked floating pricing

Flow:

Fetch Reset Frequency
    ↓
Fetch Floating Interest Table
    ↓
Benchmark = MCLR
    ↓
Fetch Interest Code
    ↓
Calculate Floating Rate
    ↓
Return R1
R1 → REPO FLOW

Same as MCLR flow.

Difference:

Benchmark = REPO
R2 FLOW

If pricing type = R2:

R2RateOfInterestCalculation.calculate()

Purpose:

Risk spread calculation
R2 EXECUTION FLOW
Call RAROC API
    ↓
Fetch Customer Rating
    ↓
Extract:
    - External Rating
      OR
    - KRAM Rating
    ↓
Fetch Minimum Spread
    ↓
Return R2

Purpose:

Risk-adjusted pricing
Credit spread calculation
R4 FLOW

If pricing type = R4:

R4RateOfInterestCalculation.calculate()

Purpose:

Product matrix spread calculation
R4 EXECUTION FLOW
Call Product Matrix API
    ↓
Fetch Segment Matrix
    ↓
Filter by:
    - tenor slab
    - amount slab
    ↓
Identify Matching R4 Key
    ↓
Extract R4 Rate
    ↓
Return R4

Purpose:

Product-specific pricing
Dynamic matrix-based spreads
R4 DYNAMIC KEY MAPPING

Matrix decides:

R4A
R4B
R4C

System dynamically selects applicable key.

No hardcoded mapping.

STEP 9 — COLLECT PARALLEL RESULTS

After all futures complete:

Successful Results
+
Error Results

are aggregated.

STEP 10 — VALIDATE CALCULATION RESULTS

Method:

validatePricingCalc()

Functionality:

Check pricing errors
Aggregate failures
Decide whether ROI can continue

If errors exist:

Error response generated

Else:

Continue to ROI calculation
STEP 11 — FINAL ROI CALCULATION

Method:

calculateCompositeOffer()
ROI CALCULATION FLOW

System extracts:

R1
R2
R3
R4

Then calculates:

TOTAL ROI = R1 + R2 + R3 + R4

Purpose:

Final pricing determination
STEP 12 — VALIDATE ROI

Method:

pricingValidation()

Purpose:

Validate calculated pricing against business rules
STEP 13 — PRODUCT VALIDATION

Flow:

Fetch Product API
    ↓
Get Segment Validation Rules
    ↓
Extract Validation IDs
    ↓
Call Validation Service

Purpose:

Business rule validation
Segment eligibility validation
Pricing policy validation
STEP 14 — VALIDATION STAGE PROCESSING

Based on:

VALIDATION_STAGE

System determines:

workflow mode
pricing flag
event type
STEP 15 — FINAL RESPONSE GENERATION

Final response contains:

ROI Details
Individual Components
Validation Results
Errors (if any)

Returned to caller.

COMPLETE BUSINESS VIEW
Request
   ↓
Validation
   ↓
Offer Identification
   ↓
Parallel Pricing Execution
   ↓
R1 Calculation
   ↓
R2 Calculation
   ↓
R4 Calculation
   ↓
ROI Aggregation
   ↓
Business Validation
   ↓
Product Validation
   ↓
Final Response
OVERALL SYSTEM PURPOSE

This pricing engine acts as:

Enterprise Loan Pricing Orchestrator

It dynamically calculates:

benchmark rates
funding spreads
risk spreads
product spreads
final ROI

using:

DB rules
APIs
matrices
validations
configurable business rules.
