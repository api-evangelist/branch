# Branch Insurance GraphQL API

## Overview

Branch Insurance (ourbranch.com) is an InsurTech platform that enables partners to embed home, auto, renters, and umbrella insurance directly into their own workflows. Unlike many insurance providers that rely on REST APIs, Branch has built their partner integration layer natively on GraphQL, offering a Quote to Bind API that covers the complete insurance lifecycle.

## Platform Description

Branch Insurance provides bundled policy quoting, binding, payment processing, and policy management through a single GraphQL endpoint. Partners — such as mortgage lenders, real estate platforms, car dealerships, and financial services companies — can embed Branch's insurance products directly into their customer-facing workflows. The GraphQL API exposes the full insurance value chain:

- **Quoting**: Generate real-time home, auto, renters, and umbrella insurance quotes based on applicant data, property characteristics, and vehicle information
- **Binding**: Convert accepted quotes into active policies with a single mutation
- **Payment Processing**: Handle premium collection, recurring billing, and payment method management
- **Policy Management**: Retrieve policy documents, manage coverage changes, and handle endorsements
- **Document Retrieval**: Access declarations pages, certificates of insurance, and other policy documents

## Why GraphQL

Branch chose GraphQL for their Quote to Bind API because insurance data is highly relational and partner use cases vary significantly. A mortgage lender embedding homeowners insurance needs different fields than a car dealership embedding auto coverage. GraphQL's field-level selection allows partners to request exactly the data their workflow requires without over-fetching, which is especially valuable when quoting decisions involve dozens of rating factors and coverage options.

GraphQL also simplifies the iteration cycle for insurance products — Branch can add new coverage types, endorsements, and rating factors to the schema without breaking existing partner integrations.

## API Endpoint

- **Production**: `https://v2.api.ourbranch.com/`
- **Staging / Playground**: `https://staging.v2.api.ourbranch.com/`
- **Documentation**: `https://docs.v2.api.ourbranch.com/`

## Authentication

Partners authenticate using API keys issued by Branch. Keys are passed as HTTP headers on each request. Branch operates a separate staging environment for integration development and testing.

## Schema Overview

The conceptual GraphQL schema for Branch Insurance covers the following domain areas:

### Core Insurance Entities

- **Quote**: A priced insurance offer for a specific risk, valid for a limited time
- **Policy**: An active insurance contract resulting from a bound quote
- **Applicant**: The person seeking insurance coverage
- **Property**: Real property being insured (home, condo, rental unit)
- **Vehicle**: Automobile being insured under an auto policy
- **Driver**: A person covered under an auto policy
- **Coverage**: A specific coverage type and limit within a policy
- **Premium**: Pricing breakdown including base premium, fees, and taxes

### Product Lines

- **HomeQuote / HomePolicy**: Homeowners insurance products
- **AutoQuote / AutoPolicy**: Personal auto insurance products
- **RentersQuote / RentersPolicy**: Renters insurance products
- **UmbrellaQuote / UmbrellaPolicy**: Umbrella/excess liability products
- **BundleQuote**: Multi-line quote combining home and auto into a single bundled offer

### Workflow Types

- **QuoteRequest**: Input type capturing all data needed to generate a quote
- **BindRequest**: Input type to convert a quote to an active policy
- **PaymentMethod**: Credit card, bank account, or other payment instrument
- **BillingSchedule**: Monthly, semi-annual, or annual payment cadence

### Supporting Entities

- **Address**: Standardized property or mailing address
- **Lender**: Mortgage lender or lienholder for escrow billing
- **LossHistory**: Prior claims history for rating purposes
- **CreditProfile**: Insurance credit score tier (not raw credit score)
- **Endorsement**: Optional coverage add-on or modification
- **Document**: Policy document with download URL
- **Agent**: Licensed insurance agent or broker, if applicable
- **Partner**: The integrated partner platform consuming the API

## Key Queries

- `getQuote(id: ID!)`: Retrieve a specific quote by ID
- `listQuotes(filter: QuoteFilter!)`: List quotes for a partner or applicant
- `getPolicy(id: ID!)`: Retrieve an active policy
- `listPolicies(filter: PolicyFilter!)`: List policies with filtering
- `getDocument(id: ID!)`: Retrieve a policy document
- `getApplicant(id: ID!)`: Retrieve applicant profile
- `getPartner(id: ID!)`: Retrieve partner account information

## Key Mutations

- `createQuote(input: QuoteRequest!)`: Generate a new insurance quote
- `updateQuote(id: ID!, input: QuoteUpdateInput!)`: Modify a quote before binding
- `bindQuote(id: ID!, input: BindRequest!)`: Convert a quote to an active policy
- `addPaymentMethod(input: PaymentMethodInput!)`: Add a payment method
- `processPayment(input: PaymentInput!)`: Process a premium payment
- `requestEndorsement(policyId: ID!, input: EndorsementInput!)`: Request a coverage change
- `cancelPolicy(policyId: ID!, input: CancellationInput!)`: Cancel an active policy
- `retrieveDocument(policyId: ID!, type: DocumentType!)`: Generate and retrieve a document

## Coverage Types

Branch Insurance supports the following coverage categories within their schema:

**Homeowners**
- Dwelling (Coverage A)
- Other Structures (Coverage B)
- Personal Property (Coverage C)
- Loss of Use (Coverage D)
- Personal Liability (Coverage E)
- Medical Payments (Coverage F)
- Water Backup
- Equipment Breakdown
- Identity Fraud

**Auto**
- Bodily Injury Liability
- Property Damage Liability
- Uninsured/Underinsured Motorist
- Comprehensive
- Collision
- Medical Payments / PIP
- Roadside Assistance
- Rental Reimbursement
- Gap Coverage

**Umbrella**
- Excess Liability
- Umbrella Personal Liability

## Rating Factors

The schema captures the primary rating inputs used by Branch's underwriting engine:

- Property characteristics: year built, square footage, construction type, roof age, roof material
- Location data: address, county, fire district, flood zone, distance to coast
- Loss history: prior claims within 5 years, claim types, claim amounts
- Credit profile: insurance credit tier
- Vehicle data: year, make, model, VIN, annual mileage, garaging address
- Driver data: age, license status, driving history, prior violations

## Integration Patterns

Partners typically implement Branch integration in one of three patterns:

1. **Embedded Quote Flow**: Partner collects applicant data in their own UI, calls `createQuote`, displays Branch quote results, and calls `bindQuote` upon customer acceptance.

2. **Pre-Fill Flow**: Partner passes known data (from a mortgage application, for example) to pre-populate the quote, reducing friction for the end customer.

3. **Comparison Flow**: Partner generates multiple quotes across providers and surfaces Branch alongside others as a bundled option.

## Schema File

See `branch-schema.graphql` in this directory for the full conceptual GraphQL schema derived from Branch Insurance's Quote to Bind API domain model.
