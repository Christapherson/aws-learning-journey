# Project #4 - High Availability Web Application with Application Load Balancer

**Date:** November 13, 2025  
**Focus:** Multi-AZ Web Architecture, Application Load Balancer, RDS Integration  
**Time Invested:** ~4 hours  
**Status:** ✅ Complete and Functional

---

## Project Overview

Built a production-grade High Availability (HA) web application across two Availability Zones in AWS us-east-2 region. The architecture demonstrates fault tolerance, load distribution, and proper security segmentation.

**Live Infrastructure:**
- **Load Balancer DNS:** `ha-web-alb-[YOUR-DNS].us-east-2.elb.amazonaws.com`
- **Availability Zones:** us-east-2a, us-east-2b
- **Backend:** 2 Apache web servers
- **Database:** RDS MySQL Multi-AZ

---

## Architecture Components

### Network Layer (VPC)
- **VPC CIDR:** 10.0.0.0/16
- **Public Subnets:** 
  - 10.0.0.0/20 (us-east-2a)
  - 10.0.16.0/20 (us-east-2b)
- **Private Subnets:**
  - 10.0.128.0/20 (us-east-2a)
  - 10.0.144.0/20 (us-east-2b)
- **Internet Gateway:** Attached to VPC for public subnet internet access

### Compute Layer
- **Instance Type:** t3.micro (2 instances)
- **Operating System:** Amazon Linux 2023
- **Web Server:** Apache HTTP Server (httpd)
- **Placement:** One instance per Availability Zone
- **Access:** SSH via key pair authentication

### Load Balancing
- **Type:** Application Load Balancer (Layer 7)
- **Scheme:** Internet-facing
- **Target Group:** HTTP health checks on port 80
- **Health Check Settings:**
  - Interval: 30 seconds
  - Timeout: 5 seconds
  - Healthy threshold: 2
  - Unhealthy threshold: 2

### Database Layer
- **Engine:** MySQL 8.0
- **Instance Class:** db.t3.micro
- **Storage:** 20 GB SSD
- **Deployment:** Multi-AZ (optional configuration)
- **Network:** Private subnets only
- **Backup:** Automated snapshots enabled

### Security Architecture

**ALB Security Group (`ha-web-alb-sg`):**
- Inbound: HTTP (80) from 0.0.0.0/0
- Outbound: All traffic

**Web Server Security Group (`ha-web-server-sg`):**
- Inbound: 
  - HTTP (80) from ALB security group
  - SSH (22) from my IP
- Outbound: All traffic

**Database Security Group (`ha-web-db-sg`):**
- Inbound: MySQL (3306) from web server security group
- Outbound: All traffic

---

## Implementation Steps

### Phase 1: Foundation
1. Created SSH key pair for secure instance access
2. Built VPC with 4 subnets across 2 AZs
3. Configured Internet Gateway and route tables
4. Created 3 security groups with least-privilege rules

### Phase 2: Compute
1. Launched 2 EC2 instances in separate AZs
2. SSH'd into each instance using key pair
3. Manually installed Apache HTTP Server
4. Created custom HTML pages showing hostname and AZ

### Phase 3: Load Balancing
1. Created target group with health check configuration
2. Registered both EC2 instances as targets
3. Deployed Application Load Balancer
4. Verified health checks passing (both targets healthy)

### Phase 4: Database
1. Created DB subnet group spanning private subnets
2. Deployed RDS MySQL instance
3. Configured security group for MySQL access from web servers only
4. Verified connectivity from EC2 instances

---

## Key Technical Decisions

### Why Application Load Balancer?
- Layer 7 routing capabilities (can route based on URL paths, headers)
- Better suited for HTTP/HTTPS web applications
- Integrated health checks with auto-healing
- Foundation for future features (SSL termination, path-based routing)

### Why Multi-AZ Deployment?
- **High Availability:** Survives single AZ failure
- **Zero Downtime:** ALB automatically routes around failed instances
- **Production Ready:** Same architecture pattern used by enterprise applications

### Why Private Subnets for Database?
- **Security:** Database not accessible from internet
- **Defense in Depth:** Multiple security layers (NACLs, Security Groups, subnet isolation)
- **Best Practice:** Data layer should never be publicly exposed

---

## Challenges & Learnings

### Challenge 1: User Data Script Failures
**Problem:** Initial attempts to use User Data scripts to install Apache consistently failed. Scripts weren't executing or were encountering network issues during instance launch.

**Solution:** Pivoted to manual Apache installation via SSH after instance launch. This approach:
- Provides immediate feedback if installation fails
- Allows real-time troubleshooting
- Verifiable before registering instances with load balancer

**Learning:** User Data scripts are powerful but can fail silently. For learning/troubleshooting, manual installation provides better visibility.

### Challenge 2: EC2 Instance Connect Failures
**Problem:** AWS's browser-based Instance Connect repeatedly failed to establish connections.

**Lesson:** Always create SSH key pairs during instance launch. Relying on AWS's convenience features (Instance Connect) can block critical troubleshooting access.

### Challenge 3: Security Group Configuration Complexity
**Problem:** Initially struggled with proper security group referencing (allowing HTTP from ALB's security group, not IP range).

**Solution:** Security groups can reference other security groups as sources. This is critical for:
- Dynamic environments (instances can change IPs)
- Proper security boundaries (logical grouping)
- Scalability (don't need to update rules when adding instances)

**Syntax:** In web server SG, source is `sg-XXXXXXXX` (ALB's security group ID), not an IP range.

---

## Testing & Verification

### Load Balancer Testing
```bash
# Access via ALB DNS
curl http://ha-web-alb-XXXXXXXXXX.us-east-2.elb.amazonaws.com

# Response alternates between:
# "Hello from ip-10-0-0-XX in AZ us-east-2a"
# "Hello from ip-10-0-16-XX in AZ us-east-2b"
```

### Database Connectivity Testing
```bash
# From EC2 instance
mysql -h ha-web-db.XXXXXXXXXX.us-east-2.rds.amazonaws.com -u admin -p

# Verify databases
SHOW DATABASES;
# Output shows 'webapp' database exists
```

### Health Check Verification
- Both targets show "Healthy" status in target group
- Health checks passing consistently (2/2 checks)
- No failed health checks after 10+ minutes

---

## Production Readiness Gaps

This project demonstrates HA architecture foundations, but production deployment would require:

1. **SSL/TLS:** HTTPS listener on ALB with ACM certificate
2. **Auto Scaling:** Auto Scaling Group to handle traffic spikes
3. **Monitoring:** CloudWatch alarms for CPU, memory, failed health checks
4. **Logging:** ALB access logs to S3, CloudWatch Logs for application logs
5. **Backup Strategy:** Automated RDS snapshots, cross-region replication
6. **DNS:** Route 53 with custom domain name instead of ALB DNS
7. **WAF:** Web Application Firewall for security filtering
8. **Cost Optimization:** Reserved Instances or Savings Plans for predictable workloads

---

## AWS Services Used

- **EC2:** Virtual servers for web application
- **VPC:** Network isolation and subnet management
- **Application Load Balancer:** Layer 7 load distribution
- **RDS:** Managed MySQL database
- **Security Groups:** Stateful firewalls
- **IAM:** Key pair management for SSH access

---

## Cost Analysis

**Monthly Costs (Approximate):**
- 2 x t3.micro EC2: $12-15 (730 hours/month)
- Application Load Balancer: $16-20 (base + LCU charges)
- RDS db.t3.micro: $12-15 (Single-AZ pricing)
- Data Transfer: $1-5 (depends on traffic)
- **Total: ~$41-55/month**

**Cost Optimization:**
- Instances are in Free Tier if within first 12 months
- Can reduce to Single-AZ for development (not production)
- Stop instances when not actively testing

---

## Key Learnings

1. **Multi-AZ Architecture:** Learned how to properly distribute resources across availability zones for fault tolerance
2. **Load Balancer Mechanics:** Understanding target groups, health checks, and routing algorithms
3. **Security Group Referencing:** How to create dynamic security rules using security group IDs as sources
4. **RDS Networking:** Database deployment in private subnets with proper security isolation
5. **Troubleshooting Methodology:** When automated solutions fail (User Data, Instance Connect), fall back to fundamentals (SSH, manual commands)

---

## Next Steps

**Potential Enhancements:**
1. Add Auto Scaling Group to automatically adjust instance count based on demand
2. Implement SSL/TLS with ACM certificate
3. Create CloudFormation template to automate entire infrastructure deployment
4. Add CloudWatch monitoring and alerting
5. Build simple web application that reads/writes to RDS database
6. Implement CI/CD pipeline for application deployment

---

## Commands Reference

### Apache Installation
```bash
sudo yum update -y
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
```

### Create Custom Web Page
```bash
EC2_AVAIL_ZONE=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)
echo "<h1>Hello from $(hostname -f) in AZ $EC2_AVAIL_ZONE</h1>" | sudo tee /var/www/html/index.html
```

### MySQL Client Installation
```bash
sudo yum install -y mariadb105
```

### Database Connection
```bash
mysql -h [RDS-ENDPOINT] -u admin -p
```

---

## Reflection

Today's project took significantly longer than expected due to User Data script issues and EC2 Instance Connect failures. However, the troubleshooting process taught valuable lessons about:

- **Reliability vs Convenience:** Automated tools are convenient but can fail opaquely
- **Fundamentals Matter:** SSH key pairs and manual installation are reliable fallbacks
- **Architecture Design:** Understanding *why* resources are placed in specific subnets and security groups

**Building a HA web application from scratch reinforced that production-grade architecture requires attention to:**
- Network segmentation (public vs private subnets)
- Security layers (multiple security groups with specific rules)
- Redundancy (multiple AZs)
- Health monitoring (ALB target health checks)

This architecture pattern forms the foundation for countless AWS production workloads. The skills practiced today—VPC design, security group configuration, load balancing, and RDS deployment—are directly transferable to enterprise AWS environments.

---

**Status:** Project complete and verified working. All components healthy and responding correctly. Ready for Phase 2 (SA Professional certification study) starting Day 43.
