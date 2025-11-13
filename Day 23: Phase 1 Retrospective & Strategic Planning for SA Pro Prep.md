# Day 23: Phase 1 Retrospective & Strategic Planning for SA Pro Prep

**Date:** November 13, 2025  
**Phase:** 1 of 3 (Portfolio Building)  
**Progress:** Day 23 of 42 (55% complete)  
**Days until Phase 2:** 19 days (SA Pro prep starts November 25, 2025)

---

## 📊 Current Status Check

**Portfolio Projects Completed:** 4/10
1. ✅ **Project #1:** S3 + CloudFront Portfolio Website (Live: https://d3sow6jxmgyxlc.cloudfront.net/)
2. ✅ **Project #2:** Serverless URL Shortener (Live API: https://3xoo2ucyxk.execute-api.us-east-2.amazonaws.com/prod)
3. ✅ **Project #3:** S3 + Lambda Image Processor (Event-driven automation)
4. ✅ **Project #4:** High Availability Web App (Multi-AZ with ALB, Auto Scaling, RDS Multi-AZ)

**AWS Certifications Held:** 4
- AWS Cloud Practitioner ✅
- AWS Solutions Architect Associate ✅
- AWS Machine Learning Associate ✅
- AWS AI Practitioner ✅

**Context:** Caught up after powerlifting state championship (Nov 1-3), anniversary travel, and life getting busy. Back on track and ready to strategically finish Phase 1 before SA Pro certification prep.

---

## 🔍 PART 1: Project Retrospective (What Did I Actually Learn?)

### Project #3: Serverless Image Processor

**What I Built:**
- S3 bucket with event notifications
- Lambda function triggered by S3 uploads
- Image processing pipeline with error handling
- Automated workflow with no servers to manage

**Technical Concepts Learned:**
- Event-driven architecture patterns
- S3 event notifications and prefix filtering
- Lambda execution context and cold starts
- IAM policies for cross-service permissions
- Error handling in serverless environments

**What Clicked:**
- Learned the complexity of original and processed images with S3 Events, Lambda Processing, Memory and why time outs are so important.

**What Was Hard:**
- It was difficult to understand how lambda layers functioned, the important of it, and troubleshooting when uploading objects to the original bucket that failed to process and WHY they failed.

**Real-World Application:**
- Automated image processing for user uploads
- Thumbnail generation for web applications
- Document processing pipelines
- Log file analysis workflows

---

### Project #4: High Availability Multi-Tier Web Application

**What I Built:**
- VPC with public/private subnets across 2 Availability Zones
- Application Load Balancer distributing traffic
- EC2 instances in Auto Scaling Group
- RDS MySQL Multi-AZ for high availability
- Security Groups implementing defense-in-depth

**Technical Concepts Learned:**
- Multi-AZ architecture for fault tolerance
- Load balancing algorithms and health checks
- VPC networking with public/private subnet design
- Security group chaining (ALB → EC2 → RDS)
- Auto Scaling policies and target groups
- RDS Multi-AZ failover mechanisms

**What Clicked:**
- The three tiered architecture, reason as to why it's architected the way it is, the importance of understanding the network flow traffic, and importance of high availability in case of AZ failure.

**What Was Hard:**
- When troubleshooting, failing to remember that when an instance is attempting to get updates from the external network (internet) I failed to remember that I'd need to attach an internet gateway to the VPC to ALLOW internet traffic to exist!

**Real-World Application:**
- Production web applications with 99.9% uptime
- E-commerce platforms handling traffic spikes
- API backends requiring high availability
- Database-driven applications with failover

---

## 📋 PART 2: Knowledge Gap Analysis for SA Professional

### ✅ Services I've Used Hands-On (Strong Foundation)

**Compute:**
- EC2 (instances, user data, SSH, security groups) ✅
- Lambda (serverless functions, triggers, IAM roles) ✅
- Auto Scaling (scaling policies, target groups) ✅

**Storage:**
- S3 (buckets, objects, event notifications, static hosting) ✅
- EBS (attached to EC2, basic understanding) ⚠️ *Need deeper dive*

**Networking:**
- VPC (subnets, route tables, Internet Gateway) ✅
- Security Groups (stateful firewall rules) ✅
- NACLs (stateless network ACLs, basic understanding) ⚠️ *Need deeper dive*
- Application Load Balancer (target groups, health checks, listeners) ✅

**Database:**
- DynamoDB (NoSQL, single-table design, on-demand billing) ✅
- RDS Multi-AZ (MySQL, high availability, automated backups) ✅

**Application Integration:**
- API Gateway (REST APIs, CORS, stages, deployment) ✅

**Content Delivery:**
- CloudFront (CDN, edge locations, SSL/TLS) ✅

**Identity & Access:**
- IAM (roles, policies, least privilege principle) ✅

---

### ❌ CRITICAL GAPS: Services Required for SA Pro (Not Touched Yet)

**Infrastructure as Code (HIGHEST PRIORITY):**
- ❌ **CloudFormation** - 70% of SA Pro exam involves IaC templates
- ❌ **AWS CDK** - Alternative IaC approach
- ❌ **StackSets** - Multi-account/region deployments

**Monitoring & Logging (ESSENTIAL):**
- ❌ **CloudWatch** - Metrics, alarms, logs, dashboards
- ❌ **CloudTrail** - API audit logging
- ❌ **CloudWatch Logs Insights** - Log analysis and querying
- ❌ **X-Ray** - Distributed tracing

**Networking Advanced (SA PRO HEAVY):**
- ❌ **Transit Gateway** - Multi-VPC connectivity
- ❌ **Direct Connect** - Hybrid cloud connectivity
- ❌ **Route 53** - DNS, traffic policies, health checks
- ❌ **VPC Peering** - Cross-VPC communication
- ❌ **PrivateLink** - Private connectivity to services
- ❌ **VPN** - Site-to-site and client VPN

**Storage Deep Dive:**
- ❌ **EFS** - Elastic File System
- ❌ **Storage Gateway** - Hybrid storage
- ❌ **S3 Glacier** - Long-term archival
- ❌ **FSx** - Managed file systems

**Messaging & Decoupling:**
- ❌ **SQS** - Message queuing
- ❌ **SNS** - Pub/sub notifications
- ❌ **EventBridge** - Event bus

**Caching:**
- ❌ **ElastiCache** - Redis/Memcached
- ❌ **DAX** - DynamoDB Accelerator

**Containers (SA PRO):**
- ❌ **ECS** - Container orchestration
- ❌ **EKS** - Kubernetes on AWS
- ❌ **Fargate** - Serverless containers

**Migration & Hybrid (SA PRO 20% weight):**
- ❌ **Database Migration Service (DMS)** - Database migrations
- ❌ **DataSync** - Data transfer
- ❌ **Snow Family** - Petabyte-scale data transfer
- ❌ **Application Discovery Service** - Migration planning

**Multi-Account Architecture (SA PRO 26% weight - HIGHEST):**
- ❌ **AWS Organizations** - Multi-account management
- ❌ **Service Control Policies (SCPs)** - Account permissions
- ❌ **Control Tower** - Landing zone automation
- ❌ **Cross-account IAM roles** - Account-to-account access

**Advanced Database:**
- ❌ **Aurora** - MySQL/PostgreSQL compatible (SA PRO heavy focus)
- ❌ **Aurora Serverless** - On-demand database
- ❌ **Aurora Global Database** - Cross-region replication
- ❌ **Database purpose selection** - When to use which DB

**Disaster Recovery:**
- ❌ **Backup strategies** - RPO/RTO requirements
- ❌ **Pilot Light** - DR pattern
- ❌ **Warm Standby** - DR pattern
- ❌ **Multi-region architectures** - Active-active/active-passive

**Security Advanced:**
- ❌ **KMS** - Key management
- ❌ **Secrets Manager** - Credential rotation
- ❌ **GuardDuty** - Threat detection
- ❌ **WAF** - Web application firewall
- ❌ **Shield** - DDoS protection
- ❌ **Inspector** - Vulnerability scanning

**Well-Architected Framework:**
- ⚠️ **5 Pillars** - Basic awareness, need deep understanding
- ❌ **Cost Optimization** - Strategies and tools
- ❌ **Performance Efficiency** - Architecture patterns
- ❌ **Reliability** - Design principles
- ❌ **Security** - Best practices
- ❌ **Operational Excellence** - DevOps practices

---

## 📅 PART 3: Strategic Plan for Days 24-42 (19 Days Remaining)

### Phase 1 Goals Before SA Pro Prep

**Primary Goal:** Build demonstrable hands-on competency in SA Associate-level services while strategically learning SA Pro prerequisites

**Secondary Goal:** Complete 2 more portfolio projects using Infrastructure as Code

**Timeline Constraint:** 19 days to cover critical gaps before Phase 2 intensive SA Pro studying begins

---

### Week-by-Week Breakdown

**Week 4: Days 24-28 (November 14-18) - IaC Foundation**

**Theme:** Infrastructure as Code + Monitoring

**Services to Learn:**
- CloudFormation (templates, stacks, parameters, outputs)
- CloudWatch (metrics, alarms, logs)
- CloudTrail (audit logging, event history)

**Daily Focus:**
- **Day 24:** CloudFormation fundamentals (template anatomy, resources, parameters)
- **Day 25:** CloudFormation hands-on (recreate VPC from Project #4 as code)
- **Day 26:** CloudWatch deep dive (metrics, alarms, dashboards for existing projects)
- **Day 27:** CloudTrail + CloudWatch Logs (audit logging, log analysis)
- **Day 28:** **PROJECT #5 START:** Deploy a CloudFormation-based infrastructure

**Learning Method:** 
- 60-90 minute focused sessions
- Hands-on labs using AWS Free Tier
- Document everything in GitHub daily

---

**Week 5: Days 29-35 (November 19-25) - Advanced Networking + Messaging**

**Theme:** Networking depth + Decoupling patterns

**Services to Learn:**
- Route 53 (DNS, hosted zones, routing policies)
- Transit Gateway (multi-VPC connectivity concepts)
- SQS (message queuing, decoupling)
- SNS (pub/sub notifications)
- ElastiCache (Redis/Memcached caching strategies)

**Daily Focus:**
- **Day 29:** Route 53 fundamentals (DNS, record types, routing policies)
- **Day 30:** Transit Gateway concepts (hub-and-spoke, multi-VPC)
- **Day 31:** SQS deep dive (standard vs FIFO, dead letter queues)
- **Day 32:** SNS + SQS integration (fanout pattern, message filtering)
- **Day 33:** ElastiCache overview (when to use Redis vs Memcached)
- **Day 34-35:** **PROJECT #5 COMPLETE:** Serverless architecture with SQS/SNS decoupling + CloudFormation deployment

**Expected Project #5:**
- Event-driven architecture using SQS for asynchronous processing
- SNS for multi-subscriber notifications
- Lambda functions processing queue messages
- All infrastructure deployed via CloudFormation
- CloudWatch monitoring with custom alarms

---

**Week 6: Days 36-42 (November 26 - December 2) - SA Pro Prerequisites + Portfolio Polish**

**Theme:** SA Pro readiness + Final preparations

**Services to Learn:**
- AWS Organizations (multi-account basics)
- Aurora vs RDS (when to use which)
- Migration tools overview (DMS, DataSync)
- Well-Architected Framework (5 pillars deep dive)

**Daily Focus:**
- **Day 36:** AWS Organizations + SCPs (multi-account architecture basics)
- **Day 37:** Aurora deep dive (Serverless, Global Database, use cases)
- **Day 38:** Migration strategies (7 Rs framework, DMS overview)
- **Day 39:** Well-Architected Framework (all 5 pillars comprehensive review)
- **Day 40:** **PROJECT #6 (OPTIONAL):** Aurora + Lambda + API Gateway serverless app
- **Day 41:** Portfolio website update (all 5-6 projects showcased with architecture diagrams)
- **Day 42:** **Phase 1 Final Review** - Prepare for SA Pro transition

---

## 🎯 PART 4: Phase 2 Transition Strategy (Day 43+)

**Phase 2 Start Date:** November 25, 2025 (Day 43)  
**Phase 2 End Date:** January 31, 2026 (Day 98)  
**Duration:** 56 days (8 weeks) for SA Pro certification

**What Changes in Phase 2:**
- **Study mode:** Video courses (Adrian Cantrill) + practice exams (Jon Bonso)
- **Time commitment:** 20-25 hours/week (vs current 10-15 hours/week)
- **Focus:** Certification-specific exam prep, not portfolio building
- **Goal:** Pass SA Professional exam by late January/early February 2026

**SA Pro Prerequisites You'll Have by Day 42:**
- ✅ CloudFormation (IaC foundation)
- ✅ CloudWatch/CloudTrail (monitoring/logging)
- ✅ Route 53 (DNS fundamentals)
- ✅ SQS/SNS (messaging patterns)
- ✅ ElastiCache (caching concepts)
- ✅ AWS Organizations basics (multi-account)
- ✅ Aurora vs RDS (database selection)
- ✅ Well-Architected Framework (5 pillars)
- ✅ 5-6 portfolio projects demonstrating integration

**What You'll Still Need to Learn in Phase 2:**
- Transit Gateway deep dive (complex routing)
- Direct Connect (hybrid connectivity)
- Advanced migration scenarios (7 Rs in detail)
- Multi-account patterns (Control Tower, cross-account IAM)
- Aurora advanced features (Global Database, Serverless v2)
- Container orchestration (ECS/EKS overview)
- Advanced security (KMS, GuardDuty, WAF)

---

## ✅ Success Metrics for Phase 1 Completion (Day 42)

**By December 2, 2025, I will have:**

**Portfolio Projects:**
- [ ] 5-6 total projects completed and documented ✅
- [ ] All projects have architecture diagrams ✅
- [ ] Portfolio website updated with all projects ✅
- [ ] At least 2 projects using CloudFormation/IaC ✅
- [ ] GitHub repository with daily documentation ✅

**Technical Competencies:**
- [ ] Hands-on experience with 20+ AWS services ✅
- [ ] Can explain multi-tier architecture design decisions ✅
- [ ] Understand event-driven serverless patterns ✅
- [ ] Know when to use different database types ✅
- [ ] Can create CloudFormation templates from scratch ✅
- [ ] Understand Well-Architected Framework principles ✅

**SA Pro Readiness:**
- [ ] Completed all planned Phase 1 learning content ✅
- [ ] Adrian Cantrill SA Pro course purchased ✅
- [ ] Jon Bonso practice exams purchased ✅
- [ ] AWS Free Tier account set up for SA Pro labs ✅
- [ ] Study schedule blocked on calendar (20-25 hrs/week) ✅

**Job Search Preparation:**
- [ ] Portfolio projects can be explained in 2-minute pitches ✅
- [ ] Resume updated with project descriptions ✅
- [ ] LinkedIn profile showcases GitHub portfolio ✅
- [ ] Can articulate "why Solutions Architect/TAM?" ✅

## 📊 Reality Check: Timeline to TAM L4 Role

**Current Date:** November 13, 2025 (Day 23)  
**Phase 2 Start:** November 25, 2025 (Day 43)  
**Phase 2 End:** January 31, 2026 (Day 98) - **SA Pro exam target**  
**Phase 3 Start:** February 1, 2026 (Day 99) - **Job applications begin**  
**Expected Job Offer:** March-April 2026 (realistic timeline per bootcamp data)

**Key Milestone:** SA Professional certification is **MANDATORY** for TAM L4 applications at Amazon (organization requirement by end of year). Passing SA Pro by late January/early February 2026 keeps you on track for Spring 2026 hiring.

---

## 🔥 Tomorrow's Focus (Day 24)

**Goal:** CloudFormation fundamentals - learn Infrastructure as Code from scratch

**Why:** CloudFormation is your #1 gap. SA Pro assumes you know IaC. 70% of exam questions involve template syntax, stack operations, and multi-account deployments.

**What we'll build:** Recreate your VPC from Project #4 as CloudFormation template (proving you understand networking AND IaC).

**Time estimate:** 90-120 minutes

---

**Energy level at end of Day 23:** 8.5 energy level :)

**Biggest insight from today's retrospective:** Building a three tiered architecture can be as simple or as complex as you'd like. It all depends on the purpose and how far you'd like to customize the experience when building.

**One commitment for Days 24-42:** Commit myself to each day at 2x the amount.

---

*Day 23 completed: November 13, 2025*  
*Next session: Day 24 - CloudFormation fundamentals*
