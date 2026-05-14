# Tutorial 3: Create IAM User and Permissions

## What is IAM?

IAM (Identity and Access Management) lets you create users and control what they can do in AWS. Instead of using your root account (which has full access), you create IAM users with limited permissions.

## Why Use IAM?

- **Security:** Root account = full power. If leaked, disaster.
- **Control:** Give developers only what they need
- **Tracking:** Know who did what

---

## Step-by-Step: Create IAM User

### 1. Login to AWS Console

- Go to https://console.aws.amazon.com
- Login with your root account

### 2. Open IAM Service

- Search "IAM" in top search bar
- Click IAM service

### 3. Create User

- Click **Users** (left sidebar)
- Click **Create user** button
- User name: `developer-user` (or any name)
- Click **Next**

### 4. Set Permissions

You have 3 options:

**Option A: Add to Group (Recommended)**

- Click **Create group**
- Group name: `Developers`
- Search and select: **AdministratorAccess** (for learning)
- Click **Create user group**

**Option B: Attach Policies Directly**

- Search: `AmazonEC2FullAccess`
- Check: `AmazonS3FullAccess`
- Check: `AmazonRDSFullAccess`

**Option C:** Copy from existing user (skip for now)

- Click **Next**

### 5. Review and Create

- Review details
- Click **Create user**

### 6. Setup Access (Important!)

- Click on the created user
- Go to **Security credentials** tab
- Scroll to **Access keys**
- Click **Create access key**
- Select: **Command Line Interface (CLI)**
- Check: "I understand..."
- Click **Next** → **Create access key**

⚠️ **SAVE THESE NOW:**

- **Access Key ID:** `AKIA...`
- **Secret Access Key:** `wJal...` (only shown once!)
- Click **Download .csv file** (backup)

---

**Cost:** $0 (IAM is FREE)

---

## 🗑️ HOW TO DELETE (When Done Learning)

### Delete IAM User:

- Go to IAM → Users
- Select the user (checkbox)
- Click **Delete** button
- Type the username to confirm
- Click **Delete**

### Delete Access Keys (if user still needed):

- Click on user
- **Security credentials** tab
- Find **Access keys** section
- Click **Actions** → **Deactivate** (or **Delete**)

---

## Best Practices

- ✅ Never share root account
- ✅ Enable MFA (Multi-Factor Authentication) on root
- ✅ Create separate users for each person
- ✅ Give minimum permissions needed
- ✅ Rotate access keys regularly
