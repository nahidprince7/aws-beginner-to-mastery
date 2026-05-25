# Tutorial 5: Create Elastic IP

## What is Elastic IP?

Elastic IP (EIP) = Static public IP that doesn't change when you stop/start your instance.

**Problem without EIP:**

- Stop instance → Start instance → New IP every time
- Domain pointing breaks
- SSH config breaks

**With EIP:**

- Same IP forever (until you release it)
- Domain stays connected
- Consistent SSH access

---

## Create Elastic IP

### Step 1: Allocate Elastic IP

- EC2 Dashboard → **Elastic IPs** (left sidebar under "Network & Security")
- Click **Allocate Elastic IP address**
- Network Border Group: Keep default
- Public IPv4 address pool: Amazon's pool of IPv4 addresses
- Click **Allocate**

You'll get a new IP like: `52.45.123.89`

### Step 2: Associate with Your Instance

- Select the new Elastic IP (checkbox)
- **Actions** → **Associate Elastic IP address**
- Resource type: Instance
- Instance: Select `my-learning-server`
- Private IP address: Keep default (auto-selected)
- Click **Associate**

### Step 3: Verify

- Go to **Instances**
- Select your instance
- Check **Public IPv4 address** = Your new Elastic IP

---

## Test Elastic IP

```bash
# SSH with NEW Elastic IP (replace with yours)
ssh -i ~/.ssh/my-aws-key.pem ubuntu@YOUR_ELASTIC_IP

# Test: Stop and Start instance from AWS Console
# IP will NOT change anymore!
```

---

## Cost Warning

| Status                                | Cost          |
|---------------------------------------|---------------|
| While associated with running instance | $0 ✅         |
| While associated with stopped instance | $0.005/hour (~$3.60/month) ❌ |
| Not associated with any instance       | $0.005/hour ❌ |

---

## 🗑️ DISASSOCIATE & RELEASE INSTRUCTIONS

### When Stopping Instance for Days:

**Disassociate** (keeps the IP reserved):

- EC2 → Elastic IPs
- Select your EIP → **Actions** → **Disassociate Elastic IP address**
- Confirm
- Cost: $0.005/hour until re-associated

**Release** (deletes IP forever):

- EC2 → Elastic IPs
- Select your EIP → **Actions** → **Release Elastic IP address**
- Type "Release" to confirm

**Important Notes:**

- Disassociate = Keep IP reserved (you still pay $0.005/hr)
- Release = Delete IP forever (charges stop)
- To avoid charges: Release EIP when completely done
- You can have 5 EIPs per region by default (soft limit)
