# 🚀 AWS Learning Journey

![AWS Cloud Practitioner](https://img.shields.io/badge/AWS-Cloud%20Practitioner-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![AWS Solutions Architect Associate](https://img.shields.io/badge/AWS-Solutions%20Architect%20Associate-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![AWS AI Practitioner](https://img.shields.io/badge/AWS-AI%20Practitioner-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![AWS Machine Learning Engineer](https://img.shields.io/badge/AWS-ML%20Engineer%20Associate-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)

My 90-day intensive program to transition from IT Support Engineer to Solutions Architect.

## 📅 Timeline

- **Start Date:** October 13, 2025
- **Target Completion:** January 7, 2026
- **Goal:** Solutions Architect or TAM role ($110-130k)
- **Current Status:** Day 6 of 90 - Week 1 Complete ✅

## 📊 Progress Tracker

- **Certifications:** 4/4 AWS Associate-level certs ✅
- **Projects Completed:** 1/10 ✅
- **GitHub Commits:** Daily streak active 🔥
- **Next Cert Target:** AWS Certified Developer - Associate (December 2025)

**Last Updated:** October 18, 2025

## 🏗️ Recent Projects

### Week 1 Capstone: S3 + CloudFront Static Website
**[Live Demo](https://d3sow6jxmgyxlc.cloudfront.net/)** | [Documentation](2025-10-18.md)

- Built production-grade static website infrastructure
- Configured S3 bucket for static website hosting
- Implemented CloudFront CDN for global content delivery
- Secured with HTTPS using CloudFront SSL certificate
- Configured IAM bucket policies for public read access

**Tech Stack:** Amazon S3, CloudFront, IAM

**Architecture:**
```
┌─────────┐      ┌──────────────┐      ┌────────────┐
│  Users  │─────>│  CloudFront  │─────>│ S3 Bucket  │
│ Global  │      │ Edge Locations│      │  (Origin)  │
└─────────┘      └──────────────┘      └────────────┘
   HTTPS              Cache                  HTTP
```

**Build Time:** 30 minutes  
**Key Learning:** Dual-layer S3 permissions (public access settings + bucket policy)

## 📚 What I'm Learning This Week

**Week 2 Focus:** EC2 and VPC Fundamentals
- Launching and managing EC2 instances
- SSH access and remote server management
- Security groups and network access control
- Public vs private subnet architecture
- Building web servers on EC2

## 🗓️ Planned Projects

### Week 1-2: Foundation
- [x] Static website on S3 + CloudFront
- [ ] EC2 web server deployment
- [ ] VPC and networking basics

### Week 3-4: Serverless
- [ ] URL shortener (Lambda + DynamoDB)
- [ ] Image resizer (Lambda + S3)
- [ ] REST API with API Gateway

### Week 5-6: Advanced Architecture
- [ ] EC2 Auto Scaling setup
- [ ] Multi-tier application with RDS
- [ ] Load balancing with ALB

### Week 7-8: Automation
- [ ] Python boto3 automation scripts
- [ ] AWS CLI scripting
- [ ] Infrastructure as Code

### Week 9-10: Integration
- [ ] CI/CD pipeline
- [ ] Full-stack serverless application
- [ ] Multi-service integration project

### Week 11-12: Capstone
- [ ] Complex real-world application
- [ ] Complete architecture documentation
- [ ] Portfolio presentation ready

## 🛠️ Resources

- [AWS Free Tier](https://aws.amazon.com/free/)
- [boto3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
- [AWS Architecture Blog](https://aws.amazon.com/blogs/architecture/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)

## 💡 Why This Journey?

I'm documenting my transition from IT Support Engineer to Solutions Architect to build a portfolio of real AWS projects. Theory and certifications open doors, but hands-on building gets you through them. Every project here is production-grade infrastructure deployed on AWS, not tutorials or simulations.

**The Goal:** Land an SA or TAM role making $110-130k by demonstrating I can both architect solutions AND build them.

## 📬 Connect

- **GitHub:** [Christapherson](https://github.com/Christapherson)
- **LinkedIn:** [Connect with me](https://linkedin.com/in/christapherson) *(update with your actual LinkedIn)*

---

**Daily Progress:** Check the dated `.md` files in this repo for detailed day-by-day learning documentation.
