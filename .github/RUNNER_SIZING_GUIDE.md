# GitHub Actions Runner Sizing Guide

This document provides guidelines for selecting appropriate runner sizes for different workflow jobs to optimize cost and performance.

## Runner Types

### GitHub-Hosted Runners

#### Small (ubuntu-latest)
- **Specs**: 2-core CPU, 7 GB RAM, 14 GB SSD
- **Use Cases**:
  - Linting and code formatting checks
  - Documentation generation
  - Small unit tests
  - Security scans (dependency scanning, SAST)
- **Typical Duration**: < 5 minutes
- **Cost**: Free (included in GitHub plan limits)

#### Medium (ubuntu-latest-4-cores or custom)
- **Specs**: 4-core CPU, 16 GB RAM, 14 GB SSD
- **Use Cases**:
  - Medium-sized test suites
  - Building single services
  - Integration tests
  - Docker image builds (single service)
- **Typical Duration**: 5-15 minutes
- **Cost**: Paid (if using larger runners)

#### Large (ubuntu-latest-8-cores or custom)
- **Specs**: 8-core CPU, 32 GB RAM, 14 GB SSD
- **Use Cases**:
  - Full test suites with multi-jvm tests
  - Building entire monorepos
  - Performance tests
  - Large Docker image builds
- **Typical Duration**: 15-30 minutes
- **Cost**: Paid (higher tier)

## Job Type to Runner Mapping

| Job Type | Recommended Runner | Timeout | Tags |
|----------|-------------------|---------|------|
| Security Scans (SAST, SCA) | ubuntu-latest | 10 min | cost-center:security, size:small |
| Linting/Formatting | ubuntu-latest | 5 min | cost-center:quality, size:small |
| Unit Tests (Java/Scala) | ubuntu-latest | 10 min | cost-center:testing, size:small |
| Integration Tests | ubuntu-latest | 15 min | cost-center:testing, size:medium |
| Multi-JVM Tests | ubuntu-latest | 20 min | cost-center:testing, size:medium |
| Build (SBT/Maven) | ubuntu-latest | 15 min | cost-center:build, size:medium |
| Docker Build | ubuntu-latest | 15 min | cost-center:build, size:medium |
| Deploy | ubuntu-latest | 10 min | cost-center:deploy, size:small |

## Timeout Guidelines

Always specify timeouts to prevent runaway jobs:

- **Security scans**: 10 minutes
- **Linting**: 5 minutes
- **Unit tests**: 10 minutes
- **Integration tests**: 15 minutes
- **Builds**: 15-20 minutes
- **Deployments**: 10 minutes

## Execution Time Tracking

All jobs should track execution time using the following pattern:

```yaml
- name: Job Start Time
  id: start
  run: echo "start_time=$(date +%s)" >> $GITHUB_OUTPUT

# ... job steps ...

- name: Job Duration
  if: always()
  run: |
    end_time=$(date +%s)
    duration=$((end_time - ${{ steps.start.outputs.start_time }}))
    echo "Job duration: ${duration}s"
    echo "job_duration=${duration}" >> $GITHUB_OUTPUT
```

## Cost Monitoring Labels

Add these labels to all workflow jobs for cost tracking:

```yaml
jobs:
  example-job:
    runs-on: ubuntu-latest
    env:
      COST_CENTER: "testing"  # testing, build, security, deploy, quality
      RUNNER_SIZE: "small"     # small, medium, large
      PROJECT: "project-name"
```

## Best Practices

1. **Start small**: Begin with the smallest runner that works, then scale up if needed
2. **Use caching**: Leverage GitHub Actions cache to reduce build times
3. **Parallelize**: Use matrix strategies to distribute work across multiple smaller runners
4. **Monitor**: Track job execution times and adjust runner sizes accordingly
5. **Set timeouts**: Always specify timeout-minutes to prevent cost overruns
6. **Tag everything**: Use cost center and size tags for visibility

## Migration Checklist

When updating an existing workflow:

- [ ] Identify the job type and workload characteristics
- [ ] Select appropriate runner size from the table above
- [ ] Add timeout-minutes configuration
- [ ] Add execution time tracking steps
- [ ] Add cost monitoring environment variables
- [ ] Test the workflow with the new configuration
- [ ] Monitor execution times and adjust if needed

## Examples

See `.github/workflows/templates/` for reusable workflow examples with proper runner sizing.
