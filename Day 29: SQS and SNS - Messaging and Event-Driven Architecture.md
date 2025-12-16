# Day 29: SQS and SNS - Messaging and Event-Driven Architecture

**Date:** December 14, 2025  
**Focus:** Event-driven architecture, message queues, pub/sub patterns  
**Time Investment:** 90 minutes  
**Phase 1 Progress:** Day 29 of 42 (69% complete)

---

## What I Learned

### Event-Driven Architecture Fundamentals

**Problem:** Synchronous processing makes users wait for every operation to complete. If one service fails, the entire request fails.

**Solution:** Decouple services using message queues (SQS) and pub/sub (SNS). User gets fast response, background tasks process asynchronously.

**Key Benefit:** Failure isolation - if email service is down, order still succeeds.

---

### SQS (Simple Queue Service)

**Core Concepts:**
- **Visibility Timeout:** When consumer receives message, it becomes invisible to other consumers. If processing fails, visibility timeout expires and message becomes available for retry. Set to Lambda timeout + buffer.
- **Dead Letter Queue (DLQ):** After maxReceiveCount retries (e.g., 3 attempts), failed messages move to DLQ for manual investigation by ops team.
- **Long Polling:** Set WaitTimeSeconds=20 to reduce API calls and cost. Consumer waits up to 20 seconds for messages instead of returning immediately if queue is empty.

**Queue Types:**
- **Standard:** Unlimited throughput, best-effort ordering, may deliver duplicates. Use for: video processing, email sending, log processing.
- **FIFO:** Strict ordering, exactly-once delivery, 3,000 msg/sec limit. Use for: banking transactions, inventory updates, chat messages.

**Critical Understanding:** Both Standard AND FIFO support DLQ. FIFO is NOT required for DLQ support.

---

### SNS (Simple Notification Service)

**Pub/Sub Pattern:** One publisher broadcasts to many subscribers simultaneously.

**Subscription Types:** Email, SMS, Lambda, SQS, HTTP endpoints

**Message Filtering:** Subscribers define filter policies to receive only relevant messages based on message attributes.

Example:
```json
// Only receive high-value orders
{
  "order_value": [{"numeric": [">=", 1000]}]
}
```

**Benefit:** Reduces unnecessary Lambda invocations and processing costs by filtering at SNS level before delivery.

---

### SNS → SQS Fan-Out Pattern (Production Standard)

**Architecture:**
```
Event occurs
    ↓
Publish to SNS Topic
    ↓
┌────────┼────────┬────────┐
↓        ↓        ↓        ↓
Queue1  Queue2  Queue3  Queue4
↓        ↓        ↓        ↓
Lambda1 Lambda2 Lambda3 Lambda4
```

**Benefits:**
1. Fast API response (publish to SNS takes ~10ms)
2. Failure isolation (one Lambda fails, others continue)
3. Independent scaling (scale each Lambda independently)
4. Easy to extend (add new workflow = add new queue subscription)
5. Durable retry (SQS handles retries with visibility timeout + DLQ)

**Why SNS → SQS → Lambda instead of SNS → Lambda directly?**
- SQS provides visibility timeout control
- SQS provides configurable retry logic
- SQS buffers messages during traffic spikes
- SQS enables Dead Letter Queue pattern

---

## Real-World Architecture Examples

### Example 1: Video Processing Platform
```
S3 Upload Event
    ↓
SNS Topic "VideoUploaded"
    ↓
├─→ [1080p Queue - Standard] → 1080p Lambda
├─→ [720p Queue - Standard] → 720p Lambda  
├─→ [480p Queue - Standard] → 480p Lambda
├─→ [Thumbnail Queue - Standard] → Thumbnail Lambda
└─→ [Database Queue - Standard] → DB Update Lambda
```

**Why Standard queues?** Order doesn't matter (720p can finish before 1080p), need high throughput.

**Completion Tracking:** Each Lambda writes to DynamoDB. Aggregator Lambda checks when all 5 tasks complete, then sends "Video Ready" email via SNS.

---

### Example 2: E-commerce Order Processing
```
Customer clicks "Place Order"
    ↓
[Payment Queue - FIFO] → Payment Lambda (charge credit card)
    ↓
Payment SUCCESS → Publish to SNS "OrderPlaced"
    ↓
├─→ [Inventory Queue - FIFO] → Update stock count
├─→ [Fraud Queue - Standard] → Async fraud check (10 seconds)
├─→ [Email Queue - Standard] → Confirmation email
├─→ [SMS Queue - Standard] → SMS notification
├─→ [Analytics Queue - Standard] → Track metrics
└─→ [Warehouse Queue - Standard] → Ship order
     ↑ (filtered: order_value < $1,000)
    OR
    [Manual Review Queue - Standard] → Human approval
     ↑ (filtered: order_value >= $1,000)
```

**Why FIFO for Payment?** Prevents duplicate charges (deduplication by order ID).

**Why FIFO for Inventory?** Prevents overselling (process stock updates in exact order).

**Why Standard for everything else?** Order doesn't matter, need high throughput, occasional duplicates acceptable.

**SNS Filtering:** High-value orders (>$1,000) automatically route to manual review queue. Regular orders auto-ship to warehouse.

---

## Key Learnings & Corrections

### Correction #1: DLQ Support
**Initial misconception:** Only FIFO queues support DLQ.  
**Reality:** Both Standard and FIFO queues support DLQ. Queue type is chosen based on ordering requirements, not DLQ support.

### Correction #2: DLQ Purpose
**Initial misconception:** DLQ retries failed messages automatically.  
**Reality:** DLQ is where messages go to DIE after retries fail. Retries happen automatically in the SOURCE queue via visibility timeout. DLQ is for manual investigation.

### Correction #3: Visibility Timeout Configuration
**Initial approach:** Always set to 15 minutes (Lambda's max timeout).  
**Better approach:** Set to Lambda timeout + 10-20% buffer. If Lambda timeout is 5 minutes, set visibility timeout to 6 minutes. Faster failures = faster retries.

### Correction #4: Service Outage Handling
**Initial approach:** Implement multi-region failover for temporary service outages.  
**Better approach:** SQS automatically buffers messages and retries when service recovers. Multi-region failover is for disaster recovery, not temporary outages.

---

## How This Connects to My Portfolio Projects

### URL Shortener Enhancement Opportunity
**Current:** Synchronous API - user waits for DynamoDB write + response.

**Enhanced with SQS/SNS:**
```
User creates short URL
    ↓
API Gateway → Lambda → DynamoDB → Return response (< 200ms)
    ↓
Publish to SNS "URLCreated"
    ↓
├─→ [Email Queue] → Send confirmation email
├─→ [Analytics Queue] → Log to analytics system
└─→ [QR Queue] → Generate QR code asynchronously
```

**Benefit:** User gets response in <200ms instead of waiting for email/analytics/QR generation.

### High Availability Web App Enhancement
**Current:** Single-region deployment with ALB + Auto Scaling + RDS.

**Enhanced with SQS/SNS:**
```
User submits form
    ↓
Publish to SNS "FormSubmitted"
    ↓
├─→ [Processing Queue] → Process form data
├─→ [Email Queue] → Send confirmation email
└─→ [CRM Queue] → Update CRM system
```

**Benefit:** Decoupled processing, failure isolation, easy to add new integrations.

---

## SA Pro Exam Relevance

**Exam Weight:** SQS and SNS appear in approximately 15-20% of SA Pro questions, particularly in:
- Domain 1: Design Solutions for Organizational Complexity (decoupling architectures)
- Domain 3: Continuous Improvement for Existing Solutions (modernizing monoliths)

**Key Patterns for Exam:**
1. SNS → SQS fan-out (one event, multiple workflows)
2. Standard vs FIFO queue selection (ordering vs throughput)
3. DLQ configuration (maxReceiveCount, visibility timeout)
4. Message filtering (reduce Lambda invocations)
5. Visibility timeout calculation (Lambda timeout + buffer)

**Interview Relevance:** TAM L4 interviews test "How would you architect X?" scenarios. Being able to explain event-driven architectures, queue selection rationale, and failure handling demonstrates senior-level thinking.

---

## Next Steps

**Remaining Phase 1 Days:** 13 days (Days 30-42)

**Upcoming Topics:**
- Advanced CloudWatch monitoring patterns
- Container services (ECS/EKS fundamentals)
- Database deep dive (Aurora, DynamoDB advanced)
- Portfolio project enhancements

**Phase 2 Start:** Day 43 (November 25, 2025) - SA Professional intensive prep begins

---

## Reflections

Today's session reinforced that **architecture is about trade-offs, not absolutes**. Standard vs FIFO isn't "one is better" - it's "which trade-offs fit this use case?"

The SNS → SQS fan-out pattern solves a problem I've seen at Amazon: monolithic functions that do too much and fail catastrophically. Decoupling via messaging creates resilient systems where failures are isolated and workflows scale independently.

**Key insight:** Good architecture makes failure handling automatic (SQS retry + DLQ) instead of requiring custom retry logic in every Lambda function.
