# Day 30: Aurora Database - Architecture and Features

**Date:** December 16, 2025  
**Focus:** Aurora database architecture, features, and decision framework  
**Time Investment:** 90 minutes  
**Phase 1 Progress:** Day 30 of 42 (71.4% complete, 12 days remaining)

---

## What I Learned

### Aurora Architecture: The Game-Changer

**Key Differentiator:** Aurora uses SHARED storage architecture vs RDS's separate storage per instance.

**RDS Architecture:**
- Primary instance has its own EBS volume
- Each read replica has its own separate EBS volume (COPY of data)
- Data must be replicated from primary storage to replica storage
- Result: Replication lag (seconds to minutes)

**Aurora Architecture:**
- Primary + all read replicas read from SAME shared storage cluster
- Storage cluster: 6 copies of data across 3 AZs
- Result: <10ms replication lag (essentially real-time)

**Why this matters:**
- Faster failover (30 seconds vs RDS's 1-2 minutes)
- Near-zero replica lag (<10ms vs seconds/minutes)
- Auto-scaling storage (10GB to 128TB, no manual intervention)
- Self-healing storage (corrupted blocks automatically repaired)

---

### Aurora vs RDS: When to Use Each

**Choose Aurora when:**
- Need >5 read replicas (Aurora supports up to 15)
- Failover must be <1 minute (Aurora = 30 seconds)
- Storage >64TB or unpredictable growth (Aurora auto-scales to 128TB)
- 99.99% uptime SLA required (Aurora's faster failover enables this)
- Global presence needed (Aurora Global Database <1s cross-region lag)

**Choose RDS when:**
- Database <64TB with predictable growth
- Budget-sensitive (RDS is 20% cheaper than Aurora)
- Need database engines Aurora doesn't support (Oracle, SQL Server, MariaDB)
- ≤5 read replicas sufficient
- 1-2 minute failover acceptable

**Critical trap:** Aurora ONLY supports MySQL and PostgreSQL. For other engines, must use RDS.

---

### Aurora-Specific Features

**1. Aurora Global Database**
- Cross-region replication with <1 second lag
- Up to 5 secondary regions
- Failover to secondary region in <1 minute
- Use case: Global applications with users in multiple continents

**2. Aurora Serverless v1 vs v2**
- v1: Scales to zero (pauses after inactivity), 30-second cold start, good for dev/test
- v2: Instant scaling, always warm, fine-grained scaling (0.5 ACU increments), good for production with variable load
- Standard Aurora (provisioned): Choose instance size, always running, best for predictable workloads

**Important distinction:** Standard Aurora is NOT serverless. Must provision instances.

**3. Aurora Backtrack (MySQL only)**
- Time travel up to 72 hours back in time
- Rewind database to point before bad query/delete
- Same cluster, same endpoint (no new database needed)
- Recovery time: ~1 minute

**4. Aurora Clone**
- Creates copy of database in ~10 minutes
- Uses copy-on-write (only stores differences from original)
- Cost: Pay only for differences, not full database size
- Use case: Dev/test environments, what-if analysis

**5. Aurora Custom Endpoints**
- Route different workloads to different read replicas
- Example: User queries → small replicas, analytics → large replica
- Prevents heavy analytics from impacting user experience
- This is called "workload isolation"

**6. Aurora ML Integration**
- Call SageMaker models or Comprehend from SQL queries
- No ETL pipelines needed
- Use case: Real-time sentiment analysis, fraud detection

---

### Aurora Endpoints (Critical for Applications)

**Cluster Endpoint (Writer):**
- Points to primary instance
- Use for: INSERT, UPDATE, DELETE (all writes)
- Automatically updates to new primary after failover

**Reader Endpoint:**
- Load balances across ALL read replicas
- Use for: SELECT queries (all reads)
- Aurora manages load balancing automatically

**Application pattern:**
```python
# Writes → Cluster endpoint
write_conn = connect("my-cluster.cluster-abc123.us-east-1.rds.amazonaws.com")
write_conn.execute("INSERT INTO users VALUES (...)")

# Reads → Reader endpoint  
read_conn = connect("my-cluster.cluster-ro-abc123.us-east-1.rds.amazonaws.com")
read_conn.execute("SELECT * FROM users WHERE id = 123")
```

**Key rule:** If SQL is SELECT → reader endpoint. If SQL is INSERT/UPDATE/DELETE → cluster endpoint.

---

## Key Corrections from Knowledge Checks

### Correction #1: Aurora Replicas Share Storage
**Initial misconception:** Aurora replicas have separate copies of data like RDS.  
**Reality:** Aurora replicas read from the SAME shared storage cluster. This is THE fundamental architectural difference that enables <10ms lag.

### Correction #2: Standard Aurora ≠ Serverless
**Initial misconception:** Aurora is serverless by default.  
**Reality:** Standard Aurora requires provisioned instances (db.r5.large, etc.). Aurora Serverless v1/v2 are separate products for specific use cases (variable workloads, dev/test).

### Correction #3: Login Queries Use Reader Endpoint
**Initial misconception:** Login queries should use cluster (writer) endpoint.  
**Reality:** Login queries are SELECT operations (reads), so they use reader endpoint. Only INSERT/UPDATE/DELETE use cluster endpoint.

---

## Real-World Decision Example

**Scenario:** PostgreSQL database, 45TB current size, growing 3TB/month, need 8 read replicas, 99.99% uptime SLA, flexible budget.

**Decision:** Aurora PostgreSQL

**Reasoning:**
1. Will exceed 64TB in ~6 months (45TB + 3TB/month × 6 = 63TB). RDS max is 64TB. Aurora auto-scales to 128TB.
2. Need 8 read replicas. RDS max is 5. Aurora supports up to 15.
3. 99.99% uptime = 52 minutes downtime/year allowed. Aurora's 30-second failover vs RDS's 1-2 minutes provides necessary buffer for multiple failover events.

---

## SA Pro Exam Relevance

**Exam Weight:** Aurora appears in 10-15% of SA Pro questions.

**Common question patterns:**
- "When should you use Aurora vs RDS?" (decision framework)
- "How does Aurora achieve <10ms replica lag?" (shared storage architecture)
- "What's the maximum storage for Aurora?" (128TB auto-scaling)
- "How many read replicas does Aurora support?" (up to 15)
- "Which database engines does Aurora support?" (MySQL and PostgreSQL only - this is a trap!)
- "How do you isolate analytics workloads from user queries?" (Custom Endpoints)

**Interview relevance:** TAM L4 interviews test database architecture decisions. Being able to explain Aurora's shared storage architecture and when to use Aurora vs RDS demonstrates senior-level thinking.

---

## How This Connects to My Career Goals

Aurora knowledge is critical for TAM L4 role because:
- Enterprise customers run large databases requiring high availability
- TAMs must recommend appropriate database solutions for customer requirements
- Understanding cost vs performance trade-offs (Aurora 20% premium for specific benefits)
- Explaining complex architectures (shared storage) to non-technical stakeholders

---

## Next Steps

**Phase 1 Remaining:** 12 days (Days 31-42)

**Tomorrow (December 17):** 3 sessions covering Days 31-33
- Advanced monitoring patterns
- Container services (ECS basics)
- Database migration strategies

**Phase 2 Start:** January 1, 2026 - SA Professional intensive prep

---

## Reflections

Aurora's shared storage architecture is elegant - by decoupling compute from storage, AWS solved the replication lag problem that plagued traditional databases. The 6-copy distributed storage with automatic healing demonstrates how cloud-native design can fundamentally improve on traditional architectures.

**Key insight:** Not all problems need the most powerful solution. Aurora is 20% more expensive and only makes sense when you need its specific benefits (>5 replicas, >64TB storage, <30s failover, global presence). RDS is still the right choice for many workloads.

Understanding when NOT to use a technology is as important as understanding when to use it.
