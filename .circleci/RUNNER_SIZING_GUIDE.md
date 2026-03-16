# CircleCI Runner Sizing Guide

This document provides guidelines for selecting appropriate runner resource classes for different job types in CircleCI to optimize cost and performance.

## Resource Classes

CircleCI offers several resource class options for Docker executors:

### Small
- **Specs**: 1 vCPU, 2GB RAM
- **Resource Class**: `small`
- **Use Cases**:
  - Linting and code formatting
  - Simple unit tests
  - Security scans (dependency checks)
  - Documentation generation
- **Typical Duration**: < 5 minutes
- **Cost**: Low

### Medium
- **Specs**: 2 vCPUs, 4GB RAM
- **Resource Class**: `medium`
- **Use Cases**:
  - Scala/Java builds (SBT, Maven)
  - Integration tests
  - Docker image builds
  - Test suites with moderate complexity
- **Typical Duration**: 5-15 minutes
- **Cost**: Medium (default)

### Medium+
- **Specs**: 3 vCPUs, 6GB RAM
- **Resource Class**: `medium+`
- **Use Cases**:
  - Large Scala builds
  - Multi-JVM tests
  - Complex integration test suites
- **Typical Duration**: 10-20 minutes
- **Cost**: Higher

### Large
- **Specs**: 4 vCPUs, 8GB RAM
- **Resource Class**: `large`
- **Use Cases**:
  - Full monorepo builds
  - Performance tests
  - Heavy parallel testing
- **Typical Duration**: 15-30 minutes
- **Cost**: High

## Job Type to Resource Class Mapping

| Job Type | Recommended Resource Class | Notes |
|----------|----------------------------|-------|
| Build (SBT/Maven) | medium | Scala compilation is CPU-intensive |
| Unit Tests | small | Most unit tests are lightweight |
| Integration Tests | medium | May need more resources for containers |
| Multi-JVM Tests | medium+ or large | Requires more resources for parallel JVMs |
| Security Scans | small | Dependency scanning is lightweight |
| Docker Builds | medium | Container builds need moderate resources |
| Deploy | small | Deployment scripts are usually lightweight |

## Configuration Example

```yaml
jobs:
  build:
    resource_class: medium  # Right-sized for Scala builds
    docker:
      - image: circleci/openjdk:8-jdk
    environment:
      COST_CENTER: "build"
      RUNNER_SIZE: "medium"
    steps:
      - checkout
      - run: sbt compile
```

## Cost Monitoring

All jobs should include cost monitoring labels in the `environment` section:

```yaml
environment:
  COST_CENTER: "testing"  # testing, build, security, deploy
  RUNNER_SIZE: "medium"   # small, medium, medium+, large
  PROJECT: "service-name"
```

## Execution Time Tracking

Track job execution times using these steps:

```yaml
steps:
  - run:
      name: Job Start Time
      command: echo "export JOB_START_TIME=$(date +%s)" >> $BASH_ENV

  # ... other steps ...

  - run:
      name: Job Duration
      command: |
        END_TIME=$(date +%s)
        DURATION=$((END_TIME - JOB_START_TIME))
        echo "Job duration: ${DURATION}s"
      when: always
```

## Best Practices

1. **Start with medium**: The default resource class is usually a good starting point
2. **Scale down if possible**: Monitor execution times and scale down to `small` if jobs complete quickly
3. **Scale up cautiously**: Only increase to `medium+` or `large` if builds are timing out or very slow
4. **Use caching**: Leverage CircleCI's caching to reduce build times instead of using larger runners
5. **Monitor costs**: Track which jobs consume the most credits and optimize those first
6. **Parallelize when beneficial**: Sometimes running multiple smaller jobs in parallel is more efficient than one large job

## Migration Checklist

When updating an existing job:

- [ ] Determine current execution time and resource usage
- [ ] Select appropriate resource class based on job type
- [ ] Add `resource_class` configuration
- [ ] Add cost monitoring environment variables
- [ ] Add execution time tracking
- [ ] Test the configuration
- [ ] Monitor and adjust if needed

## Resource Class Pricing

Resource classes are billed in CircleCI credits:

- **small**: 5 credits per minute
- **medium**: 10 credits per minute (default)
- **medium+**: 15 credits per minute
- **large**: 20 credits per minute

Optimizing resource classes can significantly reduce credit consumption and costs.
