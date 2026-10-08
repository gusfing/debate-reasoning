# Worked Example: Tricky Distributed Bug Triage

## Scenario: Intermittent 504 Gateway Timeouts During Traffic Bursts
**Problem Statement**: During promotional notification blasts, the `/api/v2/user/cart` endpoint intermittently spikes to 504 Gateway Timeouts (up to 3.8% error rate) for ~90 seconds, then recovers. CPU utilization remains under 35%, memory usage is stable at 48%, and database queries show normal latency.

---

### Phase 1: Problem Framing
- **Goal**: Identify the exact root cause of the 504 spikes and implement a permanent fix.
- **Hard Constraints**: Must not add memory pressure; fix must be deployable as an incremental patch.
- **Evaluation Criteria**: Root cause reproducibility > zero regression on steady-state latency.

---

### Phase 2: Blind Generation (Hypotheses)

#### Builder (Hypothesis A: Downstream Network Latency)
- **Thesis**: An external third-party inventory pricing API called inside the cart route is suffering transient timeouts under burst load, causing the ingress gateway to time out at 15s.
- **Load-Bearing Assumption**: Outgoing HTTP calls to inventory API lack tight per-request timeouts.
- **Confidence**: Medium.

#### Skeptic (Hypothesis B: Connection Pool Starvation)
- **Thesis**: Internal HTTP connection pool or database connection pool is exhausting max idle connections. Incoming requests block on acquiring a connection from the pool until the gateway 15s timeout fires.
- **Vulnerable Assumption Targeted**: Low CPU does not mean low thread contention; connection queue waits consume zero CPU.
- **Confidence**: High.

#### Contrarian (Hypothesis C: Event Loop / Garbage Collection Pause)
- **Thesis**: Stop-the-world GC pauses or synchronous file I/O inside logging middleware is freezing worker event loops during burst payload serializations.
- **Confidence**: Low.

#### Pragmatist (Triage & Mitigations)
- **Thesis**: Regardless of internal cause, the gateway timeout is 15s, but downstream client timeout is 3s. Set explicit aggressive timeouts (1.5s) with fast-fail circuit breakers on all external calls.
- **Confidence**: High.

#### Verifier (Empirical Testing Sprint)
- **Test 1**: Inspect ingress Nginx/ALB access logs around 504 occurrences.
  - `[VERIFIED]`: Upstream response time for 504s is exactly 15.001s.
- **Test 2**: Inspect internal connection pool metrics in Prometheus dashboard.
  - `[VERIFIED]`: `http_client_pool_pending_requests` spikes from 0 to 850 concurrently during the 90-second burst window. Max connections was hardcoded to `50`.
- **Test 3**: Check third-party inventory API latency during incident window.
  - `[REFUTED]`: Third-party API p99 latency was 42ms during the entire event. The requests were never reaching the wire because they were waiting for an internal pool socket!

---

### Phase 3: Cross-Examination Matrix
- **Builder Concession**: Concedes Hypothesis A was wrong—the third-party API was fast, but requests queued inside the local application's undersized connection pool.
- **Contrarian Concession**: Concedes GC pause metrics show max pause of 8ms (far below 15s).
- **All Roles Agree**: The Skeptic's connection pool starvation hypothesis was proven by the Verifier's Prometheus metrics audit.

---

### Phase 4: Final Adjudication (Judge)

```markdown
## Final Verdict: HTTP Client Connection Pool Saturation

### The Root Cause
The internal HTTP client connection pool was hardcoded to `MaxIdleConns: 50` and `MaxConnsPerHost: 50`.
Under traffic bursts of 1,200 req/sec, connection acquisition queued in memory, causing workers
to hang until the 15-second gateway timeout elapsed.

### Action Plan
1. Increase `MaxConnsPerHost` to 500.
2. Configure `AcquireTimeout: 500ms` with fast fallback.
3. Add Prometheus alerting when connection pool saturation exceeds 70%.

### Verification
Simulated burst test of 2,000 req/sec showed 0 timeouts, p99 latency remained at 28ms.
```
