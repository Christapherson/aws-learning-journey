# Day 21: Serverless File Processor with S3 + Lambda

**Date:** November 5, 2025 (Tuesday)  
**Phase 1 Progress:** Day 21 of 42 (50% complete)  
**Time Investment:** 3 hours  
**Energy Level:** 10/10 → 8/10 (solid execution despite technical challenges)

---

## 🎯 Today's Goal

Build Project #3: **Automated File Processor** using event-driven serverless architecture (S3 + Lambda + CloudWatch)

---

## 📦 What I Built

### Project #3: Serverless File Processor

**Live System Components:**
- **S3 Bucket:** `chris-image-processor-2025`
- **Lambda Function:** `image-processor` (Python 3.12)
- **CloudWatch Logs:** Full execution audit trail
- **IAM Role:** `image-processor-lambda-role`

**GitHub:** [day-21-image-processor/](https://github.com/Christapherson/aws-learning-journey/tree/main/day-21-image-processor)

---

## 🏗️ Architecture Diagram
```
┌─────────────┐
│   User      │
└──────┬──────┘
       │ 1. Upload image
       ▼
┌─────────────────────────────────────┐
│  S3 Bucket: chris-image-processor   │
│                                     │
│  ┌──────────────┐                  │
│  │ originals/   │                  │
│  │ - test.jpg   │                  │
│  └──────┬───────┘                  │
│         │                           │
│         │ 2. S3 Event Notification │
│         ▼                           │
│  ┌──────────────┐                  │
│  │ Event Trigger│                  │
│  └──────┬───────┘                  │
└─────────┼───────────────────────────┘
          │
          │ 3. Invoke Lambda
          ▼
┌─────────────────────────────┐
│  Lambda: image-processor    │
│                             │
│  - Validate file type       │
│  - Log file details         │
│  - Copy to processed/       │
└──────────┬──────────────────┘
           │
           │ 4. Copy operation
           ▼
┌─────────────────────────────────────┐
│  S3 Bucket: chris-image-processor   │
│                                     │
│  ┌──────────────┐                  │
│  │ processed/   │                  │
│  │ - test.jpg   │ ✅ File ready    │
│  └──────────────┘                  │
└─────────────────────────────────────┘
           │
           │ 5. Logs sent
           ▼
┌─────────────────────────┐
│  CloudWatch Logs        │
│                         │
│  - Execution details    │
│  - File sizes           │
│  - Success/error status │
└─────────────────────────┘
```

---

## 🔧 Technical Implementation

### AWS Services Used

1. **Amazon S3**
   - Storage for original and processed files
   - Event notifications trigger Lambda
   - Folder structure: `originals/` and `processed/`

2. **AWS Lambda**
   - Runtime: Python 3.12
   - Memory: 128 MB
   - Timeout: 30 seconds
   - Deployment: ZIP file (2.5 KB)

3. **CloudWatch**
   - Execution logs with timestamps
   - File size tracking
   - Success/error monitoring

4. **IAM**
   - Role: `image-processor-lambda-role`
   - Policies: `AWSLambdaBasicExecutionRole` + `AmazonS3FullAccess`

### Key Features

**Event-Driven Processing:**
- S3 event notification triggers Lambda automatically
- Processing happens within 2-3 seconds of upload
- No polling or scheduled jobs required

**File Validation:**
- Only processes image files (.jpg, .jpeg, .png, .gif)
- Non-image files gracefully rejected (no crashes)
- Returns HTTP 200 with rejection message

**Infinite Loop Prevention:**
- Trigger watches `originals/` prefix only
- Lambda ignores uploads to `processed/`
- Prevents recursive invocations

**Comprehensive Logging:**
- File name and size logged to CloudWatch
- Processing timestamps recorded
- Success/failure status tracked

---

## 💻 Code Walkthrough

### Lambda Function (lambda_function.py)

**Core Logic:**
```python
def lambda_handler(event, context):
    # 1. Extract S3 event details
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # 2. Validate folder location (originals/ only)
    if not key.startswith('originals/'):
        return rejection_response
    
    # 3. Validate file extension (images only)
    valid_extensions = ['.jpg', '.jpeg', '.png', '.gif']
    if file_extension not in valid_extensions:
        return rejection_response
    
    # 4. Copy file to processed/ folder
    copy_source = {'Bucket': bucket, 'Key': key}
    s3_client.copy_object(
        CopySource=copy_source,
        Bucket=bucket,
        Key=f"processed/{filename}"
    )
    
    # 5. Return success with metadata
    return success_response
```

**Error Handling:**
- Try-catch wraps entire function
- Logs errors to CloudWatch
- Returns HTTP 500 with error details

---

## 🧪 Testing & Validation

### Test Scenarios Executed

**✅ Test 1: JPEG Upload (Happy Path)**
- Uploaded `test-image.jpg` (2.9 MB) to `originals/`
- Lambda triggered automatically
- File copied to `processed/` within 3 seconds
- CloudWatch logs confirmed success

**✅ Test 2: PNG Support**
- Uploaded `test-image.png` to `originals/`
- Successfully processed and copied
- Both JPEG and PNG formats working

**✅ Test 3: Invalid File Type (Error Handling)**
- Uploaded `test-file.pdf` to `originals/`
- Lambda rejected gracefully (HTTP 200)
- No file appeared in `processed/` (correct behavior)
- CloudWatch logs: "Skipping non-image file"

### CloudWatch Log Output
```
2025-11-12T04:21:52.531Z - INIT_START Runtime Version: python:3.12.v35
2025-11-12T04:21:52.656Z - START RequestId: 63d5c332-7d96-42bd-ac7a-5f2a88a2f
2025-11-12T04:21:43.070Z - Processing file: originals/test-image.jpg from bucket: chris-image-processor-2025
2025-11-12T04:21:43.355Z - File size: 3024671 bytes (2.88 MB)
2025-11-12T04:21:43.356Z - Copying originals/test-image.jpg to processed/test-image.jpg...
2025-11-12T04:21:43.493Z - Successfully processed originals/test-image.jpg -> processed/test-image.jpg
2025-11-12T04:21:53.515Z - END RequestId: 63d5c332-7d96-42bd-ac7a-5f2a88a2f
2025-11-12T04:21:53.515Z - REPORT RequestId: 63d5c332-7d96-42bd-ac7a-5f2a88a2f
Duration: 324.73 ms   Billed Duration: 325 ms   Memory Size: 128 MB   Max Memory Used: 68 MB
```

**Key Metrics:**
- Execution time: 324 ms (well under 30 second timeout)
- Memory usage: 68 MB / 128 MB allocated (53% utilization)
- Cold start included (INIT_START on first invocation)

---

## 🚧 Challenges & Solutions

### Challenge 1: Mac vs Linux Binary Incompatibility

**Problem:**
- Initial plan included image resizing with Pillow library
- Installed Pillow on Mac: `pip3 install --target ./package pillow`
- Lambda deployment failed with `ImportModuleError`
- Root cause: Mac-compiled Pillow binaries don't work on Lambda's Linux environment

**Attempted Solutions:**
1. Tried AWS-provided Lambda layers (no Pillow layer exists)
2. Attempted public Klayers Pillow layer (permission errors)
3. Different Pillow layer versions (resource not found errors)

**Final Solution:**
- Simplified project to focus on **event-driven architecture** instead of image manipulation
- Removed Pillow dependency entirely
- Lambda now copies files without modification (still demonstrates S3 triggers, serverless patterns, validation logic)
- **Learning:** Lambda layers must be compiled for target OS. For production: use Docker-based builds or AWS-managed layers

### Challenge 2: Python Runtime Version Confusion

**Problem:**
- Lambda function initially showed "Python 3.14" (doesn't exist yet)
- Couldn't find runtime setting in Configuration UI
- Layer compatibility issues

**Solution:**
- Found Edit Runtime Settings via Code tab → Runtime settings → Edit
- Changed from Python 3.14 → Python 3.12
- Verified compatibility before attempting layer additions

**Learning:** Always verify Lambda runtime version matches available libraries and AWS support

### Challenge 3: Time Management Decision

**Context:** 
- 3:14 PM when debugging layer issues
- Could spend 30+ minutes learning Docker-based Lambda layers
- Or simplify and deliver working portfolio project tonight

**Decision:** 
- Chose simplified approach (Option 3)
- Priorities: Complete Day 21 on schedule, demonstrate event-driven architecture, maintain portfolio momentum

**Outcome:** 
- Working project delivered in remaining time
- Core learning objectives achieved (S3 events, Lambda triggers, serverless patterns)
- Trade-off documented as "Future Enhancement" rather than failure

---

## 💰 Cost Analysis

### Estimated Production Costs

**Scenario:** Small business processing 10,000 images/month

**AWS Lambda:**
- Requests: 10,000/month
- Duration: ~325 ms average
- Memory: 128 MB
- Cost: **$0.002/month** (first 1M requests free, then $0.20 per million)

**Amazon S3:**
- Storage: 10 GB (5 GB originals + 5 GB processed)
- PUT requests: 10,000/month
- GET requests: 20,000/month (Lambda reads + user downloads)
- Cost: **~$0.50/month**

**CloudWatch Logs:**
- Log data ingestion: ~50 MB/month
- Storage: ~100 MB total
- Cost: **~$0.10/month**

**Total Monthly Cost: ~$0.62**

**Comparison to EC2 Alternative:**
- t3.micro instance (24/7): $7.50/month minimum
- Serverless savings: **92% cheaper** for low-volume use cases
- Automatic scaling included (no capacity planning needed)

---

## 🎓 Key Learnings

### Technical Concepts Mastered

1. **Event-Driven Architecture**
   - S3 event notifications trigger Lambda automatically
   - No polling overhead or scheduled jobs
   - Processing happens near real-time (2-3 seconds)

2. **Serverless Computing**
   - Pay-per-execution model (not always-on infrastructure)
   - Automatic scaling from 0 to thousands of concurrent executions
   - AWS manages all infrastructure (no server patching, no capacity planning)

3. **IAM Permissions Model**
   - Lambda needs execution role to access S3
   - Least privilege principle (production would use tighter S3 permissions)
   - Trust relationships allow Lambda to assume the role

4. **CloudWatch Logging**
   - All print() statements automatically captured
   - Log streams organized by execution
   - Essential for debugging serverless functions (no SSH access to servers)

5. **Prefix-Based Filtering**
   - S3 event notifications support prefix/suffix filters
   - Critical for preventing infinite loops
   - Enables multiple processors on same bucket (different folders)

### Solutions Architect Insights

**When to Use Serverless:**
- ✅ Event-driven workloads (file uploads, API requests, data changes)
- ✅ Unpredictable or sporadic traffic patterns
- ✅ Short-lived processing tasks (<15 minutes)
- ✅ Cost optimization for low-volume use cases
- ❌ Long-running batch jobs (use Batch or EC2)
- ❌ Stateful applications requiring persistent connections
- ❌ Workloads requiring specialized hardware (GPUs, etc.)

**Production Considerations:**
- Use environment variables for configuration (not hardcoded values)
- Implement dead letter queues for failed executions
- Set up CloudWatch alarms for error rates
- Use AWS X-Ray for distributed tracing
- Consider VPC integration if accessing private resources

---

## 📈 Portfolio Value

### What This Project Demonstrates

**To Hiring Managers:**
1. Understanding of **event-driven architecture** (not just request-response)
2. Ability to design **serverless solutions** (modern cloud pattern)
3. **Cost-conscious engineering** (comparing alternatives, analyzing trade-offs)
4. **Production thinking** (logging, error handling, validation)
5. **Problem-solving under constraints** (pivoted when layer issues arose)

**Interview Talking Points:**
- "I built an event-driven file processor using S3 and Lambda that processes uploads automatically within 2-3 seconds"
- "The system costs $0.62/month for 10,000 files - 92% cheaper than an always-on EC2 instance"
- "I implemented prefix-based filtering to prevent infinite loops when Lambda writes to the same bucket"
- "When I encountered binary compatibility issues with Lambda layers, I adapted the solution to maintain project momentum while documenting the limitation"

### Differentiation from Other Candidates

**Most candidates show:**
- Static websites (S3 + CloudFront) ✅ I have this (Project #1)
- REST APIs (API Gateway + Lambda) ✅ I have this (Project #2)

**Fewer candidates show:**
- Event-driven processing ✅ **I have this (Project #3)**
- S3 event notifications and triggers
- Serverless file workflows
- CloudWatch log analysis

---

## 🔮 Future Enhancements

### Phase 1 (Next 1-2 Days)
1. **Thumbnail Generation**
   - Use Docker to build Linux-compatible Pillow layer
   - Resize images to 800px width, maintain aspect ratio
   - Store thumbnails in `processed/thumbnails/` subfolder

2. **Metadata Extraction**
   - Extract EXIF data (camera, location, timestamp)
   - Store metadata in DynamoDB table
   - Enable searchable image catalog

### Phase 2 (Future Projects)
3. **SNS Notifications**
   - Send email when processing completes
   - Include file details and processed URL
   - Alert on errors for manual review

4. **Dead Letter Queue**
   - Route failed executions to SQS queue
   - Enable retry logic with exponential backoff
   - Separate persistent failures for investigation

5. **User-Facing Upload Interface**
   - Add API Gateway + pre-signed S3 URLs
   - Create web UI for direct browser uploads
   - Implement CloudFront distribution for global access

---

## 📚 Resources Used

**AWS Documentation:**
- [S3 Event Notifications](https://docs.aws.amazon.com/AmazonS3/latest/userguide/NotificationHowTo.html)
- [Lambda Python Handler](https://docs.aws.amazon.com/lambda/latest/dg/python-handler.html)
- [IAM Roles for Lambda](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)

**Learning References:**
- Adrian Cantrill SA Associate course (Lambda section)
- AWS Free Tier limits for cost optimization

---

## ✅ Day 21 Checklist

**Planning & Setup:**
- [✅] Created S3 bucket with `originals/` and `processed/` folders
- [✅] Set up IAM role with S3 permissions
- [✅] Configured Lambda function (Python 3.12, 128 MB, 30s timeout)

**Implementation:**
- [✅] Wrote Lambda function with validation logic
- [✅] Implemented error handling and logging
- [✅] Created deployment package (ZIP file)
- [✅] Uploaded code to Lambda

**Configuration:**
- [✅] Configured S3 event notification (originals/ prefix)
- [✅] Connected S3 trigger to Lambda function
- [✅] Verified permissions (IAM role attached correctly)

**Testing:**
- [✅] Tested JPEG upload (success)
- [✅] Tested PNG upload (success)
- [✅] Tested PDF upload (graceful rejection)
- [✅] Verified CloudWatch logs (execution details captured)

**Documentation:**
- [✅] Created architecture diagram
- [✅] Wrote case study with business value
- [✅] Documented challenges and solutions
- [✅] Added cost analysis
- [✅] Committed to GitHub

---

## 🎯 Tomorrow's Focus (Day 22)

**Options:**
1. **Add Pillow layer properly** (Docker-based build, learn production Lambda deployment)
2. **Start Project #4** (High-Availability Web App with EC2, ALB, RDS Multi-AZ)
3. **Polish existing projects** (add monitoring, improve documentation, create demo videos)

**Decision pending:** Will assess energy level and time available tomorrow

---

## 💭 Reflection

Today reinforced a critical lesson: **perfect is the enemy of done**. When I hit the Lambda layer compatibility wall at 3:14 PM, I had a choice: spend unknown time learning Docker-based Lambda builds, or simplify the project to ship something working tonight.

I chose to ship. The simplified version still demonstrates:
- Event-driven architecture ✅
- Serverless computing patterns ✅
- S3 event notifications ✅
- Lambda function development ✅
- CloudWatch logging ✅
- Error handling and validation ✅

In an interview, I can say: "I built this in 3 hours, encountered binary compatibility issues with Lambda layers, adapted the solution to maintain momentum, and documented the limitation as a future enhancement."

That story shows problem-solving, prioritization, and pragmatic engineering - more valuable than spending all night on image resizing.

**Three portfolio projects complete. Seven weeks until SA Pro certification study begins. On track.**

---

**GitHub:** [day-21-image-processor/](https://github.com/Christapherson/aws-learning-journey/tree/main/day-21-image-processor)  
**Portfolio Status:** 3/10 projects complete  
**Phase 1 Progress:** 21/42 days (50% complete)  
**Momentum:** Strong 🔥
