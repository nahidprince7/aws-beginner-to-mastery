## Tutorial 1: What is AWS? + AWS Free Tier Explained

## What is AWS?

AWS (Amazon Web Services) = Cloud computing platform by Amazon.
Instead of buying physical servers, you rent computing resources from Amazon's data centers worldwide.

## What Can You Do with AWS?

- Host websites & apps (like your own VPS)
- Store files (images, videos, backups)
- Run databases (MySQL, PostgreSQL, MongoDB)
- Send emails (SES - Simple Email Service)
- Machine Learning (AI models, image recognition)
- Serverless functions (code that runs without servers)
- CDN (deliver content fast worldwide)

## AWS vs Traditional Hosting

| Traditional Hosting | AWS |
|-------------------|-----|
| Pay monthly (fixed) | Pay per hour/GB used |
| Limited scalability | Scale instantly |
| One location | Global data centers |
| Manual setup | Automate everything |
| Expensive to start | Free tier + pay as you grow |

## Key AWS Services (You'll Learn)

### Compute
- EC2 - Virtual servers (like VPS)
- Lambda - Run code without servers

### Storage
- S3 - Store files (images, videos, backups)
- EBS - Hard drives for EC2 instances

### Database
- RDS - Managed MySQL/PostgreSQL
- DynamoDB - NoSQL database

### Networking
- Route 53 - Domain & DNS management
- CloudFront - CDN (content delivery)
- VPC - Private network in cloud

### Security
- IAM - User & permission management
- Security Groups - Firewalls

## AWS Free Tier Explained

### What is Free Tier?
12 months FREE from signup date + Always Free services.

### 12 Months Free (From Signup)

| Service | Free Limit | Actual Value |
|---------|------------|--------------|
| EC2 | 750 hrs/month (t2.micro or t3.micro) | ~$8.50/month |
| S3 | 5 GB storage + 20,000 GET requests | ~$0.50/month |
| RDS | 750 hrs/month (db.t2.micro, db.t3.micro) | ~$15/month |
| EBS | 30 GB SSD storage | ~$3/month |
| Data Transfer | 15 GB outbound/month | ~$1.35/month |

**Total Value:** ~$28.85/month FREE for 12 months

### Always Free (Forever):

| Service | Free Limit |
|---------|------------|
| Lambda | 1 million requests/month |
| DynamoDB | 25 GB storage |
| CloudWatch | 10 custom metrics |
| SNS | 1,000 email notifications |

## How Free Tier Works

### Example: EC2 Free Tier
750 hours/month = How much?

- 1 instance running 24/7 = 720-744 hrs/month ✅ (FREE)
- 2 instances running 24/7 = 1440-1488 hrs/month ❌ (750 FREE, rest charged)
- 1 instance running 12 hrs/day = ~360 hrs/month ✅ (FREE, 390 hrs unused)

**Golden Rule:**
1 t3.micro instance running 24/7 for 12 months = $0

## What Happens After 12 Months?
You start paying for:

- EC2: ~$8.50/month (if running 24/7)
- RDS: ~$15/month (if running 24/7)
- S3: Still cheap (~$0.023/GB/month)

Always Free services continue forever.

## How to Check Your Free Tier Usage

### Step 1: Login to AWS Console
https://console.aws.amazon.com

### Step 2: Open Billing Dashboard
Top-right: Click your account name
Click Billing and Cost Management

### Step 3: View Free Tier Usage
Left sidebar: Free Tier
See usage for each service
Alerts when approaching limits

## Free Tier Best Practices

✅ Use only t2.micro or t3.micro (other types not free)
✅ Stop instances when not using (saves free tier hours)
✅ Monitor usage weekly (avoid surprises)
✅ Set billing alarms (get alerts before charges)
✅ Delete unused resources (old snapshots, volumes)
✅ Stay within 15 GB data transfer/month

❌ Don't run multiple instances 24/7 (exceeds 750 hrs)
❌ Don't ignore billing dashboard (blind usage = charges)
❌ Don't use large instance types (expensive!)

## AWS Regions
AWS has data centers worldwide (Regions).

### Popular Regions
- us-east-1 (N. Virginia) - Cheapest, most services
- us-west-2 (Oregon) - Popular, stable
- eu-west-1 (Ireland) - Europe
- ap-south-1 (Mumbai) - India/Bangladesh
- ap-southeast-1 (Singapore) - Southeast Asia

### Choose Region Based On:
- Proximity to users (lower latency)
- Service availability (new features launch in us-east-1 first)
- Cost (some regions cheaper)

For learning: Use us-east-1 (most services, cheapest)

## Common AWS Terms

| Term | Meaning |
|------|---------|
| Instance | Virtual server (EC2) |
| AMI | Operating system image |
| Snapshot | Backup of disk |
| Volume | Hard drive attached to EC2 |
| Bucket | Storage container in S3 |
| IAM | User & permission system |
| VPC | Private network |
| Region | Geographic location of data centers |
| AZ (Availability Zone) | Isolated data center in a region |

Cost: $0 (reading is free!)

## Key Takeaways
✅ AWS = Rent computing instead of buying servers
✅ Free Tier = 12 months free + always free services
✅ 1 t3.micro running 24/7 = FREE for 12 months
✅ 750 hours/month = enough for 1 instance running all day
✅ Monitor usage to avoid unexpected charges
✅ After 12 months, you pay ~$8-$15/month for same setup
