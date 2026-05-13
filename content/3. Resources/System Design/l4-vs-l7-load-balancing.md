# L4 vs L7 Load Balancing

**Date:** 2026-02-01
**Category:** Load Balancing
**Related Systems:** N/A

## Key Concepts

- **L4 (Transport Layer)** operates at TCP level - sees only IP addresses and ports
- **L7 (Application Layer)** operates at HTTP level - can inspect URLs, headers, cookies, request body
- L7 must **terminate the TCP connection** from the client, creating two separate connections (client↔LB and LB↔backend)
- L4 can forward packets without termination - faster but no content inspection
- **TLS termination**: L7 must decrypt HTTPS traffic to inspect HTTP content; L4 can pass encrypted traffic through untouched

## Design Decisions & Trade-offs

### TLS Termination Trade-offs
- **Benefit**: Centralized certificate management, CPU offload from backends, enables WAF/traffic inspection
- **Risk**: Traffic between LB and backend may be unencrypted, exposing data on internal network
- **Mitigation**: Re-encrypt traffic to backend (more CPU overhead) for sensitive data (banking, healthcare, PII)

### When to Choose L7
- Need content-based routing (`/api/*` → API servers, `/static/*` → CDN)
- Need to inspect headers, cookies, or user-agent for routing decisions
- Running a WAF for security inspection
- Need sticky sessions based on cookies

### When to Choose L4
- Don't need smart routing - just distribute connections
- Need maximum performance with minimal latency
- Passing through encrypted traffic without inspection
- WebSocket or other persistent TCP connections (though L7 can handle these too)

## Gaps Identified

- When exactly would L4 be the right choice? (didn't fully explore scenarios)
- How do L4 load balancers handle health checks without seeing application-level responses?
- Connection pooling between LB and backends
- Specific algorithms (round-robin, least connections) at each layer

## Q&A Highlights

**Q:** Why is L4 faster than L7?
**A:** L4 only looks at IP/port headers and can forward packets. L7 must fully terminate the TCP connection, buffer and parse the HTTP request, then open a new connection to the backend - more work.

**Q:** What happens to TLS with L7 load balancing?
**A:** L7 must decrypt TLS to see HTTP content. This is "TLS termination" - the LB holds certs and does crypto work.

**Q:** Is TLS termination "extra security"?
**A:** No - it's a trade-off. Traffic between LB and backend is often unencrypted, creating risk if internal network is compromised. For sensitive data, re-encrypt to backends.

**Q:** URL path routing - L4 or L7?
**A:** L7. URL path is HTTP-level information inside the TCP payload. L4 can't see it.

## Resources to Review

None mentioned

## Next Steps

- Explore specific scenarios where L4 is preferred
- Study health check mechanisms at both layers
- Learn about connection pooling and keep-alive between LB and backends
- Understand sticky sessions implementation at L7
