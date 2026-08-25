# Threat Model - Fraud Service

## Overview

### Service Purpose
The Fraud Service is a distributed, highly-available Akka application utilizing Distributed Data (CRDT) technology for fraud detection and prevention. It provides low-latency voting services, shopping cart management, service registry, replicated caching, and metrics collection across a distributed cluster environment.

### Service Scope
- Real-time fraud voting and detection
- Distributed shopping cart management
- Service discovery and registry
- Distributed caching for fraud patterns
- Cluster-wide metrics collection and replication
- Conflict-free replicated data types (CRDTs) for eventual consistency

## Data Flow Diagram

```
┌─────────────────┐
│   External      │
│   Clients       │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────────────────────┐
│              Fraud Service Cluster                  │
│                                                     │
│  ┌──────────┐      ┌──────────┐      ┌──────────┐ │
│  │  Node 1  │◄────►│  Node 2  │◄────►│  Node 3  │ │
│  │          │ CRDT │          │ CRDT │          │ │
│  │  ┌────┐  │ Sync │  ┌────┐  │ Sync │  ┌────┐  │ │
│  │  │Vote│  │      │  │Cart│  │      │  │Reg │  │ │
│  │  └────┘  │      │  └────┘  │      │  └────┘  │ │
│  │  ┌────┐  │      │  ┌────┐  │      │  ┌────┐  │ │
│  │  │Cach│  │      │  │Metr│  │      │  │Data│  │ │
│  │  └────┘  │      │  └────┘  │      │  └────┘  │ │
│  └──────────┘      └──────────┘      └──────────┘ │
│         │                 │                 │      │
└─────────┼─────────────────┼─────────────────┼──────┘
          │                 │                 │
          ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────┐
│           Data Storage / Persistence                │
└─────────────────────────────────────────────────────┘
```

## Dependencies

### External Libraries
- **Akka Actor** (2.5.x): Core actor system
- **Akka Cluster** (2.5.x): Cluster membership and partitioning
- **Akka Distributed Data** (2.5.x): CRDT implementations
- **Akka Remote** (2.5.x): Remote communication
- **ScalaTest** (3.0.x): Testing framework
- **Scala Standard Library** (2.12.x): Core language features

### Infrastructure Dependencies
- JVM Runtime (Java 8+)
- Network infrastructure for cluster communication
- Gossip protocol support
- Multi-node networking

### Data Structure Dependencies
- **PNCounter**: Increment/decrement counters
- **PNCounterMap**: Counter maps for voting
- **ORSet**: Add-remove sets for service registry
- **LWWMap**: Last-writer-wins maps for shopping carts
- **Flag**: Boolean flags for state coordination
- **GSet**: Grow-only sets

## Entry Points

1. **Voting Service API**
   - Interface: Actor messages (Open, Close, Vote, GetVotes)
   - Trust Level: Authenticated users
   - Data: Participant IDs, vote counts
   - Validation: Participant validation, open/closed state checks

2. **Shopping Cart API**
   - Interface: Actor messages (AddItem, RemoveItem, GetCart)
   - Trust Level: Authenticated customers
   - Data: User IDs, product IDs, quantities, prices
   - Validation: User authentication, product validation

3. **Service Registry API**
   - Interface: Actor messages (Register, Lookup)
   - Trust Level: Internal services
   - Data: Service names, ActorRefs
   - Validation: Service authorization

4. **Cache API**
   - Interface: Actor messages (PutInCache, GetFromCache, Evict)
   - Trust Level: Internal services
   - Data: Key-value pairs for fraud patterns
   - Validation: Key format validation

5. **Metrics Collection API**
   - Interface: Internal actor messages
   - Trust Level: System components
   - Data: Heap usage, performance metrics
   - Validation: Metric type validation

6. **Cluster Gossip Protocol**
   - Interface: Akka Distributed Data replication
   - Trust Level: Cluster members
   - Data: CRDT updates, cluster state
   - Validation: Node authentication

## Exit Points

1. **Client Responses**
   - Destination: API clients
   - Protocol: Actor messages
   - Data: Vote results, cart contents, service addresses

2. **CRDT Replication**
   - Destination: Other cluster nodes
   - Protocol: Gossip protocol
   - Data: CRDT state updates

3. **Logging System**
   - Destination: Log aggregation service
   - Data: Application logs, audit trails, errors
   - Sensitivity: May contain PII and transaction data

4. **Monitoring/Metrics Export**
   - Destination: Monitoring systems (Prometheus, etc.)
   - Data: Performance metrics, cluster health

5. **Event Stream**
   - Destination: Event processing systems
   - Data: Business events, state changes

## Assets

### Critical Data Assets
- **Fraud Voting Data**: Vote counts and participant information
- **Shopping Cart Data**: Customer purchases, product selections, pricing
- **Customer PII**: User identifiers, transaction history
- **Fraud Patterns**: Cached fraud detection rules and patterns
- **Service Registry**: Internal service topology and endpoints

### System Assets
- **CRDT State**: Distributed data structures and their replicas
- **Cluster State**: Node membership, partition information
- **Actor References**: Internal communication endpoints
- **Cache Data**: Performance-critical fraud pattern cache

### Configuration Assets
- **Cluster Configuration**: Seed nodes, roles, ports
- **Security Configuration**: Authentication, authorization settings
- **Replication Settings**: Consistency levels, write/read quorums

## Trust Levels

### Level 1: Untrusted
- Public internet
- Unauthenticated requests
- Unknown external systems

### Level 2: External Authenticated
- Verified customers
- Authenticated API consumers
- Third-party integrations with valid credentials

### Level 3: Internal Services
- Microservices within the organization
- Authenticated internal systems
- Service-to-service communication

### Level 4: Cluster Members
- Authenticated cluster nodes
- Internal fraud service instances
- CRDT replication peers

### Level 5: System Core
- Local actor system
- JVM process space
- Core system components

## STRIDE Threat List

### Spoofing Threats
| ID | Threat | Affected Component | Severity |
|----|--------|-------------------|----------|
| S1 | Unauthorized node joining cluster as legitimate member | Cluster Membership | Critical |
| S2 | Spoofed voting requests inflating vote counts | Voting Service | High |
| S3 | Shopping cart hijacking through user ID spoofing | Shopping Cart | Critical |
| S4 | Fake service registration in service registry | Service Registry | High |
| S5 | Impersonation of cluster gossip messages | CRDT Replication | High |

### Tampering Threats
| ID | Threat | Affected Component | Severity |
|----|--------|-------------------|----------|
| T1 | CRDT state manipulation during gossip | Data Replication | Critical |
| T2 | Shopping cart price tampering | Shopping Cart Data | Critical |
| T3 | Fraud pattern cache poisoning | Cache Service | High |
| T4 | Vote count manipulation | Voting Service | High |
| T5 | Service registry entry modification | Service Registry | Medium |
| T6 | Metrics data tampering | Metrics Collection | Low |

### Repudiation Threats
| ID | Threat | Affected Component | Severity |
|----|--------|-------------------|----------|
| R1 | Users denying fraudulent transactions | Shopping Cart | High |
| R2 | Lack of audit trail for votes | Voting Service | Medium |
| R3 | No tracking of service registration changes | Service Registry | Medium |
| R4 | Insufficient logging of CRDT operations | Data Replication | Medium |

### Information Disclosure Threats
| ID | Threat | Affected Component | Severity |
|----|--------|-------------------|----------|
| I1 | Exposure of customer PII in shopping carts | Shopping Cart | Critical |
| I2 | Fraud pattern disclosure through cache queries | Cache Service | High |
| I3 | Voting data exposure revealing user behavior | Voting Service | High |
| I4 | Service topology disclosure | Service Registry | Medium |
| I5 | Unencrypted CRDT gossip revealing business data | Network Layer | Critical |
| I6 | Sensitive data in log files | Logging System | High |
| I7 | Metrics revealing business intelligence | Metrics Service | Medium |

### Denial of Service Threats
| ID | Threat | Affected Component | Severity |
|----|--------|-------------------|----------|
| D1 | Vote flooding overwhelming voting service | Voting Service | High |
| D2 | Shopping cart creation flood exhausting memory | Shopping Cart | High |
| D3 | Cache poisoning consuming storage | Cache Service | Medium |
| D4 | Excessive service registrations | Service Registry | Medium |
| D5 | Gossip storm overloading network | CRDT Replication | High |
| D6 | Split-brain scenarios causing data conflicts | Cluster | Critical |

### Elevation of Privilege Threats
| ID | Threat | Affected Component | Severity |
|----|--------|-------------------|----------|
| E1 | Exploiting deserialization to execute arbitrary code | Message Serialization | Critical |
| E2 | Privilege escalation through service registry manipulation | Service Registry | High |
| E3 | Unauthorized admin operations on voting service | Voting Service | High |
| E4 | Cluster admin access through configuration injection | Cluster Management | Critical |
| E5 | JVM access through actor system vulnerabilities | Runtime | Critical |

## Countermeasures

### Spoofing Countermeasures
- **CM-S1**: Implement TLS mutual authentication for cluster communication
- **CM-S2**: Enable Akka cluster security with node authentication
- **CM-S3**: Implement token-based authentication for voting API
- **CM-S4**: Use secure session management for shopping cart operations
- **CM-S5**: Implement service authentication for registry operations
- **CM-S6**: Enable cluster password protection and secret-based joining
- **CM-S7**: Validate all user IDs against authentication service

### Tampering Countermeasures
- **CM-T1**: Use CRDT merge conflict resolution with validation
- **CM-T2**: Implement cryptographic signatures for critical data updates
- **CM-T3**: Enable gossip message signing and verification
- **CM-T4**: Implement business rule validation for shopping cart operations
- **CM-T5**: Use immutable data structures with versioning
- **CM-T6**: Enable cache entry validation and integrity checks
- **CM-T7**: Implement WriteMajority/ReadMajority consistency levels

### Repudiation Countermeasures
- **CM-R1**: Comprehensive audit logging for all operations
- **CM-R2**: Implement digital signatures for non-repudiation
- **CM-R3**: Log all CRDT updates with timestamps and node IDs
- **CM-R4**: Persistent audit trail with tamper-evident storage
- **CM-R5**: Transaction logging with cryptographic chaining
- **CM-R6**: Integrate with SIEM for centralized audit management

### Information Disclosure Countermeasures
- **CM-I1**: Enable TLS encryption for all network communication
- **CM-I2**: Implement data encryption at rest for sensitive cart data
- **CM-I3**: Use data masking in logs for PII
- **CM-I4**: Implement field-level encryption for sensitive shopping cart fields
- **CM-I5**: Access control for service registry queries
- **CM-I6**: Secure log storage with access controls
- **CM-I7**: Encrypt gossip protocol messages
- **CM-I8**: Implement data classification policies

### Denial of Service Countermeasures
- **CM-D1**: Implement rate limiting for all API endpoints
- **CM-D2**: Configure mailbox size limits and backpressure
- **CM-D3**: Use circuit breakers for external dependencies
- **CM-D4**: Implement shopping cart timeout and cleanup
- **CM-D5**: Cache size limits and eviction policies
- **CM-D6**: Monitoring and alerting for resource usage
- **CM-D7**: Split-brain resolver configuration
- **CM-D8**: Network-level rate limiting and DDoS protection
- **CM-D9**: Implement request throttling per user/service

### Elevation of Privilege Countermeasures
- **CM-E1**: Use secure deserialization with class whitelisting
- **CM-E2**: Enable Java serialization filters
- **CM-E3**: Implement role-based access control (RBAC)
- **CM-E4**: Run services with least privilege
- **CM-E5**: Regular security patching and dependency updates
- **CM-E6**: Enable Java Security Manager with restrictive policies
- **CM-E7**: Input validation and sanitization for all entry points
- **CM-E8**: Implement authorization checks before privileged operations
- **CM-E9**: Secure configuration management with validation

### Additional Security Measures
- **CM-G1**: Implement fraud detection algorithms on voting patterns
- **CM-G2**: Anomaly detection for shopping cart behavior
- **CM-G3**: Regular security assessments and penetration testing
- **CM-G4**: Security monitoring with real-time alerting
- **CM-G5**: Incident response procedures
- **CM-G6**: Business continuity and disaster recovery planning
- **CM-G7**: Security awareness training
- **CM-G8**: Vulnerability management program
- **CM-G9**: Regular backup and recovery testing

## Security Controls Summary

| Control Type | Implementation Status | Priority |
|--------------|----------------------|----------|
| Authentication | Required | Critical |
| Authorization | Required | Critical |
| Encryption (Transit) | Required | Critical |
| Encryption (At Rest) | Required | High |
| Audit Logging | Required | High |
| Input Validation | Required | Critical |
| Rate Limiting | Required | High |
| Monitoring | Required | Critical |
| Data Masking | Required | High |
| Session Management | Required | High |
| CRDT Validation | Required | High |
| Split-Brain Resolution | Required | Critical |

## Compliance Considerations

- **PCI-DSS**: Shopping cart payment data handling
- **GDPR**: Customer PII protection and right to deletion
- **SOC 2**: Security and availability controls
- **Data Residency**: Geographic data storage requirements

## Review and Updates

- **Document Version**: 1.0
- **Last Updated**: 2024
- **Review Frequency**: Quarterly
- **Next Review Date**: Q1 2025
- **Document Owner**: Security Team / Fraud Prevention Team
- **Approved By**: Chief Security Officer, Chief Risk Officer
- **Classification**: Confidential
