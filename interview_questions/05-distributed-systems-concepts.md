# Distributed Systems Concepts

## Overview

A distributed system is **multiple independent computers working together** to solve a problem or provide a service, communicating over a network.

## What is a Distributed System?

### Examples
- Microservices architecture
- Database replication across data centers
- Load-balanced web servers
- Cloud services (AWS, GCP, Azure)

### Why they're needed
- **Scalability:** Handle more users by adding more machines.
- **Reliability:** If one machine fails, others continue working.
- **Performance:** Distribute load geographically; serve users from closest location.

### Challenges
- **Complexity:** Hard to reason about multiple machines failing independently.
- **Consistency:** Multiple copies of data may be out of sync.
- **Latency:** Network communication is slow compared to local memory.
- **Partial failure:** One machine may fail while others work, leading to unclear state.

## CAP Theorem

The **CAP Theorem** states: In a distributed system, you can achieve **at most 2 of 3 guarantees**:

### The Three Properties

**Consistency (C):**
- Every read returns the **most recent write**.
- All nodes see the same data at the same time.
- Example: You transfer $100 from account A to B. After transfer, all nodes show the new balance.

**Availability (A):**
- Every request gets a response, even if nodes fail.
- System remains operational 24/7.
- Example: A social media API responds to every request, even if some servers are down.

**Partition Tolerance (P):**
- System continues working even if network partition occurs (nodes can't talk to each other).
- Must tolerate it because network failures happen in real world.

### The Trade-off

When a network partition happens, you must choose:

**Partition Tolerance is always needed** (can't eliminate network failures), so really it's C vs A:

#### CP (Consistency + Partition Tolerance)
- Choose consistency: block requests if you can't guarantee latest data.
- During partition: return error to user instead of stale data.
- **Use case:** Financial systems (banking, stock trading). Users prefer "service unavailable" over inconsistent balances.
- **Example:** If DB cluster partitions, some nodes refuse writes to avoid conflicting updates.

#### AP (Availability + Partition Tolerance)
- Choose availability: always respond, even if data might be stale.
- During partition: nodes work independently, sync up when partition heals.
- **Use case:** Social media, e-commerce. Users prefer slightly stale data over no service.
- **Example:** If cache cluster partitions, each partition serves reads with its own copy. When healed, data "eventually" converges.

### Simple example

Imagine 2 bank servers S1 and S2, synced account balance.

**Normal operation:**
- User writes $100 transfer on S1.
- S1 immediately syncs to S2.
- User reads from S2, sees updated balance. ✓ Consistency maintained.

**During network partition (S1 ↔ S2 can't talk):**

**CP choice (consistency):**
- S1 blocks the transfer: "Can't guarantee sync to S2, so no writes."
- User gets error. No service, but data is consistent. (Banks do this.)

**AP choice (availability):**
- S1 completes transfer immediately. S2 doesn't know yet.
- If user reads from S2, sees old balance. (Eventually sync when partition heals.)
- Service available, but temporarily inconsistent. (Social media does this.)

## Replication and Consistency Models

### Why replicate?
- **Fault tolerance:** If one replica fails, others serve requests.
- **Performance:** Read from closest replica.
- **Availability:** More replicas = higher uptime.

### Replication strategies

#### Master-Slave (Primary-Replica)
- **Master:** Handles all writes.
- **Slaves:** Replicate data from master, serve reads.

```
Write request → Master → apply write
                    ↓ (replicate)
                Slave 1
                Slave 2
                Slave 3

Read request → can hit any slave
```

**Pros:** Simple, strong consistency (reads from master).
**Cons:** Master is bottleneck; slave lag (reads might be stale).

#### Multi-Master (Active-Active)
- **Multiple masters:** Each can accept writes.
- **All replicate to each other.**

```
Write on Master 1 → replicates to Master 2, Master 3
Write on Master 2 → replicates to Master 1, Master 3

Conflict handling: Last-write-wins, vector clocks, or custom logic
```

**Pros:** No single bottleneck, high availability.
**Cons:** Complex conflict resolution (what if both masters receive conflicting writes?).

### Consistency models

#### Strong Consistency
Every read reflects all previous writes.

```python
write(x = 5)  # Write to server A
read(x)       # Read from server B, still returns 5 (always up-to-date)
```

**Cost:** High latency (must wait for all replicas to sync).

#### Eventual Consistency
Replicas may be temporarily out of sync, but eventually converge to same state.

```python
write(x = 5)  # Write to server A
read(x)       # Read from server B, might return stale value
# After a few seconds, all replicas sync and read(x) returns 5
```

**Cost:** Temporary inconsistency, but low latency and high availability.

## Sharding (Partitioning)

Split data across multiple machines by a shard key so each machine holds a subset.

### Why shard?
- **Horizontal scaling:** Single DB can't handle billion rows; split across 10 DBs.
- **Parallel processing:** Each shard handles queries independently → lower latency.
- **Isolation:** Failure in one shard doesn't affect others.

### Sharding strategies

#### Range-based
Shard by key ranges. Example: User IDs 0-1M on Shard 1, 1M-2M on Shard 2.

```
User ID 50 → Shard 1
User ID 1.5M → Shard 2
```

**Pros:** Simple logic.
**Cons:** Can become unbalanced (e.g., active users all in ID 0-1M range).

#### Hash-based
Apply hash function to key. Shard = hash(key) % num_shards.

```python
def shard_for_user(user_id):
    return hash(user_id) % num_shards

shard = shard_for_user(user_id=12345)  # returns 0, 1, 2, ... depending on num_shards
```

**Pros:** Balanced distribution.
**Cons:** Adding/removing shards requires re-hashing (costly); use consistent hashing.

#### Consistent Hashing
Map keys to positions on a ring. Replicas arranged on same ring.

```
Imagine a circular clock with 12 hours.
- Key "user_123" hashes to hour 3.
- Server A is at hour 3, Server B at hour 7, Server C at hour 10.
- "user_123" goes to nearest server clockwise: Server A.

If Server A dies, keys move to next server (B), others unaffected.
If add Server D at hour 5, only keys between 3-5 shift.
```

**Pros:** Minimal re-hashing on server change.
**Cons:** More complex.

## Consensus and Leader Election

### Problem
In distributed systems, nodes must agree on a value (e.g., "who is the leader?").
Network delays and failures make this hard.

### Leader Election
One node becomes the leader and coordinates decisions. If leader dies, elect new one.

#### Algorithms

**Bully Algorithm:**
- Every node has priority (e.g., by ID).
- When leader dies, highest-priority alive node becomes leader.
- Simple but not efficient.

**Raft:**
- Divides time into "terms" (like election rounds).
- Nodes vote for candidate; majority vote wins.
- Leader stays in power until failure.
- Ensures safety (same decision across all nodes) and liveness (eventually progress).

```
Term 1: Server A elected leader
Term 2: Server A fails; Server B elected leader
...
```

Used in: etcd, Consul, CockroachDB.

### Quorum-Based Consensus
Require majority to agree on decision.

```python
def write(key, value):
    # Write to leader + 2 replicas
    responses = [
        leader.write(key, value),
        replica1.write(key, value),
        replica2.write(key, value)
    ]
    
    # Require 2/3 success (quorum)
    if sum(1 for r in responses if r == success) >= 2:
        return success
    else:
        return failure
```

**Guarantees:** If quorum acknowledges write, can read from any node and see the write (as long as any quorum node is alive).

## Common Distributed Systems Problems & Solutions

### 1. Split Brain / Network Partition

**Problem:** Network link fails, cluster splits into two parts that can't communicate. Both parts think they're the real cluster; both may accept writes → conflicting data.

**Solutions:**
- **Quorum-based decision:** Require majority of nodes to agree. Minority partition stops serving requests.
- **Fencing:** Use external arbiter (e.g., shared storage) to "fence off" minority partition.
- **Unique leader ID:** Only node with highest ID in majority partition can be leader.

**Example:** In 5-node cluster, if split 3-2, the 3-node partition continues; 2-node partition stops.

### 2. Cascading Failures

**Problem:** One overloaded service fails. Upstream services retry, overloading more services. Entire system collapses like dominoes.

**Solutions:**
- **Circuit breaker:** Stop retrying if downstream service is failing; fail fast instead.
- **Bulkheads:** Isolate failures. If service A fails, don't let it bring down service B (separate thread pools, timeouts).
- **Load shedding:** Reject low-priority requests when overloaded to protect high-priority ones.
- **Auto-scaling:** Add servers when load increases before cascade starts.

### 3. Distributed Consensus / Byzantine Generals Problem

**Problem:** Multiple nodes must agree on a value, but some may be faulty/malicious. How to ensure consensus?

**Solutions:**
- **Paxos:** Complex but proven consensus algorithm for non-Byzantine faults.
- **Raft:** Simpler consensus, preferred in practice.
- **Byzantine Fault Tolerance (BFT):** For malicious nodes, but expensive; mainly used in blockchain.

### 4. Data Consistency in Replication

**Problem:** Primary and replicas out of sync. Read from replica sees stale data. Write goes to primary, replicas lag.

**Solutions:**
- **Read from primary:** Slower, but always consistent.
- **Read-after-write consistency:** After write, always read from primary for that key. Other reads from replica.
- **Eventual consistency with quorum reads:** Read from majority of replicas; if any has the write, return it.
- **Version vectors / Lamport clocks:** Track causal ordering to detect stale reads.

### 5. Distributed Transactions / ACID across multiple databases

**Problem:** Transaction spans multiple databases. One commits, another fails. How to keep consistency?

**Solutions:**
- **Two-Phase Commit (2PC):** Phase 1: all nodes prepare. Phase 2: all commit or all rollback. Slow, blocks on failures.
- **Saga Pattern:** Break transaction into steps. Each step is local transaction. If step N fails, compensating transactions undo steps N-1, N-2, etc. (Event-driven or orchestration-based.)
- **Event Sourcing:** Instead of storing state, store all events. Rebuild state by replaying events. Naturally supports rollback and replay.

### 6. Availability vs Consistency (CAP tradeoff)

**Problem:** During network partition, can't have both consistency and availability.

**Solutions:**
- **For financial systems:** Choose consistency. Block writes if can't reach all replicas.
- **For social media:** Choose availability. Accept temporary inconsistency; eventually sync.
- **Quorum reads/writes:** Balance the two. Write to majority, read from majority. Guarantees consistency even if some nodes fail.

### 7. Distributed Tracing and Debugging

**Problem:** Request spans multiple services across data centers. Hard to debug latency or failures.

**Solutions:**
- **Correlation IDs:** Assign unique ID to each request; pass through all services. Logs include ID.
- **Distributed tracing (Jaeger, Zipkin):** Automatically record request flow across services, latencies per service.
- **Centralized logging (ELK stack):** Collect logs from all services; search by correlation ID.
- **Metrics and alerting (Prometheus):** Track latency, error rate, CPU per service. Alert on anomalies.

### 8. Handling Idempotency in Distributed Systems

**Problem:** Network fails mid-request. Client retries. Server processes same request twice → duplicate effects (charged twice, created 2 orders).

**Solutions:**
- **Idempotent keys / Request IDs:** Client sends unique ID per request. Server deduplicates by ID.
- **Idempotent operations:** Design operations so repeated execution has same effect (PUT, DELETE are naturally idempotent; POST is not).
- **Deduplication cache:** Store (request_id, response) in cache. On retry, return cached response.

**Example:**
```python
@app.post("/orders")
async def create_order(request: CreateOrderRequest, idempotency_key: str):
    # Check if already processed
    cached = cache.get(f"order:{idempotency_key}")
    if cached:
        return cached  # Return same response
    
    # Create order
    order = db.insert_order(...)
    response = {'id': order.id}
    
    # Cache for 24 hours
    cache.set(f"order:{idempotency_key}", response, 86400)
    return response
```

### 9. Distributed Locking

**Problem:** Multiple services accessing shared resource (e.g., file, database row). Need exclusive access.

**Solutions:**
- **Optimistic locking:** Use version numbers. If version changed, conflict detected.
- **Pessimistic locking:** Lock resource before accessing. Other threads wait.
- **Distributed lock with Redis:** Use `SET ... NX EX` for atomic lock + TTL.
- **Lease-based locking:** Lock expires after TTL; prevents deadlocks.

**Example (Redis):**
```python
import redis
import time
import uuid

client = redis.StrictRedis()

def acquire_lock(key, ttl=10):
    lock_id = str(uuid.uuid4())
    acquired = client.set(key, lock_id, nx=True, ex=ttl)
    return lock_id if acquired else None

def release_lock(key, lock_id):
    lua_script = """
    if redis.call("get", KEYS[1]) == ARGV[1] then
        return redis.call("del", KEYS[1])
    else
        return 0
    end
    """
    client.eval(lua_script, 1, key, lock_id)

# Usage
lock_id = acquire_lock("resource:user_123")
if lock_id:
    try:
        # Do work with exclusive access
        db.update_user(user_123, ...)
    finally:
        release_lock("resource:user_123", lock_id)
```

### 10. Message Queue for Async Processing

**Problem:** Long-running task blocks HTTP request. User waits. Bad experience.

**Solutions:**
- **Message queue (RabbitMQ, Kafka):** Enqueue task, return immediately. Worker processes asynchronously.
- **Dead letter queue:** If worker fails repeatedly, move to DLQ for manual review.
- **Retries with backoff:** Exponential backoff (1s, 2s, 4s, 8s...) reduces thundering herd.
- **Ordering guarantees:** For tasks that must run in order, use single queue per entity (e.g., per user).

## Interview Practice Questions

### Distributed Systems: Consensus & Replication

1. **Explain CAP theorem. Give examples for each combination (CP, AP).**
   - **CP (Consistency + Partition Tolerance):** Bank account. Block transactions if can't sync replicas.
   - **AP (Availability + Partition Tolerance):** Social media. Always respond, eventual consistency.
   - **Trade-off:** Can't have all three. Network partitions happen; choose C or A.

2. **How would you design a distributed key-value store?**
   - **Partitioning:** Consistent hashing. Key hashes to node.
   - **Replication:** Store key on node + 2 replicas for fault tolerance.
   - **Consistency:** Quorum read/write. Write to leader + replicas; require majority ACK.
   - **Leader election:** Use Raft or Paxos.

3. **Explain master-slave replication. What's a downside?**
   - Master accepts writes; slaves replicate; slaves serve reads.
   - Downside: Slave lag. Read from slave might return stale data.
   - Mitigation: Read from master after write, or wait for replica to catch up.

4. **How to handle leader failure in master-slave setup?**
   - Detect failure (heartbeat timeout).
   - Promote highest-priority slave to master.
   - Notify clients of new master.
   - Synced slaves catch up; unsynced slaves discarded.
   - Use Raft or external arbiter (e.g., Zookeeper) to prevent split-brain.

5. **Explain Raft consensus algorithm at high level.**
   - Time divided into terms (like election rounds).
   - Nodes vote for leader candidate.
   - Leader elected if gets majority vote.
   - Leader sends heartbeats to keep authority.
   - On leader failure, new election starts.

6. **What's "split-brain"? How to prevent?**
   - **Problem:** Network partition. Both halves think they're real cluster. Both accept writes.
   - **Solution:** Require quorum (majority). Minority partition stops serving requests.
   - Example: 5-node cluster splits 3-2. Only 3-node partition continues.

7. **Design a distributed counter (like view count).**
   - **Naïve:** Single DB row. Bottleneck.
   - **Solution:** Use local counters on each server. Periodically flush to DB.
   - **Accuracy tradeoff:** Eventually consistent (exact count after flush).
   - **Scale:** Billions of increments/second across world.

8. **How to ensure "read-after-write" consistency in distributed system?**
   - After write, always read from master (or wait for replica).
   - Or: store write timestamp; read from replica only if replica's lamport clock >= timestamp.
   - Trade-off: Slower reads, but always see your writes.

### Distributed Systems: Problems & Solutions

1. **How to prevent cascading failures in microservices?**
   - **Circuit breaker:** Stop calling failing service; fail fast.
   - **Bulkheads:** Isolate failures (separate thread pools per service).
   - **Load shedding:** Reject low-priority requests during overload.
   - **Retry with backoff:** Exponential backoff (1s, 2s, 4s, 8s...).
   - **Monitoring/alerting:** Detect anomalies early.

2. **How to handle distributed transactions (spanning multiple DBs)?**
   - **2PC (Two-Phase Commit):** Phase 1: prepare. Phase 2: commit/rollback. Blocks on failures.
   - **Saga Pattern:** Break into steps. If step fails, compensating transactions undo prior steps.
   - **Event sourcing:** Log all events. Rebuild state by replaying. Naturally supports rollback.

3. **How to handle network latency in distributed system?**
   - **Async:** Don't wait for response; use callbacks/futures.
   - **Caching:** Cache frequently accessed data locally.
   - **Batching:** Group multiple requests into one.
   - **Compression:** Reduce payload size.
   - **CDN:** Serve from location near user.

4. **How to implement "exactly-once" semantics (no duplicates)?**
   - **Idempotent keys:** Client sends unique ID per request.
   - **Deduplication cache:** Store (request_id, response). On retry, return cached response.
   - **Idempotent operations:** Design operations so repeated execution = same effect (PUT, DELETE).

5. **How would you implement distributed tracing for debugging?**
   - Assign correlation ID to each request.
   - Pass ID through all services.
   - Log with correlation ID.
   - Use distributed tracing tool (Jaeger, Zipkin) to visualize request flow.

## Key Takeaways for Interviews

- **CAP theorem:** Choose between consistency and availability during partitions.
- **Replication:** Master-slave for simplicity, multi-master for availability.
- **Consensus:** Raft is preferred over Paxos for simplicity.
- **Sharding:** Use consistent hashing to minimize re-hashing.
- **Common problems:** Split-brain, cascading failures, distributed transactions.
- **Solutions:** Quorum-based decisions, circuit breakers, saga pattern.
- **Debugging:** Correlation IDs, distributed tracing, centralized logging.
