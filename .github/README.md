# CI/CD Workflow Documentation

This directory contains GitHub Actions workflows and related documentation for the Scala monorepo.

## Quick Links

- **[Runner Optimization Overview](./RUNNER_OPTIMIZATION.md)** - Comprehensive guide to the runner right-sizing initiative
- **[GitHub Actions Runner Sizing Guide](./RUNNER_SIZING_GUIDE.md)** - Detailed guidelines for selecting appropriate runners
- **[CircleCI Runner Sizing Guide](../.circleci/RUNNER_SIZING_GUIDE.md)** - CircleCI-specific resource class guidelines
- **[Runner Configuration Reference](./runner-config.yml)** - Standardized configuration settings

## Workflows

### Active Workflows

- **[frogbot.yml](./workflows/frogbot.yml)** - Security scanning with Frogbot
  - Optimized with: 10-minute timeout, small runner, cost tracking

- **[ci-optimized-example.yml](./workflows/ci-optimized-example.yml)** - Example CI pipeline
  - Demonstrates: Parallelized testing, proper runner sizing, execution metrics

### Reusable Templates

Located in `workflows/templates/`:

- **[scala-build-template.yml](./workflows/templates/scala-build-template.yml)** - Template for Scala/SBT builds
- **[security-scan-template.yml](./workflows/templates/security-scan-template.yml)** - Template for security scans
- **[test-matrix-template.yml](./workflows/templates/test-matrix-template.yml)** - Template for matrix testing

## Runner Sizing Quick Reference

### GitHub Actions

| Job Type | Runner | Timeout | Cost Center |
|----------|--------|---------|-------------|
| Security Scans | ubuntu-latest | 10 min | security |
| Linting | ubuntu-latest | 5 min | quality |
| Unit Tests | ubuntu-latest | 10 min | testing |
| Builds | ubuntu-latest | 15 min | build |
| Integration Tests | ubuntu-latest | 15 min | testing |

### CircleCI

| Job Type | Resource Class | Cost Center |
|----------|----------------|-------------|
| Builds | medium | build |
| Tests | small | testing |
| Security | small | security |

## Key Concepts

### Cost Centers

All jobs are tagged with cost centers for tracking:
- `security` - Security scanning and vulnerability detection
- `quality` - Code quality checks (linting, formatting)
- `testing` - Test execution (unit, integration, multi-jvm)
- `build` - Compilation and packaging
- `deploy` - Deployment operations

### Runner Sizes

Jobs are labeled with runner sizes:
- `small` - Lightweight operations (< 5 minutes)
- `medium` - Standard operations (5-15 minutes)
- `large` - Heavy operations (15-30 minutes)

### Execution Time Tracking

All optimized jobs track and report execution time:
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
```

## Creating a New Workflow

1. Review the [Runner Sizing Guide](./RUNNER_SIZING_GUIDE.md)
2. Choose appropriate runner size and timeout from [runner-config.yml](./runner-config.yml)
3. Use a template from `workflows/templates/` or reference `ci-optimized-example.yml`
4. Include:
   - Explicit `timeout-minutes`
   - Cost monitoring env vars (`COST_CENTER`, `RUNNER_SIZE`)
   - Execution time tracking steps
5. Test and monitor execution times
6. Adjust runner size if needed

## Best Practices

1. **Start small** - Use the smallest runner that works, scale up if needed
2. **Set timeouts** - Always specify `timeout-minutes` to prevent runaway costs
3. **Use caching** - Cache dependencies to reduce build times
4. **Parallelize** - Use matrix strategies for independent jobs
5. **Monitor** - Check execution metrics and optimize based on data
6. **Document** - Add comments explaining runner size choices

## Monitoring

Check workflow run summaries for execution metrics:
- Job duration
- Runner size used
- Cost center assignment
- Cache performance

Use this data to continuously optimize runner selections.

## Support

For questions about workflow optimization:
1. Check the relevant documentation linked above
2. Review example workflows
3. Consult the team's CI/CD channel
4. Open an issue with the `ci-optimization` label
