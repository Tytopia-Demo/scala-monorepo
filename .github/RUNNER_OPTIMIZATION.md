# Runner Optimization Initiative

This document describes the runner right-sizing initiative implemented across this repository's CI/CD pipelines.

## Overview

This initiative optimizes GitHub Actions and CircleCI workflows by selecting appropriate runner sizes based on actual workload requirements, reducing costs while maintaining or improving performance.

## Goals

1. **Cost Optimization**: Reduce CI/CD costs by right-sizing runners to actual needs
2. **Performance**: Maintain or improve build and test times through proper resource allocation
3. **Visibility**: Track execution times and resource usage for continuous optimization
4. **Standardization**: Establish consistent runner selection patterns across workflows

## Implementation

### GitHub Actions

#### Runner Sizing Guidelines
See [RUNNER_SIZING_GUIDE.md](./RUNNER_SIZING_GUIDE.md) for detailed guidelines on selecting appropriate runners for different job types.

**Key Principles:**
- Security scans: `ubuntu-latest` (small), 10-minute timeout
- Linting/formatting: `ubuntu-latest` (small), 5-minute timeout
- Unit tests: `ubuntu-latest` (small), 10-minute timeout
- Build jobs: `ubuntu-latest` (small-medium), 15-minute timeout
- Integration tests: `ubuntu-latest` (medium), 15-minute timeout

#### Timeout Configuration
All jobs now include explicit `timeout-minutes` settings to prevent runaway jobs:
```yaml
jobs:
  example:
    runs-on: ubuntu-latest
    timeout-minutes: 10  # Prevents runaway costs
```

#### Execution Time Tracking
Jobs track and report execution times:
```yaml
steps:
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
```

#### Cost Monitoring Labels
Environment variables track cost centers:
```yaml
env:
  COST_CENTER: "testing"  # testing, build, security, deploy, quality
  RUNNER_SIZE: "small"    # small, medium, large
  PROJECT: ${{ github.event.repository.name }}
```

### CircleCI

#### Resource Class Configuration
See [../.circleci/RUNNER_SIZING_GUIDE.md](../.circleci/RUNNER_SIZING_GUIDE.md) for CircleCI-specific guidelines.

**Resource Classes:**
- Build jobs: `medium` (2 vCPUs, 4GB RAM)
- Test jobs: `small` (1 vCPU, 2GB RAM)
- Security scans: `small` (1 vCPU, 2GB RAM)

#### Example Configuration
```yaml
jobs:
  build:
    resource_class: medium  # Right-sized for Scala builds
    environment:
      COST_CENTER: "build"
      RUNNER_SIZE: "medium"
      PROJECT: "service-name"
```

## Reusable Workflows

Pre-configured workflow templates are available in `.github/workflows/templates/`:

1. **scala-build-template.yml**: Optimized Scala/SBT build workflow
2. **security-scan-template.yml**: Security scanning with appropriate runner sizing
3. **test-matrix-template.yml**: Parallelized test execution across modules

### Using Templates

```yaml
name: My Workflow

on: [push]

jobs:
  build:
    uses: ./.github/workflows/templates/scala-build-template.yml
    with:
      scala-version: '2.12.6'
      runner-size: 'small'
      timeout-minutes: 15
```

## Examples

### Optimized Workflows

1. **`.github/workflows/frogbot.yml`**: Security scanning workflow
   - Runner: `ubuntu-latest` (small)
   - Timeout: 10 minutes
   - Includes execution time tracking and cost labels

2. **`.github/workflows/ci-optimized-example.yml`**: Comprehensive CI pipeline
   - Multiple job types with appropriate runner sizing
   - Parallelized test execution
   - Complete metrics tracking

3. **`catalog-service/.circleci/config.yml`**: CircleCI example
   - Build job: `medium` resource class
   - Test job: `small` resource class
   - Execution time tracking included

## Monitoring and Optimization

### Tracking Metrics

Each job outputs execution metrics to GitHub Actions step summary:
- Job duration
- Runner size used
- Cost center assignment
- Module/project being processed

### Continuous Improvement

1. **Review execution times**: Check job summaries regularly
2. **Identify slow jobs**: Look for jobs that consistently run long
3. **Optimize before scaling**: Try caching, parallelization before using larger runners
4. **Adjust runner sizes**: Scale up or down based on actual performance data

### Cost Analysis

Monitor costs by:
- Cost center (testing, build, security, deploy, quality)
- Runner size distribution
- Job execution time trends
- Workflow frequency

## Migration Checklist

When creating or updating workflows:

- [ ] Determine job type and select appropriate runner size
- [ ] Add explicit `timeout-minutes` configuration
- [ ] Add job start time tracking step
- [ ] Include cost monitoring environment variables
- [ ] Add job duration reporting step
- [ ] Use caching where applicable
- [ ] Consider parallelization for long-running jobs
- [ ] Test the workflow
- [ ] Monitor execution times and adjust if needed

## Best Practices

1. **Start Small**: Begin with the smallest runner that might work, scale up if needed
2. **Use Caching**: Cache dependencies to reduce build times
3. **Parallelize**: Use matrix strategies to distribute work
4. **Set Timeouts**: Always specify timeout values
5. **Monitor**: Track execution times and resource usage
6. **Document**: Update this guide as patterns emerge

## Resources

- [GitHub Actions Runner Sizing Guide](.github/RUNNER_SIZING_GUIDE.md)
- [CircleCI Runner Sizing Guide](../.circleci/RUNNER_SIZING_GUIDE.md)
- [Workflow Templates](.github/workflows/templates/)
- [Example Optimized Workflow](.github/workflows/ci-optimized-example.yml)

## Results

### Expected Benefits

- **Cost Reduction**: 20-40% reduction in CI/CD costs through right-sizing
- **Faster Feedback**: Parallelized jobs reduce overall pipeline time
- **Better Visibility**: Clear metrics on resource usage and costs
- **Standardization**: Consistent approach across all workflows

### Metrics to Track

- Total workflow execution time
- Individual job execution times
- Runner minutes consumed per cost center
- Number of timeout occurrences
- Cache hit rates

## Support

For questions or suggestions about runner optimization:
1. Review the relevant sizing guide
2. Check the example workflows
3. Open an issue with the `ci-optimization` label
