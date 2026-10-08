# Worked Example: Architecture Decision

## Scenario: Payment Order Ingestion Architecture
**Problem Statement**: The checkout service handles 4,000 payment requests/sec during peak flash sales. Synchronous dual-writes to the relational database and the message broker occasionally cause split-brain states when the broker drops connections. The team is debating whether to adopt full Event Sourcing with Apache Kafka, maintain the status quo with two-phase commit, or introduce a Transactional Outbox Pattern.

---

### Phase 1: Problem Framing
- **Goal**: Ensure 100% data consistency between payment order records and downstream notification/ledger events without distributed transactions.
- **Hard Constraints**: p99 checkout latency must remain under 120ms; rollback must be possible within one release cycle; team is 6 backend engineers.
- **Evaluation Criteria**: Correctness (zero lost events) > Operational simplicity > Latency.

---

### Phase 2: Blind Generation (Selected Extracts)

#### Builder (Pro-Event Sourcing)
- **Thesis**: Migrate entirely to Event Sourcing on Kafka. The event log becomes the single source of truth; state is derived via CQRS read projections.
- **Load-Bearing Assumption**: The team can safely manage event schema evolution, replay offset management, and eventual consistency in the payment ledger.
- **Confidence**: Medium.
- **Falsification Trigger**: "If CQRS read projection lag exceeds 200ms or event schema migration tooling is absent."

#### Skeptic (Adversarial Falsifier)
- **Thesis**: Full Event Sourcing introduces massive operational complexity (CQRS divergence, replay bugs, dual-model maintenance) for a team of 6 engineers.
- **Vulnerable Assumptions Targeted**: Event sourcing does not solve write-side concurrency collisions; optimistic concurrency version conflicts will cause high checkout abort rates during hot-item flash sales.
- **Confidence**: High.

#### Contrarian (Paradigm Inversion)
- **Thesis**: Neither Event Sourcing nor distributed locks are needed. Use PostgreSQL's native logical replication stream (Change Data Capture via Debezium) directly from WAL to downstream queues.
- **Load-Bearing Assumption**: DB write throughput on WAL can scale to 10k ops/sec without replication lag spikes.
- **Confidence**: Medium.

#### Pragmatist (Operational Realism)
- **Thesis**: Use the **Transactional Outbox Pattern** with a dedicated outbox table written in the same local ACID transaction as the order record, polled by an asynchronous CDC worker.
- **Two-Way Door Score**: High reversibility (plain tables, standard SQL).
- **Complexity Assessment**: Low. Zero change to read-path query models.
- **Confidence**: High.

#### Verifier (Empirical Fact-Checker)
- **Claim 1**: *Can PostgreSQL handle 4,000 inserts/sec into orders + outbox tables simultaneously?*
  - `[VERIFIED]`: pgbench test with connection pooling easily sustains 6,800 dual-row ACID writes/sec at <15ms p99 on current db instance.
- **Claim 2**: *Does Debezium WAL decoding introduce significant CPU load on primary database?*
  - `[VERIFIED: doc/perf-report]`: Logical replication adds ~4-7% CPU overhead on current instance size.
- **Claim 3**: *Is Kafka cluster already managed with 99.99% SLA?*
  - `[UNVERIFIED / REFUTED]`: Infrastructure team confirms Kafka is self-managed without 24/7 dedicated SRE coverage.

---

### Phase 3: Cross-Examination Matrix
- **Builder Concession**: Concedes that full CQRS projection management and event versioning migrations require dedicated platform tooling that the team does not currently have.
- **Skeptic Concession**: Concedes that Transactional Outbox with Debezium eliminates dual-write split-brain without modifying write schemas.
- **Pragmatist & Contrarian Convergence**: The Pragmatist agrees that CDC-based Outbox (Debezium reading the outbox table) avoids polling overhead.

---

### Phase 4: Final Adjudication (Judge)

```markdown
## Final Verdict: Transactional Outbox Pattern via PostgreSQL CDC

### The Decision
Implement the Transactional Outbox Pattern with Debezium CDC reading the outbox table.
Reject full Event Sourcing migration.

### Why This Won
1. Guarantees exactly-once message dispatch from local ACID transactions with zero dual-write race conditions.
2. Confirmed by Verifier that db handles 6,800 writes/sec (well above 4,000 peak requirement).
3. Two-way door: Schema changes are standard SQL; team requires zero new CQRS query infrastructure.

### Reversal Metric
If outbox table write contention causes p99 latency to exceed 90ms under 5k req/sec load test,
evaluate partitioning the outbox table by hash(order_id).
```
