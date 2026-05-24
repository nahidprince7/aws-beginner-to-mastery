Tutorial 2: AWS Billing Dashboard + Create Billing Alarm + Avoid Unexpected Charges
What You'll Learn

Navigate AWS Billing Dashboard
Understand your current charges
Set up billing alarms to avoid surprises
Monitor Free Tier usage
Best practices to minimize costs


Part 1: Access Billing Dashboard
Step 1: Login to AWS Console
https://console.aws.amazon.com
Step 2: Open Billing Dashboard
Method 1:

Top-right corner → Click your account name
Click Billing and Cost Management

Method 2:

Search bar → Type "Billing"
Click Billing and Cost Management


Part 2: Understanding the Billing Dashboard
Dashboard Overview
You'll see:
1. Month-to-Date Costs

Current month spending
Forecast for end of month

2. Last Month Costs

Previous month total

3. Free Tier Usage

Shows what you're using vs. limits

4. Cost by Service

Breakdown by EC2, S3, RDS, etc.


Part 3: Check Free Tier Usage
Step 1: Open Free Tier Page
Left sidebar → Click Free Tier
What You'll See:
Services with Active Usage:

EC2: Shows hours used / 750 hours free
S3: Shows storage used / 5 GB free
RDS: Shows hours used / 750 hours free

Traffic Light System:

🟢 Green: Safe (under 85%)
🟡 Yellow: Warning (85-100%)
🔴 Red: Over limit (you're being charged!)

Important Metrics to Watch:
EC2:
t3.micro running hours: 240 / 750 hours
Status: 🟢 Safe
S3:
Storage: 2.5 GB / 5 GB
GET Requests: 5,000 / 20,000
PUT Requests: 1,000 / 2,000
Status: 🟢 Safe
Data Transfer:
Data Transfer Out: 25 GB / 100 GB
Status: 🟢 Safe

Part 4: View Current Month Charges
Step 1: Open Bills Page
Left sidebar → Bills
What You'll See:
Charges by Service:
Service               | Charges
---------------------|----------
EC2                  | $0.00 (Free Tier)
S3                   | $0.00 (Free Tier)
Data Transfer        | $0.00 (Free Tier)
Tax                  | $0.00
---------------------|----------
Total                | $0.00
If you see charges:

Click the service name
See detailed breakdown
Check which resources are costing money


Part 5: Create Billing Alarm (Critical!)
Why Create Billing Alarm?
Get email alert BEFORE unexpected charges accumulate
Step 1: Enable Billing Alerts (One Time Setup)

Billing Dashboard → Right side → Billing preferences
Scroll to Alert preferences
✅ Check Receive Billing Alerts
Enter your email address
Click Save preferences

Step 2: Open CloudWatch (For Alarms)
Switch Region:

Top-right → Select US East (N. Virginia) ⚠️ IMPORTANT!
Billing metrics only available in us-east-1

Open CloudWatch:

Search bar → Type "CloudWatch"
Click CloudWatch

Step 3: Create Alarm

Left sidebar → Alarms → All alarms
Click Create alarm
Click Select metric

Step 4: Select Billing Metric

Click Billing (if not visible, ensure you're in us-east-1)
Click Total Estimated Charge
Select the checkbox for USD
Click Select metric

Step 5: Configure Alarm
Metric:

Already selected ✅

Conditions:

Threshold type: Static
Whenever EstimatedCharges is: Greater
than: 5 (you'll be alerted if bill exceeds $5)

For learning, use $5. Adjust based on comfort level.
Click Next
Step 6: Configure Notifications
Send notification to:

Create new topic
Topic name: billing-alerts
Email endpoint: your-email@example.com

Click Create topic
⚠️ Check your email and CONFIRM the subscription!
Click Next
Step 7: Name the Alarm
Alarm name: billing-alert-5-dollars
Description: Alert when monthly bill exceeds $5
Click Next → Create alarm

Part 6: Create Multiple Billing Alarms (Recommended)
Create 3 alarms at different thresholds:
Alarm 1: Early Warning

Threshold: $1
Purpose: "Hey, you're being charged something"

Alarm 2: Moderate Alert

Threshold: $5
Purpose: "Usage is increasing"

Alarm 3: Critical Alert

Threshold: $10
Purpose: "Stop everything and investigate!"

Repeat Part 5 steps for each threshold

Part 7: Set Up Budget (Alternative Method)
Step 1: Open AWS Budgets
Billing Dashboard → Left sidebar → Budgets
Step 2: Create Budget
Click Create budget
Select budget type:

✅ Cost budget - Recommended

Click Next
Step 3: Set Budget Details
Budget name: monthly-budget-10
Period: Monthly
Budget effective dates: Recurring budget
Start month: Current month
Budgeting method: Fixed
Budgeted amount: $10.00
Click Next
Step 4: Configure Alerts
Alert 1: 80% Threshold

Threshold: 80% of budgeted amount ($8)
Email: your-email@example.com

Click Add an alert threshold
Alert 2: 100% Threshold

Threshold: 100% of budgeted amount ($10)
Email: your-email@example.com

Click Next → Create budget

Part 8: Understanding Common Charges
Free Tier Services (First 12 Months):
✅ FREE:

t2.micro / t3.micro EC2 (750 hrs/month)
5 GB S3 storage
30 GB EBS storage
750 hrs RDS db.t2.micro
100 GB data transfer out

❌ NOT FREE:

Elastic IP (when instance is stopped): $0.005/hr
Snapshots (after 1st GB): $0.05/GB/month
Load Balancers: ~$18/month
NAT Gateway: $0.045/hr
Larger instance types (t2.small, t2.medium, etc.)


Part 9: Cost-Saving Best Practices
DO These:
✅ Stop instances when not using
bash# Stop = $0/hr for compute
# Keep paying only for storage (~$0.80/month for 8GB)
✅ Delete unused resources

Old snapshots
Unattached EBS volumes
Unused Elastic IPs
Old AMIs

✅ Use t3.micro/t2.micro only (free tier)
✅ Monitor Free Tier usage weekly
✅ Set multiple billing alarms ($1, $5, $10)
✅ Delete test resources immediately
✅ Use AWS Calculator before trying new services

https://calculator.aws/

DON'T Do These:
❌ Leave instances running 24/7 when learning

Exception: If within 750 hrs free tier

❌ Create multiple large instances

2+ t3.micro = Over 750 hrs limit

❌ Forget to delete old snapshots
❌ Use RDS for learning

Install MySQL on EC2 instead (free)

❌ Use NAT Gateway

$32/month minimum

❌ Create multiple Elastic IPs

Release when not using

❌ Enable detailed CloudWatch monitoring

Use basic (free) monitoring


Part 10: Monthly Cost Monitoring Routine
Every Week, Check:
1. Free Tier Usage
   Billing → Free Tier
   Check all services are 🟢 Green
2. Month-to-Date Cost
   Billing Dashboard
   Ensure $0.00 or within budget
3. Running Instances
   EC2 → Instances
   Stop anything not actively using
4. Check Email
   Look for billing alert emails
   Investigate any alerts immediately

Part 11: What To Do If You Get a Charge
Step 1: Identify the Service

Billing → Bills
Click current month
See which service has charges

Step 2: Identify the Resource
Example: EC2 Charges

Click EC2 in bill
See breakdown by:

Instance type
Region
Operation



Step 3: Find & Stop the Resource
EC2:
EC2 → Instances
Find expensive instance
Stop or Terminate
Elastic IP:
EC2 → Elastic IPs
Release unassociated IPs
Snapshots:
EC2 → Snapshots
Delete old snapshots
EBS Volumes:
EC2 → Volumes
Delete unattached volumes
Step 4: Contact AWS Support (If Needed)
For unexpected charges under $100:

AWS Console → Support Center
Create case → Account and billing
Explain situation
AWS often provides one-time courtesy credit


Part 12: Understanding Your Bill
Sample Bill Breakdown:
Service: Amazon Elastic Compute Cloud
Region: US East (N. Virginia)
------------------------------------------
LineItem/UsageType              | Cost
------------------------------------------
BoxUsage:t3.micro               | $0.00 (Free Tier)
EBS:VolumeUsage                 | $0.00 (Free Tier)
DataTransfer-Out-Bytes          | $0.00 (Free Tier)
ElasticIP:IdleAddress           | $3.60 (CHARGED!)
------------------------------------------
Total                           | $3.60
This shows:

t3.micro = Free ✅
EBS storage = Free ✅
Data transfer = Free ✅
Idle Elastic IP = CHARGED ❌

Fix: Release Elastic IP when instance stopped

Cost Calculator Examples
Scenario 1: Learning (Minimal Cost)
Resources:

1x t3.micro running 24/7
8 GB EBS storage
5 GB S3 storage
50 GB data transfer/month

Cost:

Within free tier = $0/month ✅


Scenario 2: Real Project (After Free Tier)
Resources:

1x t3.micro running 24/7
20 GB EBS storage
100 GB S3 storage
500 GB data transfer/month

Cost:

EC2: $7.50/month
EBS: $2.00/month
S3: $2.30/month
Data Transfer: $36/month (over 100 GB free)

Total: ~$47.80/month

Common Questions
Q: I stopped my instance, why am I still charged?
A: EBS storage ($0.80/month) and Elastic IP ($3.60/month) continue charging when instance is stopped.
Q: Can I pause instead of terminate?
A: Stop = Pause (keeps data, charges for storage). Terminate = Delete (no charges, lose data).
Q: How to get $0 bill?
A: Stay within free tier limits, release Elastic IP when stopped, delete unused resources.
Q: What if I exceed free tier accidentally?
A: Contact AWS support. They often provide one-time courtesy credit for learning purposes.

Billing Dashboard Checklist
✅ Billing alerts enabled
✅ Email confirmed for alerts
✅ CloudWatch alarm created ($1 threshold)
✅ CloudWatch alarm created ($5 threshold)
✅ CloudWatch alarm created ($10 threshold)
✅ AWS Budget created ($10/month)
✅ Free Tier page bookmarked
✅ Current charges = $0.00
✅ Weekly monitoring routine set

Cost: $0 (setting up billing alerts is free!)

Quick Reference: Where to Check Costs
Daily Check:
→ Billing Dashboard → Month-to-Date Total

Weekly Check:
→ Billing → Free Tier (check all services)
→ EC2 → Instances (stop unused)

Monthly Check:
→ Billing → Bills (detailed breakdown)
→ EC2 → Snapshots (delete old)
→ EC2 → Volumes (delete unattached)
→ S3 → Buckets (check storage usage)

Emergency: Stop All Charges NOW
If you see unexpected high bill:
bash# 1. Stop all EC2 instances
EC2 → Instances → Select All → Stop

# 2. Release all Elastic IPs
EC2 → Elastic IPs → Select All → Release

# 3. Delete all S3 buckets (after backing up)
S3 → Empty bucket → Delete bucket

# 4. Delete all RDS databases
RDS → Databases → Delete (no snapshot)

# 5. Delete all Load Balancers
EC2 → Load Balancers → Delete

# 6. Delete NAT Gateways (if any)
VPC → NAT Gateways → Delete

✅ You now understand:

How to access Billing Dashboard
How to read your AWS bill
How to track Free Tier usage
How to set up billing alarms (multiple thresholds)
How to create budgets
Best practices to avoid charges
What to do if you get unexpected charges

Next: Continue with your learning topics!
Remember: Check billing dashboard every week! 📊