# Day 31: AWS Organizations and Multi-Account Architecture

**Date:** December 17, 2025  
**Focus:** Multi-account architecture, SCPs, cross-account access, Control Tower  
**Time Investment:** 120 minutes  
**Phase 1 Progress:** Day 31 of 42 (73.8% complete, 11 days remaining)  
**Energy Level:** 9/10 🔥

---

## What I Learned

### The Problem: Why Single Accounts Don't Scale

Single AWS account creates five critical problems:
1. **Blast radius:** Developer mistake in dev can affect production (no isolation)
2. **Compliance boundaries:** Cannot prove hard separation for PCI-DSS/HIPAA (IAM policies can be changed)
3. **Cost allocation:** ONE bill for entire account (must rely on error-prone tagging)
4. **Service limits:** All teams share same limits (one team affects another)
5. **Environment isolation:** Hard to prevent dev credentials from accessing prod

**Solution:** AWS Organizations with multiple accounts = physical boundaries

---

### AWS Organizations: Core Components

**1. Management Account (formerly Master Account)**
- Creates the organization
- IMMUNE to Service Control Policies (SCPs do NOT apply)
- Best practice: NEVER run workloads here (only billing and org management)
- Lock down root user with MFA + hardware token

**2. Member Accounts**
- AWS accounts that belong to organization
- Subject to SCPs
- Create via Organizations or invite existing accounts

**3. Organizational Units (OUs)**
- Folders for grouping accounts
- SCPs applied to OU affect ALL accounts inside
- Example structure: Security OU, Production OU, Development OU

**4. Consolidated Billing**
- ONE bill for entire organization with per-account breakdown
- Volume discounts apply to COMBINED usage (major cost savings)
- Reserved Instances automatically shared across accounts
- Account boundary = cost boundary (no tagging needed for allocation)

**5. Service Control Policies (SCPs)**
- Permission boundaries applied to accounts or OUs
- Define MAXIMUM permissions (not grants)
- Do NOT apply to Management Account
- Affect ALL users/roles in account, including root user

---

### Service Control Policies: How They Work

**Key Principle:** For action to succeed, need BOTH SCP allow AND IAM allow

**Formula:** Effective Permissions = SCP ∩ IAM Policy (intersection)

**SCP Inheritance:**
- Flows DOWN the tree: Root → OU → Account
- All SCPs in the path apply (cumulative restrictions)
- Example: Root allows all, Production OU denies us-west-2, Account denies RDS deletion
  - Result: Account has all restrictions from Root + OU + Account level

**Critical Rules:**
1. **Explicit DENY always wins** (cannot be overridden by any allow)
2. **SCPs do NOT grant permissions** (only limit what IAM can grant)
3. **Management Account is immune** (SCPs have zero effect on it)
4. **SCPs affect root user in MEMBER accounts** (root is NOT special in member accounts)

**Common SCP Patterns:**
- Deny regions (force us-east-1 only for compliance)
- Deny root user actions (force IAM usage)
- Protect databases (prevent RDS/DynamoDB deletion)
- Require MFA for destructive operations
- Deny leaving organization (prevent rogue admin breakaway)

---

### Cross-Account IAM Roles

**The AssumeRole Pattern:**
To access resources in Account B from Account A:

**Account B (target):**
- Creates IAM role with Trust Policy: "Allow Account A to assume this role"
- Role has Permission Policy: "What can be done after assuming"

**Account A (source):**
- User needs IAM permission: `sts:AssumeRole` for the target role ARN

**Process:**
1. User in Account A calls `aws sts assume-role --role-arn <Account B role>`
2. AWS validates BOTH: Trust Policy allows Account A + User has sts:AssumeRole permission
3. AWS returns temporary credentials (valid 15 min to 12 hours)
4. User uses temporary credentials to access Account B resources

**Key Insights:**
- Trust is ONE-WAY (Account B trusts Account A, NOT bidirectional)
- Need BOTH trust policy AND IAM permission (two-sided agreement)
- Temporary credentials expire (security best practice)
- ExternalId required for cross-organization access (prevents confused deputy)

---

### AWS Control Tower

**What it is:** Automated multi-account setup using AWS Organizations best practices

**Landing Zone:** The pre-configured environment Control Tower creates
- Security OU (Log Archive account + Audit account)
- Sandbox OU (for testing)
- CloudTrail enabled across all accounts
- AWS Config for compliance monitoring
- Cross-account IAM roles for centralized management

**Guardrails:** Automated policy enforcement
- **Preventive:** Use SCPs to BLOCK actions (fence)
- **Detective:** Use AWS Config to DETECT violations (camera)
- **Mandatory:** Cannot be disabled (protect CloudTrail, prevent root user)
- **Strongly Recommended:** Should enable (prevent public S3, detect unencrypted EBS)
- **Elective:** Optional based on requirements

**Account Factory:** Self-service account creation
- Fill form (name, email, OU)
- Control Tower automatically configures logging, guardrails, IAM roles
- Ready in 15-20 minutes

**When to use Control Tower:**
- ✅ NEW multi-account setup (greenfield deployment)
- ✅ Want AWS best practices applied automatically
- ✅ Team lacks Organizations expertise
- ❌ Existing 50+ account organization (migration nightmare)
- ❌ Need customization beyond Control Tower's blueprint

---

## Key Corrections from Knowledge Checks

### Correction #1: Management Account SCP Immunity
**Misconception:** Root user bypasses SCPs in all accounts  
**Reality:** Management Account is immune to SCPs (all users in it). Root user in MEMBER accounts IS affected by SCPs.

Example: Dev Account (member) has SCP denying S3. Root user in Dev Account CANNOT create S3 buckets.

### Correction #2: Cross-Account Trust is ONE-WAY
**Misconception:** Both accounts must trust each other for cross-account access  
**Reality:** Target account trusts source account (one direction only). Source user needs sts:AssumeRole permission in their IAM policy.

Cross-account roles are NOT bidirectional. Account B trusts Account A, Account A does NOT need to trust Account B.

### Correction #3: Trust Policy + IAM Permission (BOTH Required)
**Misconception:** If Trust Policy allows my account, I can assume the role  
**Reality:** Need BOTH trust policy (target account) AND sts:AssumeRole permission (source account IAM policy).

User with ZERO IAM permissions CANNOT assume any roles, even if trust policy allows their account.

### Correction #4: Control Tower for Existing Organizations
**Misconception:** Use Control Tower to scale existing 50-account organization  
**Reality:** Control Tower is for greenfield (new) deployments. Existing complex orgs should use manual Organizations.

Enabling Control Tower on existing org requires migrating all accounts to Control Tower's prescribed OU structure (disruptive, time-consuming).

### Correction #5: Deny-List vs Allow-List SCPs
**Misconception:** Use allow-list SCP permitting only specific instance types  
**Reality:** Use deny-list SCP blocking ONLY prohibited resources (deny what's bad, allow everything else).

Example: Deny p3/p4 GPU instances, allow all others. Don't restrict to only t3/m5/c5 (too limiting).

---

## Real-World Enterprise Patterns

### Pattern #1: Security-First Multi-Account
```
Organization
├── Security OU (centralized logging, GuardDuty, SecurityHub)
├── Production OU (strict SCPs: deny non-US regions, deny database deletion, require MFA)
├── Non-Production OU (relaxed SCPs, dev and test environments)
└── Workloads OU (data science, analytics, ML)
```

### Pattern #2: Dev/Test/Prod Isolation
- Dev accounts: Full admin, auto-shutdown at night, budget limits, deny expensive instances
- Staging accounts: Read-only for developers, full access for QA
- Prod accounts: ZERO developer access, only DevOps + automated CI/CD

### Pattern #3: Compliance-Driven (PCI-DSS/GDPR)
- PCI-DSS OU: us-east-1 ONLY, maximum encryption, isolated from other accounts
- GDPR-EU OU: EU regions ONLY, deny copying data to non-EU regions
- US-Customer-Data OU: US regions ONLY

### Pattern #4: Hub-and-Spoke Networking
- Hub account: Transit Gateway, VPN to on-prem, NAT Gateway for internet
- Spoke accounts: Attach to Transit Gateway, route through hub
- Cost savings: 92% reduction ($27,740/month → $2,153/month for 20 accounts)

### Pattern #5: Cost Allocation & Chargeback
- Account boundary = cost boundary
- Marketing Account bill: $5,500/month → Invoice Marketing team
- Engineering Account bill: $55,000/month → Invoice Engineering team
- ZERO tagging needed, automatic breakdown

---

## SA Pro Exam Relevance

**Domain 1: Design Solutions for Organizational Complexity (26% of exam)**

Today's content directly maps to:
- Multi-account strategy design
- Service Control Policy implementation
- Cross-account resource access patterns
- Organizational structure best practices

**Common exam question patterns:**
- "Customer has 50 accounts, needs to prevent X. Which SCP?"
- "Customer needs cost allocation by team. Which approach?"
- "Customer has compliance requirement Y. Which account structure?"
- "Which accounts can assume this cross-account role?" (trust policy + IAM)

**This was THE most important SA Pro topic. You're now 80% ahead for Phase 2.**

---

## How This Connects to TAM L4

TAM L4 interviews test multi-account architecture design:

**Interview question:** "Customer is scaling from 1 account to 20 accounts. Design their organization structure."

**Answer (using today's knowledge):**
"I'd recommend AWS Organizations with this structure:
- Security OU for Log Archive and Audit accounts (centralized security)
- Production OU with strict SCPs (deny non-approved regions, protect databases)
- Non-Production OU with relaxed SCPs (dev/test flexibility)
- Consolidated Billing for automatic cost allocation
- Cross-account IAM roles for security team audit access
- Hub-and-spoke networking via Transit Gateway (cost optimization)

This provides blast radius containment, compliance boundaries, cost visibility, and scalability."

**That's senior-level thinking. You can now design AND justify multi-account architectures.**

---

## Next Steps

**Phase 1 Remaining:** 11 days (Days 32-42)

**Tomorrow (December 17 - later today):** Days 32-33
- Advanced networking (Direct Connect, Transit Gateway deep dive)
- Database migration strategies
- Disaster recovery patterns

**Phase 2 Start:** January 1, 2026 - SA Professional intensive prep

---

## Reflections

AWS Organizations is the foundation of enterprise AWS architecture. Single accounts don't scale - the blast radius, compliance, and cost allocation problems are insurmountable at scale.

**Key insight:** Account boundaries are HARD boundaries. IAM policies can be changed, SCPs can be modified (in member accounts), but you cannot IAM your way into a different AWS account. This is what auditors mean by "hard boundary" - physical/logical separation at the AWS infrastructure level.

**Biggest aha moment:** Management Account SCP immunity. I initially thought root user bypassed SCPs everywhere, but the reality is more nuanced - it's the MANAGEMENT ACCOUNT that's immune (all users in it), not the root user concept itself. Root in member accounts is restricted by SCPs just like any other principal.

**Cross-account IAM is elegant:** Instead of managing credentials in 20 accounts, assume roles from one central account with temporary credentials. Clean, secure, auditable.

**Control Tower is powerful but limited:** Amazing for greenfield deployments (production-ready in 1 hour), but attempting to retrofit it onto existing complex organizations is a mistake. Know when to use which tool.

Understanding when NOT to use a service is as important as understanding when to use it.
