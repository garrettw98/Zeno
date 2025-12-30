# Citizen's Royalty Payment System Architecture
## The Human Standard: Technical Specifications for National Citizen's Royalty Delivery

**Version**: 1.0
**Last Updated**: 2025
**Classification**: Technical Specification

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [System Overview](#system-overview)
3. [Architecture Components](#architecture-components)
4. [Enrollment System](#enrollment-system)
5. [Payment Processing Engine](#payment-processing-engine)
6. [Identity and Verification](#identity-and-verification)
7. [Financial Rails Integration](#financial-rails-integration)
8. [Data Architecture](#data-architecture)
9. [Security Architecture](#security-architecture)
10. [API Specifications](#api-specifications)
11. [Disaster Recovery](#disaster-recovery)
12. [Performance Requirements](#performance-requirements)
13. [Compliance Framework](#compliance-framework)

---

## Executive Summary

This document provides comprehensive technical specifications for the Citizen's Royalty payment system as outlined in The Human Standard. The system must reliably deliver monthly payments to approximately 260 million adult Americans while maintaining security, accessibility, and transparency.

### Key Design Principles

1. **Reliability First**: 99.99% uptime for payment processing
2. **Universal Accessibility**: Serve all Americans, including unbanked populations
3. **Security by Design**: Zero tolerance for fraud or unauthorized access
4. **Privacy Protection**: Minimal data collection, maximum protection
5. **Scalability**: Handle 260M+ recipients with growth capacity
6. **Transparency**: Full auditability while protecting individual privacy
7. **Resilience**: Continue operations during regional disasters

### System Scale

| Metric | Value |
|--------|-------|
| Total Recipients | 260,000,000 |
| Monthly Payments | 260,000,000 |
| Annual Payment Volume | $3.12 Trillion |
| Peak Payments/Second | 50,000 |
| Data Records | 500M+ |
| API Calls/Day | 100M+ |

---

## System Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Citizen's Royalty PAYMENT SYSTEM                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐       │
│  │   ENROLLMENT    │     │    IDENTITY     │     │   ELIGIBILITY   │       │
│  │     PORTAL      │────▶│   VERIFICATION  │────▶│    ENGINE       │       │
│  │                 │     │     SERVICE     │     │                 │       │
│  └─────────────────┘     └─────────────────┘     └─────────────────┘       │
│           │                       │                       │                 │
│           └───────────────────────┼───────────────────────┘                 │
│                                   │                                         │
│                                   ▼                                         │
│                    ┌─────────────────────────────┐                          │
│                    │      RECIPIENT DATABASE     │                          │
│                    │   (Encrypted, Distributed)  │                          │
│                    └─────────────────────────────┘                          │
│                                   │                                         │
│                                   ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │                    PAYMENT PROCESSING ENGINE                     │       │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐    │       │
│  │  │ Scheduling│  │Calculation │  │  Routing  │  │ Settlement│    │       │
│  │  │  Service  │──│  Service   │──│  Service  │──│  Service  │    │       │
│  │  └───────────┘  └───────────┘  └───────────┘  └───────────┘    │       │
│  └─────────────────────────────────────────────────────────────────┘       │
│                                   │                                         │
│           ┌───────────────────────┼───────────────────────┐                 │
│           │                       │                       │                 │
│           ▼                       ▼                       ▼                 │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐       │
│  │   ACH / RTP     │     │   DEBIT CARD    │     │    DIGITAL      │       │
│  │   BANKING       │     │   NETWORK       │     │    WALLET       │       │
│  └─────────────────┘     └─────────────────┘     └─────────────────┘       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Technology Stack

```yaml
Infrastructure:
  Cloud Provider: AWS GovCloud / Azure Government (Multi-cloud)
  Kubernetes: EKS / AKS for container orchestration
  Service Mesh: Istio for service-to-service communication
  Load Balancing: AWS ALB / Azure Front Door

Backend Services:
  Primary Language: Go (high-performance services)
  Secondary Language: Python (data processing, ML)
  Framework: gRPC for internal services, REST for external APIs
  Message Queue: Apache Kafka (event streaming)
  Job Scheduler: Temporal (workflow orchestration)

Data Layer:
  Primary Database: CockroachDB (distributed SQL)
  Document Store: MongoDB (application data)
  Cache: Redis Cluster
  Search: Elasticsearch
  Data Warehouse: Snowflake (analytics)
  Streaming: Apache Flink (real-time processing)

Security:
  Secrets Management: HashiCorp Vault
  PKI: AWS Private CA / Azure Key Vault
  WAF: AWS WAF / Cloudflare
  DDoS Protection: AWS Shield Advanced
  SIEM: Splunk Enterprise Security

Monitoring:
  APM: Datadog
  Logging: ELK Stack (Elasticsearch, Logstash, Kibana)
  Metrics: Prometheus + Grafana
  Tracing: Jaeger
  Alerting: PagerDuty
```

---

## Architecture Components

### Microservices Architecture

```
Citizen's Royalty System Microservices:

Enrollment Domain:
├── enrollment-portal-api        # Public API for enrollment
├── enrollment-workflow-service  # Enrollment state machine
├── document-processing-service  # ID/document scanning
├── address-verification-service # Address validation
└── enrollment-notification      # SMS/email notifications

Identity Domain:
├── identity-verification-api    # External verification gateway
├── biometric-service           # Facial recognition, liveness
├── document-ocr-service        # Document text extraction
├── fraud-detection-service     # Real-time fraud signals
└── identity-resolution-service  # Deduplication, linking

Eligibility Domain:
├── eligibility-engine          # Rules processing
├── residency-verification      # Address/residency checks
├── age-verification            # Birth date validation
├── exclusion-check-service     # Incarceration, death, etc.
└── eligibility-appeal-service  # Appeal processing

Payment Domain:
├── payment-orchestrator        # Master payment coordinator
├── payment-calculation-service # Amount computation
├── payment-routing-service     # Channel selection
├── ach-gateway-service         # ACH integration
├── rtp-gateway-service         # Real-Time Payments
├── card-gateway-service        # Debit card processing
├── wallet-gateway-service      # Digital wallet integration
└── payment-reconciliation      # Settlement matching

Account Domain:
├── recipient-account-service   # Recipient records
├── payment-method-service      # Payment preferences
├── notification-preferences    # Communication settings
├── account-history-service     # Transaction history
└── account-update-service      # Change processing

Reporting Domain:
├── analytics-service           # Aggregate statistics
├── audit-service              # Compliance reporting
├── fraud-reporting-service     # Fraud metrics
└── operational-dashboard       # System health
```

### Service Communication

```yaml
Internal Communication:

Synchronous (gRPC):
  - Enrollment to Identity Verification
  - Payment Calculation to Eligibility Engine
  - Account Service to Payment Routing

  Configuration:
    timeout: 5 seconds
    retry: 3 attempts with exponential backoff
    circuit_breaker: 50% failure rate triggers

Asynchronous (Kafka):
  Topics:
    - enrollment.applications.submitted
    - enrollment.applications.approved
    - enrollment.applications.rejected
    - identity.verification.completed
    - eligibility.status.changed
    - payment.scheduled
    - payment.initiated
    - payment.completed
    - payment.failed
    - account.updated
    - fraud.alert.raised

  Configuration:
    partitions: 100 per topic (scale factor)
    replication: 3
    retention: 90 days (production data)
    retention: 7 years (audit topics)
```

---

## Enrollment System

### Enrollment Portal Architecture

```
Enrollment Portal:

Frontend:
├── Web Application (React)
│   ├── Responsive design (mobile-first)
│   ├── Accessibility: WCAG 2.1 AAA
│   ├── Languages: 20+ supported
│   └── Offline capability (PWA)
├── Native iOS App
├── Native Android App
└── IVR Phone System (1-800-Citizen's Royalty-ENROLL)

Backend:
├── API Gateway (Kong)
│   ├── Rate limiting: 100 requests/minute/user
│   ├── Authentication: Login.gov integration
│   └── Input validation: Strict schema enforcement
├── Enrollment Service
│   ├── Application submission
│   ├── Document upload
│   ├── Status tracking
│   └── Appeal processing
└── Supporting Services
    ├── Document storage (S3 with encryption)
    ├── Notification service
    └── Analytics tracking
```

### Enrollment Workflow

```python
class EnrollmentWorkflow:
    """
    State machine for Citizen's Royalty enrollment process
    """

    STATES = [
        "INITIATED",
        "IDENTITY_PENDING",
        "IDENTITY_VERIFIED",
        "IDENTITY_FAILED",
        "ELIGIBILITY_PENDING",
        "ELIGIBLE",
        "INELIGIBLE",
        "PAYMENT_SETUP_PENDING",
        "ACTIVE",
        "SUSPENDED",
        "TERMINATED"
    ]

    TRANSITIONS = {
        "INITIATED": ["IDENTITY_PENDING"],
        "IDENTITY_PENDING": ["IDENTITY_VERIFIED", "IDENTITY_FAILED"],
        "IDENTITY_VERIFIED": ["ELIGIBILITY_PENDING"],
        "IDENTITY_FAILED": ["IDENTITY_PENDING", "TERMINATED"],
        "ELIGIBILITY_PENDING": ["ELIGIBLE", "INELIGIBLE"],
        "ELIGIBLE": ["PAYMENT_SETUP_PENDING"],
        "INELIGIBLE": ["ELIGIBILITY_PENDING", "TERMINATED"],
        "PAYMENT_SETUP_PENDING": ["ACTIVE"],
        "ACTIVE": ["SUSPENDED", "TERMINATED"],
        "SUSPENDED": ["ACTIVE", "TERMINATED"]
    }

    SLA_TARGETS = {
        "IDENTITY_PENDING": timedelta(minutes=15),  # Automated
        "ELIGIBILITY_PENDING": timedelta(hours=24),  # With manual review
        "PAYMENT_SETUP_PENDING": timedelta(hours=4)  # Account setup
    }

    async def process_application(self, application_id: str):
        application = await self.get_application(application_id)

        while application.state not in ["ACTIVE", "TERMINATED"]:
            next_state = await self.determine_next_state(application)
            await self.transition(application, next_state)

            if self.requires_manual_review(application):
                await self.queue_for_review(application)
                break

        return application
```

### Document Processing

```yaml
Document Processing Pipeline:

Accepted Documents:
  Primary ID:
    - US Passport
    - State Driver's License
    - State ID Card
    - Military ID
    - Permanent Resident Card
    - Employment Authorization Document

  Proof of Residency:
    - Utility bill (within 90 days)
    - Bank statement (within 90 days)
    - Lease agreement
    - Voter registration
    - Government correspondence

Processing Steps:
  1. Upload:
     - Max size: 10MB
     - Formats: JPEG, PNG, PDF
     - Encryption: Client-side before upload

  2. Quality Check:
     - Resolution: minimum 300 DPI
     - Clarity: automated blur detection
     - Completeness: all corners visible

  3. OCR Extraction:
     - AWS Textract for document parsing
     - Custom models for ID-specific fields
     - Confidence threshold: 95%

  4. Verification:
     - Cross-reference with issuing authority
     - Fraud detection patterns
     - Human review for edge cases

  5. Storage:
     - Encrypted at rest (AES-256)
     - Retention: 7 years (audit requirement)
     - Access: Strict role-based control
```

---

## Payment Processing Engine

### Payment Orchestration

```go
package payment

import (
    "context"
    "time"
)

// PaymentOrchestrator coordinates the payment lifecycle
type PaymentOrchestrator struct {
    scheduler      *SchedulingService
    calculator     *CalculationService
    router         *RoutingService
    settlement     *SettlementService
    reconciliation *ReconciliationService
}

// MonthlyPaymentCycle executes the complete monthly payment cycle
func (o *PaymentOrchestrator) MonthlyPaymentCycle(ctx context.Context, month time.Month, year int) error {

    // Phase 1: Calculate payments (Day 1-3)
    payments, err := o.calculator.CalculateAllPayments(ctx, month, year)
    if err != nil {
        return fmt.Errorf("calculation failed: %w", err)
    }

    // Phase 2: Route payments by method (Day 3-5)
    batches, err := o.router.CreatePaymentBatches(ctx, payments)
    if err != nil {
        return fmt.Errorf("routing failed: %w", err)
    }

    // Phase 3: Submit to payment rails (Day 5-10)
    for _, batch := range batches {
        if err := o.submitBatch(ctx, batch); err != nil {
            o.handleBatchFailure(ctx, batch, err)
        }
    }

    // Phase 4: Monitor settlement (Day 10-15)
    if err := o.settlement.MonitorSettlement(ctx, month, year); err != nil {
        return fmt.Errorf("settlement monitoring failed: %w", err)
    }

    // Phase 5: Reconcile (Day 15-20)
    discrepancies, err := o.reconciliation.Reconcile(ctx, month, year)
    if err != nil {
        return fmt.Errorf("reconciliation failed: %w", err)
    }

    // Phase 6: Handle discrepancies
    for _, d := range discrepancies {
        o.handleDiscrepancy(ctx, d)
    }

    return nil
}

// PaymentMethod determines routing for a recipient
type PaymentMethod int

const (
    ACH PaymentMethod = iota
    RealTimePayments
    DebitCard
    DigitalWallet
    PaperCheck // Last resort
)

// RoutingDecision contains payment routing information
type RoutingDecision struct {
    RecipientID   string
    Method        PaymentMethod
    AccountInfo   EncryptedAccountInfo
    Priority      int
    FallbackChain []PaymentMethod
}
```

### Payment Calculation Service

```python
from dataclasses import dataclass
from decimal import Decimal
from typing import Optional
import datetime

@dataclass
class PaymentCalculation:
    recipient_id: str
    base_amount: Decimal
    adjustments: list['Adjustment']
    final_amount: Decimal
    calculation_date: datetime.date
    effective_month: str

@dataclass
class Adjustment:
    type: str
    amount: Decimal
    reason: str
    reference: Optional[str]

class PaymentCalculationService:
    """
    Calculate Citizen's Royalty payments for all eligible recipients
    """

    # Base monthly payment (Year 1 = $500, increasing to $2,000)
    BASE_AMOUNTS = {
        2026: Decimal("500.00"),
        2027: Decimal("750.00"),
        2028: Decimal("1000.00"),
        2029: Decimal("1250.00"),
        2030: Decimal("1500.00"),
        2031: Decimal("1750.00"),
        2032: Decimal("2000.00"),
    }

    def calculate_payment(
        self,
        recipient: 'Recipient',
        month: int,
        year: int
    ) -> PaymentCalculation:
        """
        Calculate individual payment amount
        """
        base = self.BASE_AMOUNTS.get(year, Decimal("2000.00"))
        adjustments = []

        # Apply cost-of-living adjustment
        cola = self.get_cola_adjustment(recipient.state, year)
        if cola != Decimal("0"):
            adjustments.append(Adjustment(
                type="COLA",
                amount=cola,
                reason=f"Cost-of-living adjustment for {recipient.state}",
                reference=None
            ))

        # Apply any garnishments (limited by law)
        garnishment = self.get_garnishment(recipient.id, month, year)
        if garnishment:
            adjustments.append(garnishment)

        # Calculate final amount
        final = base + sum(adj.amount for adj in adjustments)
        final = max(final, Decimal("0"))  # Cannot go negative

        return PaymentCalculation(
            recipient_id=recipient.id,
            base_amount=base,
            adjustments=adjustments,
            final_amount=final,
            calculation_date=datetime.date.today(),
            effective_month=f"{year}-{month:02d}"
        )

    def get_garnishment(
        self,
        recipient_id: str,
        month: int,
        year: int
    ) -> Optional[Adjustment]:
        """
        Apply court-ordered garnishments (max 25% per federal law)
        """
        orders = self.garnishment_repo.get_active_orders(recipient_id)
        if not orders:
            return None

        base = self.BASE_AMOUNTS.get(year, Decimal("2000.00"))
        max_garnishment = base * Decimal("0.25")  # 25% cap

        total_garnishment = min(
            sum(order.monthly_amount for order in orders),
            max_garnishment
        )

        return Adjustment(
            type="GARNISHMENT",
            amount=-total_garnishment,
            reason="Court-ordered garnishment",
            reference=orders[0].case_number
        )
```

### Payment Batching

```yaml
Payment Batch Configuration:

ACH Batches:
  batch_size: 50,000 payments
  submission_window: 6:00 AM - 4:00 PM ET
  settlement: T+1 (next business day)
  file_format: NACHA

  Timing:
    - Day 1-3: Batch creation
    - Day 5: Submission to Federal Reserve
    - Day 6: Settlement
    - Day 7-10: Return processing

Real-Time Payments:
  batch_size: N/A (individual)
  submission_window: 24/7
  settlement: Immediate
  priority: High (expedited needs)

  Use Cases:
    - First-time recipients
    - Returned ACH remediation
    - Emergency payments

Debit Card Loads:
  batch_size: 100,000 cards
  submission_window: 8:00 AM - 6:00 PM ET
  settlement: Same day
  processor: Treasury prepaid card provider

  Card Features:
    - No fees to recipient
    - ATM access (network ATMs)
    - Point-of-sale acceptance
    - Direct deposit capable

Digital Wallet:
  batch_size: Varies by provider
  settlement: Provider-dependent

  Supported Wallets:
    - PayPal
    - Venmo
    - Cash App
    - Zelle
    - Apple Pay (via bank)
    - Google Pay (via bank)
```

---

## Identity and Verification

### Identity Verification Architecture

```
Identity Verification System:

Layer 1: Document Verification
├── OCR extraction (AWS Textract)
├── Template matching (ID format validation)
├── Security feature detection (holograms, microprint)
├── Cross-reference with DMV databases
└── International document support

Layer 2: Biometric Verification
├── Facial recognition (1:1 matching)
├── Liveness detection (anti-spoofing)
├── Voice biometrics (phone enrollment)
└── Consent management

Layer 3: Knowledge-Based Authentication
├── Credit bureau questions (limited use)
├── Government record verification
├── Carrier/utility verification
└── Social graph analysis (optional)

Layer 4: Behavioral Analysis
├── Device fingerprinting
├── Behavioral biometrics
├── Location consistency
└── Velocity checks

Integration Partners:
├── Login.gov (primary authentication)
├── Social Security Administration (SSN verification)
├── Department of State (passport verification)
├── State DMV networks (driver's license)
├── Credit bureaus (knowledge-based auth)
└── Commercial identity providers (backup)
```

### Fraud Detection

```python
class FraudDetectionService:
    """
    Real-time and batch fraud detection for Citizen's Royalty system
    """

    RISK_SIGNALS = {
        # Application Signals
        "multiple_apps_same_device": 50,
        "multiple_apps_same_ip": 30,
        "document_previously_used": 80,
        "synthetic_identity_indicators": 70,
        "address_velocity_high": 40,
        "phone_recently_ported": 35,

        # Biometric Signals
        "facial_match_low_confidence": 45,
        "liveness_check_failed": 90,
        "voice_mismatch": 55,

        # Behavioral Signals
        "session_anomaly": 25,
        "device_fingerprint_mismatch": 35,
        "geolocation_impossible": 60,

        # External Signals
        "ssn_on_death_master_file": 95,
        "identity_theft_report": 85,
        "incarceration_record": 75,
        "duplicate_enrollment_attempt": 80
    }

    RISK_THRESHOLDS = {
        "auto_approve": 20,     # Below this = auto-approve
        "manual_review": 50,    # 20-50 = manual review
        "auto_reject": 80       # Above this = auto-reject
    }

    async def evaluate_application(
        self,
        application: 'Application'
    ) -> 'FraudEvaluation':
        """
        Evaluate application for fraud risk
        """
        signals = await self.collect_signals(application)
        risk_score = self.calculate_risk_score(signals)

        decision = self.make_decision(risk_score)

        # Log for model training
        await self.log_evaluation(application.id, signals, risk_score, decision)

        return FraudEvaluation(
            application_id=application.id,
            risk_score=risk_score,
            signals=signals,
            decision=decision,
            requires_review=decision == "MANUAL_REVIEW"
        )

    def calculate_risk_score(self, signals: dict) -> int:
        """
        Weighted sum of detected risk signals
        """
        score = 0
        for signal, detected in signals.items():
            if detected and signal in self.RISK_SIGNALS:
                score += self.RISK_SIGNALS[signal]
        return min(score, 100)  # Cap at 100
```

### Deduplication System

```sql
-- Identity Resolution Schema

CREATE TABLE identity_clusters (
    cluster_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    canonical_identity_id UUID REFERENCES identities(id),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    confidence_score DECIMAL(3,2)
);

CREATE TABLE identity_cluster_members (
    cluster_id UUID REFERENCES identity_clusters(cluster_id),
    identity_id UUID REFERENCES identities(id),
    link_type VARCHAR(50),  -- 'exact_ssn', 'fuzzy_name', 'shared_address', etc.
    link_score DECIMAL(3,2),
    linked_at TIMESTAMP DEFAULT NOW(),
    PRIMARY KEY (cluster_id, identity_id)
);

-- Matching rules for deduplication
CREATE TABLE matching_rules (
    rule_id SERIAL PRIMARY KEY,
    rule_name VARCHAR(100),
    rule_type VARCHAR(50),  -- 'deterministic', 'probabilistic'
    field_weights JSONB,
    threshold DECIMAL(3,2),
    active BOOLEAN DEFAULT true
);

-- Example matching rule configuration
INSERT INTO matching_rules (rule_name, rule_type, field_weights, threshold) VALUES
(
    'exact_ssn_match',
    'deterministic',
    '{"ssn": 1.0}',
    1.0
),
(
    'fuzzy_name_dob_address',
    'probabilistic',
    '{"first_name": 0.2, "last_name": 0.3, "dob": 0.3, "address": 0.2}',
    0.85
);

-- Deduplication audit log
CREATE TABLE dedup_decisions (
    decision_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    identity_1 UUID REFERENCES identities(id),
    identity_2 UUID REFERENCES identities(id),
    rule_id INT REFERENCES matching_rules(rule_id),
    match_score DECIMAL(3,2),
    decision VARCHAR(20),  -- 'MERGE', 'NOT_MATCH', 'MANUAL_REVIEW'
    decided_by VARCHAR(50),  -- 'SYSTEM' or user_id
    decided_at TIMESTAMP DEFAULT NOW(),
    notes TEXT
);
```

---

## Financial Rails Integration

### ACH Integration

```yaml
ACH Integration Specification:

NACHA File Format:
  file_header:
    priority_code: "01"
    immediate_destination: "Federal Reserve routing"
    immediate_origin: "Treasury routing number"
    file_creation_date: "YYMMDD"
    file_id_modifier: "A-Z, 0-9"

  batch_header:
    service_class_code: "220"  # Credits only
    company_name: "US TREASURY Citizen's Royalty"
    company_discretionary_data: ""
    company_identification: "Treasury EIN"
    standard_entry_class: "PPD"  # Prearranged Payment
    company_entry_description: "Citizen's Royalty PAYMENT"

  entry_detail:
    transaction_code: "22"  # Credit to checking
    rdfi_identification: "8-digit routing"
    check_digit: "calculated"
    dfi_account_number: "recipient account"
    amount: "payment amount in cents"
    individual_id: "recipient Citizen's Royalty ID"
    individual_name: "recipient name"

Settlement:
  network: FedACH / EPN
  settlement_time: T+1
  cutoff: 2:15 PM ET (FedACH)

Return Handling:
  return_codes:
    R01: "Insufficient Funds" (N/A for credits)
    R02: "Account Closed"
    R03: "No Account"
    R04: "Invalid Account Number"
    R05: "Unauthorized Debit" (N/A)
    R06: "Returned per ODFI Request"
    R07: "Authorization Revoked"
    R08: "Payment Stopped"
    R09: "Uncollected Funds" (N/A)
    R10: "Customer Advises Unauthorized"

  return_window: 2 business days (most returns)
  late_returns: Up to 60 days (unauthorized)
```

### Real-Time Payments (RTP)

```go
package rtp

import (
    "context"
    "encoding/json"
    "time"
)

// RTPPayment represents a real-time payment message
type RTPPayment struct {
    MessageID        string    `json:"messageId"`
    CreationDateTime time.Time `json:"creationDateTime"`
    Amount           Amount    `json:"amount"`
    Debtor           Party     `json:"debtor"`
    DebtorAccount    Account   `json:"debtorAccount"`
    Creditor         Party     `json:"creditor"`
    CreditorAccount  Account   `json:"creditorAccount"`
    RemittanceInfo   string    `json:"remittanceInfo"`
}

type Amount struct {
    Value    string `json:"value"`    // Decimal as string
    Currency string `json:"currency"` // "USD"
}

// RTPClient handles RTP network communication
type RTPClient struct {
    endpoint   string
    credentials *TCHCredentials
    httpClient  *http.Client
}

// SendPayment initiates a real-time payment
func (c *RTPClient) SendPayment(ctx context.Context, payment *RTPPayment) (*RTPResponse, error) {
    // Build ISO 20022 pain.001 message
    msg := c.buildPain001(payment)

    // Sign message
    signedMsg, err := c.sign(msg)
    if err != nil {
        return nil, fmt.Errorf("signing failed: %w", err)
    }

    // Submit to TCH RTP network
    resp, err := c.submit(ctx, signedMsg)
    if err != nil {
        return nil, fmt.Errorf("submission failed: %w", err)
    }

    // Response received within 15 seconds
    return resp, nil
}

// RTP Message Limits
const (
    MaxRTPAmount = 1_000_000_00 // $1,000,000 in cents (network limit)
    Citizen's RoyaltyMaxRTP    = 3_000_00     // $3,000 (our limit for Citizen's Royalty)
)
```

### Prepaid Card Network

```yaml
Prepaid Card Specifications:

Card Features:
  network: Visa/Mastercard (dual network)
  type: Reloadable prepaid
  chip: EMV chip enabled
  contactless: Yes
  pin: Cardholder-selected

Fees (to recipient):
  monthly_fee: $0
  atm_withdrawal_in_network: $0
  atm_withdrawal_out_network: $0 (first 4/month)
  atm_balance_inquiry: $0
  point_of_sale: $0
  bill_pay: $0
  customer_service: $0
  inactivity_fee: $0
  card_replacement: $0 (first per year)

Limits:
  daily_atm_withdrawal: $1,000
  daily_purchase: $5,000
  monthly_load: Citizen's Royalty amount only

Card Issuance:
  provider: Treasury-contracted card issuer
  production: 10M cards/month capacity
  delivery: USPS First Class (5-7 days)
  activation: Online, phone, or mobile app

Security:
  fraud_monitoring: Real-time
  zero_liability: Yes
  freeze_capability: Instant via app
  alerts: Customizable transaction alerts
```

---

## Data Architecture

### Database Schema

```sql
-- Core Recipient Schema

CREATE TABLE recipients (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    ubi_id VARCHAR(12) UNIQUE NOT NULL,  -- Public identifier
    status VARCHAR(20) NOT NULL,
    enrolled_at TIMESTAMP,
    terminated_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Personally Identifiable Information (encrypted, separate store)
CREATE TABLE recipient_pii (
    recipient_id UUID PRIMARY KEY REFERENCES recipients(id),
    ssn_hash VARCHAR(64) NOT NULL,  -- For lookup
    ssn_encrypted BYTEA NOT NULL,   -- AES-256-GCM
    first_name_encrypted BYTEA NOT NULL,
    last_name_encrypted BYTEA NOT NULL,
    dob_encrypted BYTEA NOT NULL,
    encryption_key_id VARCHAR(50) NOT NULL,
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE recipient_addresses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    recipient_id UUID REFERENCES recipients(id),
    address_type VARCHAR(20),  -- 'mailing', 'residence'
    is_current BOOLEAN DEFAULT true,
    street_1 VARCHAR(100),
    street_2 VARCHAR(100),
    city VARCHAR(50),
    state CHAR(2),
    zip_code VARCHAR(10),
    country CHAR(2) DEFAULT 'US',
    verified_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE payment_methods (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    recipient_id UUID REFERENCES recipients(id),
    method_type VARCHAR(20) NOT NULL,  -- 'ach', 'rtp', 'card', 'wallet'
    is_primary BOOLEAN DEFAULT false,
    status VARCHAR(20) DEFAULT 'pending_verification',

    -- ACH/RTP
    routing_number_encrypted BYTEA,
    account_number_encrypted BYTEA,
    account_type VARCHAR(20),

    -- Card
    card_id VARCHAR(50),
    card_last_four CHAR(4),

    -- Wallet
    wallet_provider VARCHAR(50),
    wallet_id_encrypted BYTEA,

    encryption_key_id VARCHAR(50),
    verified_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Payment History
CREATE TABLE payments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    recipient_id UUID REFERENCES recipients(id),
    payment_method_id UUID REFERENCES payment_methods(id),

    amount_cents BIGINT NOT NULL,
    payment_month DATE NOT NULL,  -- First of month

    status VARCHAR(20) NOT NULL,
    status_reason VARCHAR(100),

    scheduled_at TIMESTAMP,
    initiated_at TIMESTAMP,
    settled_at TIMESTAMP,
    failed_at TIMESTAMP,

    external_reference VARCHAR(100),
    trace_number VARCHAR(50),

    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_payments_recipient ON payments(recipient_id);
CREATE INDEX idx_payments_month ON payments(payment_month);
CREATE INDEX idx_payments_status ON payments(status);

-- Eligibility History
CREATE TABLE eligibility_records (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    recipient_id UUID REFERENCES recipients(id),
    effective_date DATE NOT NULL,
    status VARCHAR(20) NOT NULL,  -- 'eligible', 'ineligible', 'suspended'
    reason_code VARCHAR(50),
    reason_description TEXT,
    verified_by VARCHAR(50),  -- 'SYSTEM' or user_id
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Data Encryption Strategy

```yaml
Encryption Architecture:

Data Classification:
  Highly Sensitive:
    - SSN
    - Bank account numbers
    - Biometric data
    encryption: AES-256-GCM
    key_rotation: 90 days
    access: Minimal (need-to-know)

  Sensitive:
    - Names
    - Dates of birth
    - Addresses
    encryption: AES-256-GCM
    key_rotation: 180 days
    access: Role-based

  Internal:
    - Citizen's Royalty IDs
    - Payment status
    - Enrollment status
    encryption: At rest only
    key_rotation: Annual
    access: Operational staff

Key Management:
  provider: HashiCorp Vault Enterprise
  hsm: AWS CloudHSM (FIPS 140-2 Level 3)

  Key Hierarchy:
    - Master Key: HSM-protected
    - Key Encryption Keys: Per-service
    - Data Encryption Keys: Per-record type

  Access Control:
    - Service accounts: Limited scope tokens
    - Human access: MFA required
    - Audit logging: All key operations
```

### Data Partitioning

```sql
-- Partition payments by month for performance

CREATE TABLE payments (
    id UUID NOT NULL,
    recipient_id UUID NOT NULL,
    payment_month DATE NOT NULL,
    -- ... other columns
) PARTITION BY RANGE (payment_month);

-- Create partitions for each month
CREATE TABLE payments_2026_01 PARTITION OF payments
    FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');

CREATE TABLE payments_2026_02 PARTITION OF payments
    FOR VALUES FROM ('2026-02-01') TO ('2026-03-01');

-- ... continuing for all months

-- Automated partition management
CREATE OR REPLACE FUNCTION create_monthly_partition()
RETURNS void AS $$
DECLARE
    partition_date DATE;
    partition_name TEXT;
    start_date DATE;
    end_date DATE;
BEGIN
    partition_date := DATE_TRUNC('month', NOW()) + INTERVAL '2 months';
    partition_name := 'payments_' || TO_CHAR(partition_date, 'YYYY_MM');
    start_date := partition_date;
    end_date := partition_date + INTERVAL '1 month';

    EXECUTE format(
        'CREATE TABLE IF NOT EXISTS %I PARTITION OF payments
         FOR VALUES FROM (%L) TO (%L)',
        partition_name, start_date, end_date
    );
END;
$$ LANGUAGE plpgsql;

-- Schedule monthly (run on 15th for 2 months ahead)
SELECT cron.schedule('0 0 15 * *', 'SELECT create_monthly_partition()');
```

---

## Security Architecture

### Defense in Depth

```
Security Layers:

Layer 1: Network Security
├── AWS Shield Advanced (DDoS protection)
├── WAF with custom rule sets
├── VPC isolation (public/private subnets)
├── Network ACLs and Security Groups
├── TLS 1.3 everywhere
└── Certificate pinning (mobile apps)

Layer 2: Application Security
├── Input validation (strict schemas)
├── Output encoding
├── Parameterized queries (SQL injection prevention)
├── CSRF protection
├── Rate limiting
├── Session management
└── Content Security Policy

Layer 3: Identity and Access
├── Login.gov integration (citizens)
├── PIV/CAC authentication (staff)
├── Role-based access control
├── Principle of least privilege
├── Just-in-time access for sensitive operations
└── Multi-factor authentication everywhere

Layer 4: Data Security
├── Encryption at rest (AES-256)
├── Encryption in transit (TLS 1.3)
├── Field-level encryption (PII)
├── Tokenization for sensitive fields
├── Data masking in logs
└── Secure key management (HSM)

Layer 5: Monitoring and Response
├── SIEM (Splunk Enterprise Security)
├── Real-time alerting
├── Behavioral analytics
├── Threat intelligence integration
├── Incident response automation
└── Forensic capability
```

### Zero Trust Architecture

```yaml
Zero Trust Implementation:

Principles:
  - Never trust, always verify
  - Assume breach
  - Verify explicitly
  - Use least privilege access
  - Microsegmentation

Implementation:

  Network:
    - No implicit trust based on network location
    - All traffic encrypted (mTLS between services)
    - Microsegmentation via service mesh
    - Network policies enforced via Kubernetes

  Identity:
    - Strong authentication required for all access
    - Device health verification
    - Continuous authentication (session re-validation)
    - Context-aware access decisions

  Data:
    - Data classified and labeled
    - Access decisions at data layer
    - Encryption tied to identity
    - Data loss prevention controls

  Application:
    - Service identity verification (SPIFFE)
    - API authentication for all calls
    - Request-level authorization
    - Immutable audit logs

Service Mesh Configuration:
  provider: Istio

  Authorization Policies:
    - Default deny all
    - Explicit allow for known service pairs
    - JWT validation required
    - mTLS enforced

  Example Policy:
    apiVersion: security.istio.io/v1beta1
    kind: AuthorizationPolicy
    metadata:
      name: payment-service-policy
    spec:
      selector:
        matchLabels:
          app: payment-service
      action: ALLOW
      rules:
      - from:
        - source:
            principals: ["cluster.local/ns/default/sa/payment-orchestrator"]
        to:
        - operation:
            methods: ["POST"]
            paths: ["/api/v1/payments/*"]
```

### Incident Response

```
Incident Response Plan:

Severity Levels:
  SEV-1 (Critical):
    - Payment system outage
    - Data breach confirmed
    - Fraud affecting >1000 recipients
    Response Time: 15 minutes
    Escalation: Immediate to leadership

  SEV-2 (High):
    - Payment delays affecting >10,000 recipients
    - Security vulnerability discovered
    - Data integrity issue
    Response Time: 1 hour
    Escalation: Within 2 hours

  SEV-3 (Medium):
    - Single service degradation
    - Minor fraud incident
    - Compliance issue
    Response Time: 4 hours
    Escalation: Within 24 hours

  SEV-4 (Low):
    - Non-critical bug
    - Documentation issue
    - Minor process deviation
    Response Time: Next business day

Response Procedures:

  Detection:
    ├── Automated monitoring alerts
    ├── User reports (call center, portal)
    ├── Security operations center
    └── Third-party notification

  Triage:
    ├── Severity assessment
    ├── Scope determination
    ├── Impact analysis
    └── Response team assembly

  Containment:
    ├── Isolate affected systems
    ├── Block malicious actors
    ├── Preserve evidence
    └── Communicate to stakeholders

  Eradication:
    ├── Root cause analysis
    ├── Remove threat
    ├── Patch vulnerabilities
    └── Verify removal

  Recovery:
    ├── Restore systems
    ├── Verify integrity
    ├── Monitor for recurrence
    └── Resume operations

  Post-Incident:
    ├── Incident report
    ├── Lessons learned
    ├── Process improvements
    └── Training updates
```

---

## API Specifications

### External API (Citizen Portal)

```yaml
openapi: 3.0.0
info:
  title: Citizen's Royalty Citizen Portal API
  version: 1.0.0
  description: API for citizens to manage their Citizen's Royalty enrollment and payments

servers:
  - url: https://api.ubi.gov/v1

paths:
  /enrollment:
    post:
      summary: Start a new enrollment application
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/EnrollmentRequest'
      responses:
        201:
          description: Enrollment started
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/EnrollmentResponse'
        400:
          description: Invalid request
        409:
          description: Duplicate enrollment

  /enrollment/{applicationId}:
    get:
      summary: Get enrollment application status
      parameters:
        - name: applicationId
          in: path
          required: true
          schema:
            type: string
      responses:
        200:
          description: Application status
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ApplicationStatus'

  /account:
    get:
      summary: Get recipient account information
      security:
        - bearerAuth: []
      responses:
        200:
          description: Account information
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Account'

  /account/payment-method:
    put:
      summary: Update payment method
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/PaymentMethodUpdate'
      responses:
        200:
          description: Payment method updated
        400:
          description: Invalid payment method

  /payments:
    get:
      summary: Get payment history
      security:
        - bearerAuth: []
      parameters:
        - name: from
          in: query
          schema:
            type: string
            format: date
        - name: to
          in: query
          schema:
            type: string
            format: date
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
      responses:
        200:
          description: Payment history
          content:
            application/json:
              schema:
                type: object
                properties:
                  payments:
                    type: array
                    items:
                      $ref: '#/components/schemas/Payment'
                  pagination:
                    $ref: '#/components/schemas/Pagination'

components:
  schemas:
    EnrollmentRequest:
      type: object
      required:
        - firstName
        - lastName
        - dateOfBirth
        - ssn
        - address
      properties:
        firstName:
          type: string
          maxLength: 50
        lastName:
          type: string
          maxLength: 50
        dateOfBirth:
          type: string
          format: date
        ssn:
          type: string
          pattern: '^\d{9}$'
        address:
          $ref: '#/components/schemas/Address'

    Payment:
      type: object
      properties:
        id:
          type: string
          format: uuid
        amount:
          type: number
          format: decimal
        paymentMonth:
          type: string
          format: date
        status:
          type: string
          enum: [scheduled, processing, completed, failed]
        settledAt:
          type: string
          format: date-time

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

### Internal Service API (gRPC)

```protobuf
syntax = "proto3";

package ubi.payment;

option go_package = "github.com/treasury/ubi/payment";

// Payment Service Definition
service PaymentService {
  // Schedule a payment for a recipient
  rpc SchedulePayment(SchedulePaymentRequest) returns (SchedulePaymentResponse);

  // Get payment status
  rpc GetPaymentStatus(GetPaymentStatusRequest) returns (PaymentStatus);

  // List payments for a recipient
  rpc ListPayments(ListPaymentsRequest) returns (ListPaymentsResponse);

  // Cancel a scheduled payment
  rpc CancelPayment(CancelPaymentRequest) returns (CancelPaymentResponse);

  // Retry a failed payment
  rpc RetryPayment(RetryPaymentRequest) returns (RetryPaymentResponse);
}

message SchedulePaymentRequest {
  string recipient_id = 1;
  int64 amount_cents = 2;
  string payment_month = 3;  // YYYY-MM format
  string payment_method_id = 4;
  PaymentPriority priority = 5;
}

enum PaymentPriority {
  NORMAL = 0;
  HIGH = 1;
  EXPEDITED = 2;
}

message PaymentStatus {
  string payment_id = 1;
  string recipient_id = 2;
  int64 amount_cents = 3;
  PaymentState state = 4;
  google.protobuf.Timestamp scheduled_at = 5;
  google.protobuf.Timestamp initiated_at = 6;
  google.protobuf.Timestamp completed_at = 7;
  string failure_reason = 8;
  string trace_number = 9;
}

enum PaymentState {
  UNKNOWN = 0;
  SCHEDULED = 1;
  PENDING = 2;
  PROCESSING = 3;
  COMPLETED = 4;
  FAILED = 5;
  CANCELLED = 6;
}
```

---

## Disaster Recovery

### Business Continuity Requirements

```yaml
Recovery Objectives:

RPO (Recovery Point Objective):
  tier_1_critical:
    - Payment processing: 0 (no data loss)
    - Recipient database: 5 minutes
  tier_2_important:
    - Enrollment system: 1 hour
    - Reporting: 24 hours
  tier_3_normal:
    - Analytics: 7 days

RTO (Recovery Time Objective):
  tier_1_critical:
    - Payment processing: 15 minutes
    - Recipient database: 30 minutes
  tier_2_important:
    - Enrollment system: 4 hours
    - Citizen portal: 2 hours
  tier_3_normal:
    - Reporting: 24 hours
```

### Multi-Region Architecture

```
Disaster Recovery Architecture:

Primary Region (us-east-1):
├── Active production workloads
├── Primary databases (multi-AZ)
├── All user traffic
└── Real-time replication to DR

DR Region (us-west-2):
├── Warm standby infrastructure
├── Synchronous replica for critical data
├── Asynchronous replica for other data
├── Ready to accept traffic in <15 minutes
└── Regular failover testing

Data Replication:
├── CockroachDB: Synchronous cross-region (RPO=0)
├── MongoDB: Async replication (RPO=5min)
├── Redis: Cross-region cluster (active-active)
├── S3: Cross-region replication
└── Kafka: MirrorMaker 2 (async)

Failover Procedures:
  Automatic:
    - Single AZ failure: Auto-failover within region
    - Database primary failure: Automatic promotion

  Manual (Regional Failure):
    - Decision authority: Operations Director
    - Execution time: <15 minutes
    - Verification: Automated health checks
    - Rollback: Document but rarely needed
```

### Backup Strategy

```yaml
Backup Configuration:

Database Backups:
  CockroachDB:
    full_backup: Daily at 00:00 UTC
    incremental: Hourly
    retention: 90 days
    storage: S3 cross-region
    encryption: AES-256

  MongoDB:
    full_backup: Daily at 02:00 UTC
    oplog: Continuous
    retention: 30 days
    point_in_time: 7 days

File Storage:
  S3:
    versioning: Enabled
    lifecycle: Transition to Glacier after 90 days
    retention: 7 years (regulatory)
    replication: Cross-region (us-east-1 to us-west-2)

Secrets and Keys:
  Vault:
    backup: Encrypted snapshot daily
    storage: Separate S3 bucket
    retention: 1 year

  HSM:
    key_backup: Encrypted to offline storage
    ceremony: Annual backup verification

Testing:
  restore_test: Monthly (random database)
  dr_drill: Quarterly
  full_failover: Annual
```

---

## Performance Requirements

### SLA Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| System Availability | 99.99% | Monthly |
| Payment Processing Uptime | 99.999% | Monthly (during payment window) |
| API Response Time (p50) | < 100ms | Continuous |
| API Response Time (p99) | < 500ms | Continuous |
| Payment Initiation | < 24 hours from schedule | Per payment |
| Payment Settlement | Per rail SLA | Per payment |
| Enrollment Processing | < 48 hours | Per application |
| Customer Service Wait | < 5 minutes | Average |

### Capacity Planning

```yaml
Capacity Requirements:

Peak Load Scenarios:

  Monthly Payment Processing (Day 1-5):
    concurrent_payments: 500,000
    payments_per_second: 50,000
    database_transactions: 100,000/sec

  Annual Enrollment Period:
    concurrent_sessions: 1,000,000
    applications_per_hour: 500,000
    document_uploads: 1,000,000/day

  Tax Season (April):
    portal_traffic: 5M unique visitors/day
    api_calls: 500M/day
    customer_service_calls: 500,000/day

Infrastructure Sizing:

  Compute:
    web_tier: 100 pods (m5.2xlarge equivalent)
    api_tier: 200 pods (m5.4xlarge equivalent)
    worker_tier: 150 pods (m5.2xlarge equivalent)
    ml_inference: 20 pods (g4dn.xlarge)

  Database:
    cockroachdb: 15 nodes (r5.4xlarge)
    mongodb: 9 nodes (r5.2xlarge)
    redis: 6 nodes (r5.xlarge)
    elasticsearch: 9 nodes (r5.2xlarge)

  Storage:
    s3: 50 PB capacity
    ebs: 100 TB high-performance

  Network:
    internet_bandwidth: 100 Gbps
    inter_az: 25 Gbps per AZ pair
    direct_connect: 10 Gbps to financial networks
```

---

## Compliance Framework

### Regulatory Requirements

```
Compliance Requirements:

Federal:
├── FedRAMP High (cloud authorization)
├── FISMA (federal security)
├── Privacy Act of 1974
├── E-Government Act of 2002
├── FITARA (IT acquisition)
├── Section 508 (accessibility)
└── NIST 800-53 (security controls)

Financial:
├── BSA/AML (anti-money laundering)
├── OFAC (sanctions screening)
├── Regulation E (electronic transfers)
├── PCI-DSS (if card processing in scope)
└── NACHA Operating Rules

Privacy:
├── Privacy Impact Assessment (PIA)
├── System of Records Notice (SORN)
├── Data minimization requirements
└── Individual access rights

Audit:
├── GAO audit support
├── IG audit support
├── Annual security assessment
└── Penetration testing (annual)
```

### Audit Trail Requirements

```sql
-- Comprehensive audit logging schema

CREATE TABLE audit_log (
    id BIGSERIAL PRIMARY KEY,
    event_id UUID NOT NULL,
    event_timestamp TIMESTAMP NOT NULL DEFAULT NOW(),

    -- Who
    actor_type VARCHAR(20) NOT NULL,  -- 'system', 'user', 'service'
    actor_id VARCHAR(100),
    actor_ip INET,
    actor_user_agent TEXT,

    -- What
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50) NOT NULL,
    resource_id VARCHAR(100),

    -- Details
    request_data JSONB,  -- Sanitized, no PII
    response_status INT,
    changes JSONB,  -- Before/after for updates

    -- Context
    correlation_id UUID,
    session_id VARCHAR(100),

    -- Integrity
    checksum VARCHAR(64),  -- SHA-256 of record
    previous_checksum VARCHAR(64)  -- Chain integrity
);

-- Partition by month for retention management
CREATE TABLE audit_log_2026_01 PARTITION OF audit_log
    FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');

-- Retention: 7 years per regulatory requirement
-- Older partitions moved to cold storage after 1 year
-- Checksum chain provides tamper evidence
```

---

## Document Control

**Version**: 1.0
**Status**: Draft for Review
**Last Updated**: 2025
**Owner**: Office of the Chief Technology Officer, Citizen's Royalty Administration
**Classification**: Technical Specification - Internal

---

*This technical specification provides the architectural blueprint for delivering the Citizen's Royalty to every eligible American. The system prioritizes reliability, security, and accessibility while maintaining the flexibility to evolve with changing requirements and technology.*
