# Tutorial 4: Create Your First EC2 Instance + Connect Using SSH + AWS Security Groups

## What is EC2?

EC2 (Elastic Compute Cloud) = Virtual server in the cloud. Like renting a VPS, but you pay only for what you use.

## What are Security Groups?

Firewall rules that control traffic to your EC2. Think: which ports are open, who can connect.

---

## Step-by-Step: Launch EC2 Instance

### 1. Open EC2 Dashboard

- AWS Console → Search "EC2"
- Click **Launch Instance** button

### 2. Configure Instance

**Name:** `my-learning-server`

**Application and OS Images (AMI):**

- Select: Ubuntu Server 22.04 LTS (Free tier eligible)
- Architecture: 64-bit (x86)

**Instance Type:**

- Select: `t3.micro` (Free tier: 750 hrs/month)
- 2 vCPU, 1 GB RAM

**Key Pair (Login):**

- Click **Create new key pair**
- Key pair name: `my-aws-key`
- Key pair type: RSA
- Private key format: `.pem`
- Click **Create key pair**
- File downloads automatically → **SAVE IT SAFELY**

**Network Settings (Security Group):**

- Click **Edit**
- Security group name: `learning-sg`
- Description: Security group for learning

**Add these rules:**

| Type  | Port | Source Type | Source | Why                 |
|-------|------|-------------|--------|---------------------|
| SSH   | 22   | My IP       | (auto-filled) | Connect via terminal |
| HTTP  | 80   | Anywhere    | 0.0.0.0/0 | Web traffic         |
| HTTPS | 443  | Anywhere    | 0.0.0.0/0 | Secure web traffic  |

⚠️ **CRITICAL:** SSH must be **My IP**, NOT Anywhere!

- Click **Add security group rule** for HTTP and HTTPS.

**Storage:**

- Keep default: 8 GB `gp3` (Free tier: up to 30 GB)

### 3. Launch

- Review everything
- Click **Launch instance**
- Wait 1-2 minutes for status: `Running`

---

## Connect to EC2 Using SSH

### Get Instance Details

- Go to EC2 Dashboard → Instances
- Select your instance
- Copy **Public IPv4 address**

### Connect from Terminal

**Linux/Mac/Windows (PowerShell/Git Bash):**

```bash
# Move key to .ssh directory (first time only)

mkdir -p ~/.ssh
mv ~/Downloads/my-aws-key.pem ~/.ssh/
chmod 400 ~/.ssh/my-aws-key.pem
```

**Connect (replace with YOUR public IP):**

```bash
ssh -i ~/.ssh/my-aws-key.pem ubuntu@YOUR_PUBLIC_IP
```

First time: Type `yes` when asked "Are you sure..."

Success looks like: `ubuntu@ip-172-31-x-x:~$`

### Test Your Server

```bash
# Check system info
lsb_release -a

# Update packages
sudo apt update

# Check free memory
free -h

# Check disk space
df -h

# Exit when done
exit
```

---

## Understanding Security Groups

Security Group = Firewall for your EC2

**Current setup:**

- Port 22 (SSH): Only YOUR IP can connect ✅
- Port 80 (HTTP): Anyone can access ✅
- Port 443 (HTTPS): Anyone can access ✅

**To modify later:**

- EC2 Dashboard → Security Groups
- Select `learning-sg`
- Inbound rules tab → Edit inbound rules
- Add/Remove/Modify ports

**Common ports you'll need later:**

- `3000`: Node.js apps
- `8000`: Laravel development
- `3306`: MySQL (⚠️ **NEVER** open to `0.0.0.0/0`!)
- `6379`: Redis (⚠️ **NEVER** open to `0.0.0.0/0`!)

---

## Cost Tracking

**Free Tier (`t3.micro`):**

- 750 hours/month = $0 (for 12 months)
- 8 GB storage = $0 (within 30 GB free tier)
- After free tier: ~$0.0104/hour = ~$7.50/month if running 24/7

**Current cost:** $0 (within free tier)

---

## 🗑️ STOP & DELETE INSTRUCTIONS

### Stop Instance (Saves money, keeps data)

**In Terminal (if connected):**

```bash
exit
```

**In AWS Console:**

- EC2 → Instances
- Select instance → Instance state → Stop instance
- Confirm

**Stopped instances:**

- CPU/RAM charges: $0/hour ✅
- Storage: $0 (within free tier) ✅
- ⚠️ Public IP will change when restarted

### Start Instance Again:

- Select instance → Instance state → Start instance
- Wait for new Public IP
- Connect with new IP

### Delete Completely (When done with ALL tutorials):

- Select instance → Instance state → Terminate instance
- Type "terminate" to confirm
- Go to Security Groups → Select `learning-sg` → Actions → Delete security groups

**Important Notes:**

- **Stop** = Pause (resume anytime, IP changes, FREE)
- **Terminate** = Delete forever (can't recover)
- **Always STOP** when not using for learning

---

## Quick Reference Commands

```bash
# Connect to server
ssh -i ~/.ssh/my-aws-key.pem ubuntu@YOUR_PUBLIC_IP

# Exit server
exit

# Update system
sudo apt update && sudo apt upgrade -y

# Check running processes
top
# Press 'q' to quit

# Check who's logged in
who
```

---

## Troubleshooting

### Permission denied (publickey):

- Check key file permissions: `chmod 400 ~/.ssh/my-aws-key.pem`
- Verify correct IP address
- Ensure using `ubuntu@` username

### Connection timeout:

- Check Security Group has SSH port 22 open to **My IP**
- Verify instance is **Running** (not Stopped)
- Check your internet connection

### Key file not found:

- Verify path: `ls ~/.ssh/my-aws-key.pem`
- Use full path in ssh command
