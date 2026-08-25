# Threat Model - Order Workflow Service

## Overview

### Service Purpose
The Order Workflow Service is a finite state machine (FSM) based Akka application that manages complex order processing workflows. It demonstrates advanced actor state management patterns using FSM and become/unbecome patterns, providing a foundation for building reliable, stateful order processing systems with automatic state transitions, timeouts, and redelivery mechanisms.

### Service Scope
- Order state management and workflow orchestration
- Finite state machine-based order processing
- Message redelivery and reliability patterns
- State transition management for order lifecycle
- Actor-based concurrent order processing
- Timeout handling and failure recovery

## Data Flow Diagram

```
┌─────────────────┐
│  Order Sources  │
│  (API, Queue)   │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│     Order Workflow Service              │
│                                         │
│  ┌────────────────────────────────┐   │
│  │   Order FSM Actor              │   │
│  │                                │   │
│  │  ┌──────┐  ┌──────┐  ┌──────┐│   │
│  │  │ New  │─►│Process│─►│Done  ││   │
│  │  └──────┘  └──────┘  └──────┘│   │
│  │      │         │         │    │   │
│  │      ▼         ▼         ▼    │   │
│  │  [Timeout] [Retry] [Complete] │   │
│  └────────────────────────────────┘   │
│              │                         │
│              ▼                         │
│  ┌────────────────────────────────┐   │
│  │   Redelivery Handler           │   │
│  │   (Requester/Receiver)         │   │
│  └────────────────────────────────┘   │
│              │                         │
└──────────────┼─────────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│  External Systems                    │
│  - Payment Service                   │
│  - Inventory Service                 │
│  - Fulfillment Service               │
└──────────────────────────────────────┘
```

## Dependencies

### External Libraries
- **Akka Actor** (2.5.x): Core actor system
- **Akka FSM** (2.5.x): Finite state machine implementation
- **Akka Typed** (2.5.x): Type-safe actor implementation
- **Scala Standard Library** (2.12.x): Core language features

### Infrastructure Dependencies
- JVM Runtime (Java 8+)
- Message persistence layer (if configured)
- External service endpoints (payment, inventory, fulfillment)
- Event streaming platform (optional)

### Internal Dependencies
- Actor supervision hierarchy
- Message serialization
- State transition configuration

## Entry Points

1. **Order Submission API**
   - Interface: HTTP REST API / Actor messages
   - Trust Level: Authenticated customers/systems
   - Data: Order details (items, quantities, customer info, payment)
   - Validation: Order schema validation, business rules

2. **Order State Query API**
   - Interface: HTTP REST API / Actor messages
   - Trust Level: Authenticated users
   - Data: Order IDs, customer IDs
   - Validation: Authorization checks, ID validation

3. **Order Modification API**
   - Interface: HTTP REST API / Actor messages
   - Trust Level: Authenticated users/admin
   - Data: Order updates, cancellations
   - Validation: Order state validation, authorization

4. **External Service Callbacks**
   - Interface: Webhook/REST endpoints
   - Trust Level: Authenticated partner services
   - Data: Payment confirmations, inventory updates
   - Validation: Signature verification, source authentication

5. **Administrative Interface**
   - Interface: Admin console / Management API
   - Trust Level: Administrative users
   - Data: Order management operations, workflow configuration
   - Validation: Admin authentication, role-based access

6. **Internal Actor Messages**
   - Interface: Actor system messages
   - Trust Level: Internal actors
   - Data: State transitions, redelivery requests
   - Validation: Actor hierarchy validation

## Exit Points

1. **Order Status Responses**
   - Destination: API clients, customers
   - Protocol: HTTP/JSON, Actor messages
   - Data: Order status, tracking information

2. **Payment Service Requests**
   - Destination: External payment gateway
   - Protocol: HTTPS/REST
   - Data: Payment details, transaction amounts
   - Sensitivity: PCI-DSS Level 1 data

3. **Inventory Service Requests**
   - Destination: Internal inventory system
   - Protocol: HTTP/gRPC
   - Data: Product IDs, quantities, reservations

4. **Fulfillment Service Requests**
   - Destination: Warehouse/logistics systems
   - Protocol: HTTP/REST, Message Queue
   - Data: Shipping addresses, order items

5. **Order Events**
   - Destination: Event streaming platform
   - Protocol: Kafka/messaging
   - Data: Order lifecycle events, state changes

6. **Logging and Audit**
   - Destination: Log aggregation system
   - Protocol: TCP/HTTP
   - Data: Order operations, state transitions, errors
   - Sensitivity: May contain PII and financial data

7. **Monitoring/Metrics**
   - Destination: Monitoring systems
   - Data: Performance metrics, error rates, SLA metrics

## Assets

### Critical Data Assets
- **Order Data**: Customer orders with items, quantities, prices
- **Customer PII**: Names, addresses, email, phone numbers
- **Payment Information**: Payment methods, transaction IDs (not full card data)
- **Order State**: Current workflow state, transition history
- **Business Rules**: Pricing rules, validation logic, workflow configurations

### System Assets
- **FSM State**: Current state of all active orders
- **Actor Mailboxes**: Queued order operations
- **State Transition Logic**: Workflow definitions and rules
- **Redelivery Queue**: Pending retry operations

### Configuration Assets
- **Workflow Configuration**: State definitions, transition rules, timeouts
- **Integration Configuration**: External service endpoints, credentials
- **Security Configuration**: Authentication, authorization settings

## Trust Levels

### Level 1: Untrusted
- Public internet
- Unauthenticated requests
- Unknown sources

### Level 2: Authenticated Customers
- Registered users
- Verified customer accounts
- Valid session tokens

### Level 3: Partner Services
- Authenticated external services (payment, shipping)
- API consumers with valid credentials
- Third-party integrations

### Level 4: Internal Services
- Internal microservices
- Service-to-service communication
- Internal actor systems

### Level 5: Administrative
- System administrators
- DevOps personnel
- Monitoring systems

### Level 6: System Core
- Local actor system
- JVM process
- Core FSM components

## STRIDE Threat List

### Spoofing Threats
| ID | Threat | Affected Component | Severity |
|----|--------|-------------------|----------|
| S1 | Unauthorized order submission using stolen credentials | Order Submission API | High |
| S2 | Spoofed payment confirmation callbacks | Payment Integration | Critical |
| S3 | Fake fulfillment status updates | Fulfillment Integration | High |
| S4 | Actor message sender spoofing | Actor System | Medium |
| S5 | Admin impersonation | Administrative Interface | Critical |

### Tampering Threats
| ID | Threat | Affected Component | Severity |
|----|--------|-------------------|----------|
| T1 | Order amount manipulation during state transitions | FSM State Logic | Critical |
| T2 | Price tampering in order data | Order Data | Critical |
| T3 | State transition manipulation to skip payment | FSM Transitions | Critical |
| T4 | Modification of redelivery messages | Redelivery Handler | High |
| T5 | Unauthorized order cancellation | Order Modification API | High |
| T6 | Workflow configuration tampering | Configuration | High |

### Repudiation Threats
| ID | Threat | Affected Component | Severity |
|----|--------|-------------------|----------|
| R1 | Customer denying order placement | Order System | High |
| R2 | Lack of audit trail for state transitions | FSM System | High |
| R3 | Insufficient logging of order modifications | Order Management | Medium |
| R4 | No proof of payment processing | Payment Integration | High |

### Information Disclosure Threats
| ID | Threat | Affected Component | Severity |
|----|--------|-------------------|----------|
| I1 | Exposure of customer PII in logs | Logging System | Critical |
| I2 | Order data disclosure through API vulnerabilities | REST API | High |
| I3 | Payment transaction details in error messages | Error Handling | Critical |
| I4 | Unencrypted order data in transit | Network Layer | Critical |
| I5 | State information leak through timing attacks | FSM Logic | Medium |
| I6 | Database credential exposure in configuration | Configuration Files | Critical |

### Denial of Service Threats
| ID | Threat | Affected Component | Severity |
|----|--------|-------------------|----------|
| D1 | Order submission flooding | Order Submission API | High |
| D2 | Actor mailbox overflow | Actor System | High |
| D3 | Infinite redelivery loops | Redelivery Handler | Medium |
| D4 | Resource exhaustion through FSM state explosion | FSM System | High |
| D5 | Timeout manipulation causing state inconsistency | Timeout Management | Medium |
| D6 | External service unavailability cascading failure | Integration Layer | High |

### Elevation of Privilege Threats
| ID | Threat | Affected Component | Severity |
|----|--------|-------------------|----------|
| E1 | Exploiting deserialization for code execution | Message Serialization | Critical |
| E2 | Privilege escalation through order modification | Authorization | High |
| E3 | Unauthorized access to admin functions | Administrative Interface | Critical |
| E4 | Bypassing workflow validation | FSM Validation | High |
| E5 | Configuration injection for security bypass | Configuration Management | Critical |

## Countermeasures

### Spoofing Countermeasures
- **CM-S1**: Implement OAuth 2.0 / JWT-based authentication for all APIs
- **CM-S2**: Use HMAC signatures for webhook callbacks from external services
- **CM-S3**: Implement mutual TLS for service-to-service communication
- **CM-S4**: Verify callback source IP addresses against whitelist
- **CM-S5**: Multi-factor authentication for administrative access
- **CM-S6**: API key rotation policy and secure key storage
- **CM-S7**: Session management with secure tokens and expiration

### Tampering Countermeasures
- **CM-T1**: Implement cryptographic checksums for order data
- **CM-T2**: Use immutable data structures for FSM state
- **CM-T3**: Validate all state transitions against business rules
- **CM-T4**: Implement price verification from authoritative source
- **CM-T5**: Digital signatures for critical operations
- **CM-T6**: Configuration integrity checks and validation
- **CM-T7**: Database constraints and transaction integrity
- **CM-T8**: Implement optimistic locking for concurrent updates

### Repudiation Countermeasures
- **CM-R1**: Comprehensive audit logging for all order operations
- **CM-R2**: Log all FSM state transitions with timestamps and actor IDs
- **CM-R3**: Implement event sourcing for order lifecycle
- **CM-R4**: Digital signatures for order confirmations
- **CM-R5**: Persistent audit trail with tamper-evident storage
- **CM-R6**: Customer notification emails for order actions
- **CM-R7**: Integration with SIEM for centralized audit management

### Information Disclosure Countermeasures
- **CM-I1**: Mandatory TLS 1.3 for all external communications
- **CM-I2**: Encrypt sensitive data at rest using AES-256
- **CM-I3**: Implement data masking for PII in logs
- **CM-I4**: Secure error handling without exposing internal details
- **CM-I5**: Field-level encryption for sensitive order data
- **CM-I6**: Implement secrets management (HashiCorp Vault, AWS Secrets Manager)
- **CM-I7**: Regular secrets rotation
- **CM-I8**: Access controls on log files and databases
- **CM-I9**: Data classification and handling policies

### Denial of Service Countermeasures
- **CM-D1**: Implement rate limiting per customer/IP address
- **CM-D2**: Configure mailbox size limits and bounded mailboxes
- **CM-D3**: Implement circuit breakers for external service calls
- **CM-D4**: Set maximum redelivery attempts with exponential backoff
- **CM-D5**: Timeout configuration for all FSM states
- **CM-D6**: Resource monitoring and auto-scaling
- **CM-D7**: Implement request throttling and backpressure
- **CM-D8**: Use supervision strategies for actor failure handling
- **CM-D9**: Dead letter queue monitoring and alerting
- **CM-D10**: Load balancing and horizontal scaling capabilities

### Elevation of Privilege Countermeasures
- **CM-E1**: Use secure deserialization with class whitelisting
- **CM-E2**: Implement role-based access control (RBAC)
- **CM-E3**: Principle of least privilege for service accounts
- **CM-E4**: Input validation and sanitization for all entry points
- **CM-E5**: Authorization checks before state transitions
- **CM-E6**: Secure coding practices and code review
- **CM-E7**: Regular dependency updates and vulnerability scanning
- **CM-E8**: Enable Java Security Manager with restrictive policies
- **CM-E9**: API security gateway with policy enforcement
- **CM-E10**: Separate admin interface from customer-facing APIs

### Additional Security Measures
- **CM-G1**: Regular penetration testing and security assessments
- **CM-G2**: Fraud detection and anomaly detection
- **CM-G3**: Security monitoring and real-time alerting
- **CM-G4**: Incident response plan and procedures
- **CM-G5**: Business continuity and disaster recovery planning
- **CM-G6**: Regular security training for development team
- **CM-G7**: Vulnerability management program
- **CM-G8**: Security champions program
- **CM-G9**: Regular backup and recovery testing
- **CM-G10**: Integration with WAF (Web Application Firewall)

## Security Controls Summary

| Control Type | Implementation Status | Priority |
|--------------|----------------------|----------|
| Authentication | Required | Critical |
| Authorization | Required | Critical |
| Encryption (Transit) | Required | Critical |
| Encryption (At Rest) | Required | Critical |
| Audit Logging | Required | Critical |
| Input Validation | Required | Critical |
| Rate Limiting | Required | High |
| Monitoring | Required | Critical |
| Data Masking | Required | High |
| Session Management | Required | High |
| FSM Validation | Required | Critical |
| Circuit Breakers | Required | High |
| Message Signing | Recommended | High |

## Compliance Considerations

- **PCI-DSS**: Payment card data handling (if applicable)
- **GDPR**: Customer PII protection, right to erasure, data portability
- **SOC 2 Type II**: Security, availability, and confidentiality
- **PSD2**: Strong customer authentication for payments
- **SOX**: Financial record keeping and audit trails
- **CCPA**: California consumer privacy requirements

## Integration Security

### Payment Gateway Integration
- Use tokenization for payment data
- Never store full card numbers
- Implement PCI-DSS SAQ validation
- Use gateway SDKs with security updates

### External Service Integration
- Mutual TLS authentication
- API key management and rotation
- Request/response validation
- Timeout and retry policies
- Circuit breaker implementation

## Operational Security

### Monitoring Requirements
- Real-time order processing metrics
- FSM state distribution tracking
- Error rate monitoring
- External service health checks
- Security event monitoring
- Performance SLA tracking

### Incident Response
- Runbook for common security incidents
- Escalation procedures
- Forensics data collection
- Communication plan
- Recovery procedures

## Review and Updates

- **Document Version**: 1.0
- **Last Updated**: 2024
- **Review Frequency**: Quarterly or after significant changes
- **Next Review Date**: Q1 2025
- **Document Owner**: Security Team / Engineering Team
- **Approved By**: Chief Security Officer, VP of Engineering
- **Classification**: Confidential
- **Distribution**: Security Team, Engineering Team, Compliance Team
