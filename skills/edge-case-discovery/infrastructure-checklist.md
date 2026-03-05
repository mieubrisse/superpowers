# Infrastructure Edge Case Checklist

Additional edge cases specific to infrastructure, deployment, and platform tooling.

## Deployment

- Deploy while previous deploy is still in progress
- Deploy fails midway (some instances on new version, some on old)
- Rollback to previous version (database migrations, config changes — are they backward compatible?)
- First deploy to a fresh environment (no existing state, no previous config)
- Deploy with zero instances running (cold start)

## Networking

- DNS resolution failure or stale cache
- TLS certificate expiry or mismatch
- Firewall rules blocking expected traffic (especially after infrastructure change)
- IPv4 vs. IPv6 differences
- NAT traversal issues
- Connection draining during graceful shutdown (in-flight requests)

## Configuration

- Missing configuration value (no default, not set in environment)
- Configuration type mismatch (string where integer expected)
- Configuration hot-reload vs. restart-required (which values can change at runtime?)
- Secret rotation (new credentials deployed before old ones expire?)
- Configuration drift between environments (dev works, staging breaks)

## Persistence and Storage

- Disk full on persistent volume
- Database connection pool exhaustion
- Database failover (primary → replica promotion)
- Backup restoration to a different point in time
- Storage encryption key rotation

## Health and Monitoring

- Health check passes but service is functionally broken (can connect but returns errors)
- Health check fails transiently during startup (readiness vs. liveness)
- Cascading failure (one unhealthy service causes dependent services to fail)
- Alert fatigue (noisy alerts mask real problems)
- Log volume spike fills disk or overwhelms log aggregator

## Resource Management

- Container OOM-killed (memory limit exceeded)
- CPU throttling under load
- Autoscaler lag (traffic spike arrives before new instances are ready)
- Resource quota exhaustion (IP addresses, load balancer rules, DNS records)
- Orphaned resources after failed teardown (leaked cloud resources costing money)

## Security

- Expired or rotated credentials in environment (API keys, database passwords)
- Overly permissive IAM/RBAC roles (principle of least privilege)
- Network policies not applied after infrastructure change
- Secrets exposed in logs, error messages, or debug output
- Supply chain: compromised base image or dependency
