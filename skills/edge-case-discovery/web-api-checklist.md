# Web API Edge Case Checklist

Additional edge cases specific to HTTP APIs and web services.

## Authentication and Authorization

- Expired tokens (JWT past expiry, session timed out)
- Revoked tokens (user logged out elsewhere, token blocklisted)
- Missing auth header entirely
- Malformed auth header (wrong scheme, truncated token)
- Valid authentication but insufficient authorization (wrong role/scope)
- Token refresh race condition (two requests refresh simultaneously)

## Request Handling

- Content-Type mismatch (sending form data with JSON content-type header)
- Missing Content-Type header
- Request body exceeds size limit
- Chunked transfer encoding with incomplete chunks
- Duplicate request headers
- HTTP method not supported for endpoint
- Query parameter arrays with zero, one, and many elements
- URL-encoded vs. non-encoded special characters in path segments

## Concurrency and Idempotency

- Same resource modified by two requests simultaneously (last-write-wins vs. conflict detection)
- Retry of a non-idempotent operation (double charge, double create)
- Request arrives after resource was deleted
- Long-running request: client disconnects mid-processing
- Thundering herd after cache expiry or service restart

## Response Handling

- Extremely large response bodies (pagination needed but not implemented)
- Empty result sets (return empty array, not 404)
- Partial failure in batch operations (some items succeed, some fail)
- Response format negotiation (Accept header mismatch)

## Rate Limiting and Quotas

- Rate limit hit — response includes retry-after?
- Per-user vs. per-IP vs. global rate limits
- Rate limit state after service restart (in-memory counters lost)
- Burst traffic patterns vs. sustained traffic

## Versioning and Compatibility

- Client sends request with unknown fields (forward compatibility)
- Client expects fields that were removed (backward compatibility)
- API version mismatch between client and server
- Webhook receiver expects old payload format after API update
