## Problem Statement
- Small and medium businesses often lack reliable systems to manage recurring payments, track subscriptions, and handle billing efficiently. Manual processes lead to missed payments, inconsistent billing, and poor visibility into revenue.
- This system provides a centralized platform to automate billing, manage subscriptions, and ensure reliable payment tracking.

   
## Users
- Primary Users ( Merchants ):
    - Business such as gym's, SaaS companies, and service providers
    - They create plans, manage customers, and track payments
- Secondary Users ( Customers ):
    - End users who subscribe to services and make payments

## This Projects Scope / MVP Feature
- Merchant onboarding (tenant + user)
- Customers management
- Plan creation (monthly/yearly)
- Subscription creation
- Invoice generation
- Payment simulation (success/failure)
- basic audit logging

## Core Domain Concepts
- Tenant: Business using the platform
- User: Internal user of the tenant
- Customer: End user who pays
- Plan: Subscription offering
- Subscription: Active plan for a customer
- Invoice: Bill generated for payment
- Payment: Transaction made against invoice

## Constraints & Assumptions
- System is multi-tenant (data isolated by tenant_id)
- Payments are simulated (no real gateway i.e Sandbox will be used)
- System must handle recurring billing
- Backend-only System (API Driven for now)

## DB Diagram

![DB Diagram](docs/db-diagram.jpg)

# System Design Diagram
## First Time Payment High Level Design
- Flow Explanation
    - User initiates subscription request
    - Subscription Service creates subscription (PENDING)
    - Invoice Service creates invoice (PENDING)
    - Payment Service creates payment (INITIATED)
    - Payment request is pushed to queue
    - Worker consumes queue and calls external payment gateway
    - Payment gateway send webhook response
    - Payment Service updates:
        - Payment - SUCCESS/FAILES
        - Invoice - PAID/FAILED
        - Subscription - ACTIVE/PAST_DUE
## Key Design Decisions
- Invoice as source of Truth
    - Payment is tied to invoice
    - Subscription depends on invoice status
- Asynchronous Processing
    - Decouples request from execution
    - Improves scalability and reliability
    Uses: Amazon SQS
## Idempotency
- Ensures safe retries (queue + webhook)
- Implemented using payment_id / txn_id
## Retry & Failure Handling
- Failed messages retried automatically
- After max attempts -> moved to Dead Letter Queue (DLQ)
## Webhook-Based Confirmation
- External Payment gateway sends final status
- System does not rely on synchronous response

![First Time Payment Flow](docs/hld/First-time-payment.drawio.png)