# Threat Model - Catalog Service

## Overview

### Service Purpose
The Catalog Service is a distributed multi-node Akka application designed to demonstrate and test actor-based communication across multiple JVM instances. It provides a foundation for building scalable, resilient catalog management systems using Akka's remote actor capabilities.

### Service Scope
- Multi-node actor communication
- Remote actor selection and messaging
- Distributed system testing capabilities
- Actor-based catalog data management

## Data Flow Diagram

```
┌──────────────┐                    ┌──────────────┐
│              │   Actor Messages   │              │
│   Node 1     │◄──────────────────►│   Node 2     │
│  (Client)    │                    │  (Service)   │
│              │                    │              │
└──────────────┘                    └──────────────┘
       │                                   │
       │                                   │
       ▼                                   ▼
┌──────────────┐                    ┌──────────────┐
│  Akka Remote │                    │ Actor System │
│  Transport   │                    │   (Ponger)   │
└──────────────┘                    └──────────────┘
```

## Dependencies

### External Libraries
- **Akka Actor** (2.5.18): Core actor system implementation
- **Akka Remote** (2.5.18): Remote actor communication
- **Akka Multi-Node TestKit** (2.5.18): Multi-node testing framework
- **ScalaTest** (3.0.5): Testing framework
- **Scala** (2.12.6): Programming language runtime

### Infrastructure Dependencies
- JVM Runtime Environment
- Network connectivity between nodes
- TCP/IP protocol stack

### Internal Dependencies
- Multi-JVM SBT plugin
- Actor system configuration

## Entry Points

1. **Remote Actor Messages**
   - Protocol: Akka Remote Protocol
   - Port: Configurable (default Akka ports)
   - Authentication: Akka security configuration
   - Data Format: Serialized actor messages

2. **Actor System Initialization**
   - Method: ActorSystem creation
   - Configuration: application.conf
   - Trust Level: Internal system

3. **Multi-Node Test Harness**
   - Interface: SBT test commands
   - Access: Development/CI environment
   - Trust Level: Trusted internal

## Exit Points

1. **Remote Actor Responses**
   - Destination: Remote actor systems
   - Protocol: Akka Remote Protocol
   - Data: Serialized response messages

2. **Logging Output**
   - Destination: Log files/console
   - Data: System events, actor messages, errors
   - Sensitivity: May contain internal state information

3. **Test Results**
   - Destination: Test reports/CI system
   - Data: Test outcomes and metrics

## Assets

### Data Assets
- **Actor State**: Internal state of catalog actors
- **Message Content**: Business data in actor messages
- **Configuration Data**: Actor system and cluster configuration
- **Session Data**: Actor communication sessions

### System Assets
- **Actor System**: Core runtime environment
- **Remote Communication Channel**: Network connections between nodes
- **Catalog Data Store**: In-memory catalog information

### Security Assets
- **Authentication Credentials**: If configured for Akka security
- **TLS Certificates**: If using secure communication
- **Configuration Secrets**: Database passwords, API keys

## Trust Levels

### Level 1: Untrusted External
- External networks
- Unvalidated input sources
- Public internet

### Level 2: Authenticated External
- Verified client applications
- Authenticated API consumers
- Partner systems

### Level 3: Internal Services
- Internal microservices
- Other nodes in the cluster
- Internal actor systems

### Level 4: Administrative
- System administrators
- Configuration management systems
- CI/CD pipelines

### Level 5: System Core
- Core actor system
- Internal JVM processes
- Local actor references

## STRIDE Threat List

### Spoofing Threats
| ID | Threat | Affected Component | Severity |
|----|--------|-------------------|----------|
| S1 | Malicious node impersonating legitimate cluster member | Actor Remote System | High |
| S2 | Unauthorized actor path access | Actor Selection | Medium |
| S3 | Message sender spoofing | Actor Messages | High |
| S4 | Configuration file tampering with false cluster seeds | Configuration | Medium |

### Tampering Threats
| ID | Threat | Affected Component | Severity |
|----|--------|-------------------|----------|
| T1 | Man-in-the-middle attack on actor messages | Network Communication | High |
| T2 | Malicious modification of serialized messages | Message Serialization | High |
| T3 | Configuration file manipulation | Configuration Files | Medium |
| T4 | Memory corruption of actor state | Actor State | Medium |

### Repudiation Threats
| ID | Threat | Affected Component | Severity |
|----|--------|-------------------|----------|
| R1 | Lack of message audit trail | Message Processing | Low |
| R2 | No actor action logging | Actor System | Medium |
| R3 | Insufficient transaction logging | Business Logic | Medium |

### Information Disclosure Threats
| ID | Threat | Affected Component | Severity |
|----|--------|-------------------|----------|
| I1 | Unencrypted network communication exposing message content | Network Transport | High |
| I2 | Verbose error messages revealing system internals | Error Handling | Medium |
| I3 | Log files containing sensitive catalog data | Logging System | Medium |
| I4 | Memory dumps exposing actor state | Runtime Environment | Low |

### Denial of Service Threats
| ID | Threat | Affected Component | Severity |
|----|--------|-------------------|----------|
| D1 | Message flooding overwhelming actor mailboxes | Actor Mailbox | High |
| D2 | Resource exhaustion through actor creation | Actor System | High |
| D3 | Network bandwidth saturation | Network Layer | Medium |
| D4 | Deadletter overflow | Dead Letter Queue | Low |

### Elevation of Privilege Threats
| ID | Threat | Affected Component | Severity |
|----|--------|-------------------|----------|
| E1 | Exploiting actor system to gain JVM access | Actor System | High |
| E2 | Configuration injection to modify security settings | Configuration | High |
| E3 | Serialization vulnerabilities enabling code execution | Deserialization | Critical |
| E4 | Unauthorized access to privileged actors | Actor Hierarchy | Medium |

## Countermeasures

### Spoofing Countermeasures
- **CM-S1**: Enable Akka SSL/TLS for remote actor communication with mutual authentication
- **CM-S2**: Implement actor path validation and whitelisting
- **CM-S3**: Use secure serializers with message signing capabilities
- **CM-S4**: Protect configuration files with file system permissions (chmod 600)
- **CM-S5**: Use Akka Security features for node authentication

### Tampering Countermeasures
- **CM-T1**: Enable TLS encryption for all remote actor communication
- **CM-T2**: Use cryptographic signing for critical messages
- **CM-T3**: Implement configuration file integrity checks (checksums)
- **CM-T4**: Use immutable data structures for actor state
- **CM-T5**: Enable Java serialization filters to prevent malicious payloads

### Repudiation Countermeasures
- **CM-R1**: Implement comprehensive audit logging for all actor messages
- **CM-R2**: Enable Akka event stream logging with persistent storage
- **CM-R3**: Add business transaction logging with timestamps and actor IDs
- **CM-R4**: Implement log aggregation with tamper-evident storage

### Information Disclosure Countermeasures
- **CM-I1**: Mandatory TLS/SSL for all network communication
- **CM-I2**: Implement custom error handling to sanitize error messages
- **CM-I3**: Encrypt sensitive data in logs or use log masking
- **CM-I4**: Disable core dumps in production or encrypt dump files
- **CM-I5**: Implement data classification and handling policies

### Denial of Service Countermeasures
- **CM-D1**: Configure mailbox size limits and backpressure mechanisms
- **CM-D2**: Implement actor creation throttling and supervision strategies
- **CM-D3**: Use rate limiting and circuit breakers at network boundaries
- **CM-D4**: Monitor system resources and implement auto-scaling
- **CM-D5**: Configure appropriate timeouts and deadletter handling

### Elevation of Privilege Countermeasures
- **CM-E1**: Run actor systems with least privilege principles
- **CM-E2**: Implement strict configuration validation and sanitization
- **CM-E3**: Use secure deserialization with class whitelisting
- **CM-E4**: Implement role-based access control for actor operations
- **CM-E5**: Regular security updates for Akka and all dependencies
- **CM-E6**: Enable Java Security Manager with restrictive policies

### Additional Security Measures
- **CM-G1**: Regular security assessments and penetration testing
- **CM-G2**: Implement monitoring and alerting for suspicious activities
- **CM-G3**: Maintain security incident response procedures
- **CM-G4**: Regular backup and disaster recovery testing
- **CM-G5**: Security training for development team
- **CM-G6**: Dependency vulnerability scanning in CI/CD pipeline

## Security Controls Summary

| Control Type | Implementation Status | Priority |
|--------------|----------------------|----------|
| Authentication | Recommended | High |
| Authorization | Recommended | High |
| Encryption (Transit) | Recommended | Critical |
| Encryption (At Rest) | Not Applicable | N/A |
| Audit Logging | Partial | High |
| Input Validation | Required | High |
| Rate Limiting | Recommended | Medium |
| Monitoring | Required | High |

## Review and Updates

- **Document Version**: 1.0
- **Last Updated**: 2024
- **Review Frequency**: Quarterly
- **Next Review Date**: Q1 2025
- **Document Owner**: Security Team
- **Approved By**: Chief Security Officer
