# Day 28: Route 53 - DNS Management & Routing Policies

**Date:** December 14, 2025  
**Phase:** 1 of 3 (Day 28 of 42 - 66.7% Complete)  
**Time Invested:** ~2 hours  
**Energy Level:** 10/10 ("lesssssss get it, grahhhhhhhhhhh")

---

## What I Learned Today

### Route 53 Fundamentals
- **DNS = Domain Name System** - translates domain names to IP addresses
- **Route 53 = AWS's highly available DNS service** (named after DNS port 53)
- **Hosted Zones** = containers for DNS records
  - Public hosted zones: Route traffic on public internet
  - Private hosted zones: Route traffic within VPCs only
  - Cost: $0.50/month per hosted zone

### DNS Record Types
- **A Record:** Domain → IPv4 address
- **AAAA Record:** Domain → IPv6 address
- **CNAME Record:** Domain → another domain (cannot use at zone apex, costs money)
- **Alias Record:** AWS-specific record for AWS resources (FREE, works at zone apex, auto-updates)
  - **Key insight:** ALWAYS use Alias for API Gateway, ALB, CloudFront, S3
- **MX Record:** Email routing
- **TXT Record:** Text data (domain verification, SPF records)

### The 6 Routing Policies

**1. Simple Routing**
- Returns single resource (or random if multiple)
- NO health checks, NO failover
- Use when: Single resource, no DR needed

**2. Weighted Routing**
- Distributes traffic by percentage (70%/30% split)
- Use when: A/B testing, blue/green deployments, canary releases
- Example: Route 10% traffic to new version, 90% to old version

**3. Latency-Based Routing**
- Routes users to AWS region with lowest network latency
- Use when: Global applications needing best performance
- Based on actual network latency, NOT geography

**4. Failover Routing** ⭐ **CRITICAL FOR DR**
- Active/passive disaster recovery with health checks
- Primary endpoint + Secondary endpoint (backup)
- Automatic switchover when primary fails
- Use when: 99.99% availability requirement, automatic DR needed
- **Key distinction:** Simple routing = NO automatic failover, Failover routing = automatic DR

**5. Geolocation Routing**
- Routes based on user's geographic location (country/continent)
- Use when: GDPR compliance, data residency laws, content localization
- **Hard boundaries** - EU users CANNOT reach US servers
- Different from latency routing (compliance vs performance)

**6. Geoproximity Routing**
- Routes based on geography with bias adjustments
- Use when: Advanced traffic shaping, capacity management
- Positive bias = pull more traffic, Negative bias = push traffic away

**Decision Tree:**
- Need DR/failover? → Failover policy
- Legal compliance for data location? → Geolocation policy
- A/B testing? → Weighted policy
- Best performance globally? → Latency policy
- Just point domain to resource? → Simple policy

### Health Checks - Making Route 53 Intelligent

**3 Types of Health Checks:**
1. **Endpoint Health Checks** - Monitor specific URL/IP
   - Protocol: HTTP, HTTPS, TCP
   - Interval: 30s (standard, $0.50/month) or 10s (fast, $1.00/month)
   - Failure threshold: Default 3 consecutive failures
   - String matching: Optional - verify response body contains specific text

2. **Calculated Health Checks** - Combine multiple health checks with AND/OR logic
   - Use when: Application has multiple components (ALB + RDS + ElastiCache)
   - Only healthy if ALL dependencies healthy
   - Prevents partial failure scenarios

3. **CloudWatch Alarm Health Checks** - Monitor CloudWatch alarm state
   - Use when: Need custom metrics (Lambda errors, DynamoDB throttling)
   - Proactive monitoring before users impacted

**Health Check Best Practices:**
- Create dedicated `/health` endpoint (lightweight, tests dependencies)
- Set appropriate TTL (60-120s for production balances speed vs cost)
- Test failover regularly (chaos engineering)
- Use string matching to verify response body, not just HTTP 200

**Cost/Benefit Analysis:**
- 2-minute detection requirement:
  - Standard 30s interval: Detects in 90s (3 × 30s), costs $0.50/month ✅
  - Fast 10s interval: Detects in 30s (3 × 10s), costs $1.00/month (overkill)
- TAM thinking: Meet SLA at lowest cost, upgrade only when requirement demands it

### CloudWatch Integration
- Health check metrics → CloudWatch dashboards
- Alarms trigger SNS notifications (email, SMS, Slack)
- Automated alert loop: Failure → Alarm → SNS → Team responds

---

## Hands-On Architecture Design

Designed multi-region failover architecture for URL Shortener project:

**Architecture Components:**
- Primary Region: us-east-1 (API Gateway + Lambda + DynamoDB)
- Secondary Region: us-west-2 (API Gateway + Lambda + DynamoDB Global Tables)
- Route 53 Failover routing with health checks on both regions
- Health checks: HTTPS, /prod/health endpoint, 30s interval, 3 failure threshold
- CloudWatch alarms for health check failures, DynamoDB throttling, Lambda errors
- Estimated cost: ~$14/month for 99.99% availability

**Key Design Decisions:**
- **Failover routing** (not Simple) - Automatic DR with health checks required for 99.99% availability
- **Alias records** (not CNAME) - Free queries, works at zone apex, AWS-native
- **30-second interval** (not 10s) - 90-second detection meets SLA, saves 50% cost
- **DynamoDB Global Tables** - Automatic replication between regions
- **TTL 30 seconds** - Fast failover without excessive DNS query costs

**Failover Flow:**
1. Primary health check fails (3 × 30s = 90 seconds)
2. Route 53 marks primary unhealthy
3. DNS responses switch to secondary region
4. Total failover time: ~2-3 minutes (health check + DNS propagation)
5. Primary recovers → automatic failback

---

## How This Connects to My Portfolio Projects

### Project #1: CloudFront Portfolio Website
**Enhancement:** Purchase custom domain (christapherson.dev), use Alias record to point to CloudFront
**Business value:** Professional portfolio URL, demonstrates DNS configuration

### Project #2: URL Shortener API
**Current:** Ugly API Gateway URL
**Production architecture:** Multi-region failover with Route 53 (designed today, not built due to cost)
**Interview talking point:** "Documented multi-region DR architecture with automatic failover, optimized for 99.99% availability while managing costs"

### Project #4: HA Web Application
**Current:** Multi-AZ within single region
**Route 53 upgrade:** Failover routing to secondary region for true disaster recovery
**Key insight:** Multi-AZ = high availability within region, Multi-region + Route 53 = disaster recovery across regions

---

## Key Insights & TAM-Level Thinking

**1. Simple vs Failover Routing Distinction (CRITICAL)**
- Simple routing = NO health checks, keeps pointing to crashed endpoint
- Failover routing = automatic DR, switches to healthy endpoint after health check failures
- Multi-region architecture REQUIRES failover routing for automatic DR

**2. Geolocation vs Geoproximity**
- Geolocation = COMPLIANCE (hard boundaries, EU data stays in EU, GDPR enforcement)
- Geoproximity = PERFORMANCE (traffic shaping with bias, no legal guarantees)
- Wrong answer in interview = "Use geoproximity for GDPR" (shows misunderstanding of compliance)

**3. CNAME vs Alias for AWS Resources**
- CNAME: Limited (no zone apex), costs money, manual IP updates
- Alias: AWS-native, free queries, works everywhere, auto-updates
- **ALWAYS use Alias for API Gateway, ALB, CloudFront, S3**

**4. Cost Optimization in Health Checks**
- Standard 30s interval meets most SLAs (90-second detection)
- Fast 10s interval only when requirement < 60 seconds
- TAM balances technical requirements with cost optimization

**5. Route 53 Appears in ~60% of Enterprise Architectures**
- Global applications need latency-based routing
- DR strategies require failover routing
- Compliance requirements use geolocation routing
- Understanding routing policies = understanding production AWS architectures

---

## Tomorrow's Focus

Continue Phase 1 service breadth:
- **Option 1:** SQS/SNS messaging patterns (event-driven architectures)
- **Option 2:** Advanced monitoring (add CloudWatch to remaining projects)
- **Option 3:** Container fundamentals (ECS/EKS intro)

**Phase 1 Status:** Day 28 of 42 (66.7% complete)
**Days Remaining:** 14 days until Phase 2 (SA Pro prep begins November 25)
**Portfolio Projects:** 4/4 complete ✅

---

## Reflections

**What clicked today:**
- Understanding WHY failover routing is required for DR (Simple routing keeps pointing to crashed endpoint)
- The distinction between compliance (geolocation) vs performance (geoproximity/latency)
- Cost/benefit analysis for health check intervals (meet SLA at lowest cost)
- Alias records are AWS magic for AWS resources (free, flexible, auto-updating)

**What needs more practice:**
- Actually implementing Route 53 configuration in AWS Console (can do this when I purchase domain)
- Building the mental model of "which routing policy for which scenario" (comes with practice)
- Understanding the full failover flow with DNS propagation timing

**TAM-level thinking demonstrated:**
- Balancing technical requirements with cost optimization
- Understanding when to use fast vs standard health checks
- Designing for 99.99% availability without over-engineering
- Explaining architectural decisions with business context
