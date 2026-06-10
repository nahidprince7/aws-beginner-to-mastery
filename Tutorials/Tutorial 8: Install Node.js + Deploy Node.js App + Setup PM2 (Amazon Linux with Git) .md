# Tutorial 8: Install Node.js + Deploy Node.js App + Setup PM2 (Amazon Linux with Git)

## What You'll Learn

- Setup Git authentication (SSH or Token)
- Install Node.js and npm on Amazon Linux
- Deploy Node.js application from Git
- Install and configure PM2 (process manager)
- Setup PM2 to auto-restart on server reboot
- Configure Nginx as reverse proxy
- Setup SSL for Node.js app

## Prerequisites

- ✅ EC2 instance running (Amazon Linux)
- ✅ SSH access working
- ✅ Elastic IP assigned (optional)
- ✅ Domain/subdomain ready (optional but recommended for SSL)
- ✅ Git repository (public or private)

## Part 1: Update System & Install Git

### Update Amazon Linux
```bash
sudo dnf update -y
```

### Install Git
```bash
sudo dnf install git -y
```

### Verify Installation
```bash
git --version
```

## Part 2: Setup Git Authentication

### Option A: Public Repository (No Authentication)
If your repo is public, skip to Part 3

### Option B: Private Repository - Personal Access Token

#### Step 1: Generate Token on GitHub

Go to: https://github.com/settings/tokens
Click Generate new token → Tokens (classic)
Note: AWS Server Access
Select scopes: ✅ repo (full control)
Click Generate token
⚠️ COPY THE TOKEN NOW (you won't see it again!)

#### Step 2: Configure Git on Server
```bash
# Set up credential helper
git config --global credential.helper store

# Set your identity
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

When cloning, use:
```bash
git clone https://YOUR_TOKEN@github.com/username/repo.git
```

Or clone normally and enter token when prompted:
```bash
git clone https://github.com/username/repo.git
# Username: your_github_username
# Password: YOUR_TOKEN (paste the token, not your password)
```

### Option C: Private Repository - SSH Key (Recommended)

#### Step 1: Generate SSH Key on Server
```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your_email@example.com"

# Press Enter for all prompts (default location, no passphrase)
```

#### Step 2: Copy Public Key
```bash
# Display public key
cat ~/.ssh/id_ed25519.pub
```
Copy the entire output (starts with ssh-ed25519)

#### Step 3: Add to GitHub

Go to: https://github.com/settings/keys
Click New SSH key
Title: AWS EC2 Server
Key type: Authentication Key
Paste your public key
Click Add SSH key

#### Step 4: Test Connection
```bash
# Test SSH connection
ssh -T git@github.com
```
Should show: Hi username! You've successfully authenticated... ✅

#### Step 5: Clone Using SSH
```bash
git clone git@github.com:username/repo.git
```

## Part 3: Install Node.js

### Install NVM (Node Version Manager)
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
```

### Load NVM
```bash
source ~/.bashrc
```

### Install Node.js LTS
```bash
nvm install --lts
```

### Verify Installation
```bash
node -v
npm -v
```
Should show: Node v20.x and npm v10.x ✅

## Part 4: Install Nginx

### Install Nginx on Amazon Linux 2023
```bash
sudo dnf install nginx -y
```

### Start & Enable Nginx
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

### Check Status
```bash
sudo systemctl status nginx
```
Press 'q' to quit

### Test in Browser
Open: `http://YOUR_ELASTIC_IP`
Should show: Nginx test page ✅

## Part 5A: Create NEW Node.js Application

### Create App Directory
```bash
sudo mkdir -p /var/www/nodeapp
sudo chown -R ec2-user:ec2-user /var/www/nodeapp
cd /var/www/nodeapp
```

### Initialize Node.js Project
```bash
npm init -y
```

### Install Express
```bash
npm install express
```

### Create Application
```bash
nano app.js
```

Paste This Code:
```javascript
const express = require('express');
const app = express();
const PORT = 3000;

app.get('/', (req, res) => {
  res.json({
    status: 'success',
    message: 'Node.js app running on AWS!',
    server: 'AWS EC2 - Amazon Linux',
    nodeVersion: process.version,
    timestamp: new Date().toISOString()
  });
});

app.get('/health', (req, res) => {
  res.json({
    status: 'healthy',
    uptime: process.uptime()
  });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

Save: Ctrl + O → Enter → Ctrl + X

### Test Application
```bash
node app.js
```

Open new terminal, test:
```bash
curl http://localhost:3000
```
Should show JSON response ✅
Stop the app: Ctrl + C
Continue to Part 5

## Part 5B: Deploy EXISTING Node.js App from Git

### Clone Your Repository
```bash
cd /var/www
sudo mkdir nodeapp
sudo chown -R ec2-user:ec2-user nodeapp
cd nodeapp
git clone https://github.com/yourusername/your-node-repo.git .
```

### Install Dependencies
```bash
npm install
```

### Create/Update .env (if needed)
```bash
nano .env
```

Add your environment variables:
```env
PORT=3000
NODE_ENV=production
DATABASE_URL=your_database_url
```

Save: Ctrl + O → Enter → Ctrl + X

### Test Application
```bash
node app.js
# Or: npm start
```

Test:
```bash
curl http://localhost:3000
```
Stop: Ctrl + C

## Part 6: Install PM2 (Process Manager)

### Install PM2 Globally
```bash
npm install -g pm2
```

### Start App with PM2
```bash
cd /var/www/nodeapp

# Start (replace app.js with your main file)
pm2 start app.js --name nodeapp

# Or if using npm start
pm2 start npm --name nodeapp -- start
```

### Verify Running
```bash
pm2 status
```
Should show: nodeapp status online ✅

### View Logs
```bash
pm2 logs nodeapp
```

## Part 7: Configure PM2 Auto-Start on Reboot

### Save PM2 Process List
```bash
pm2 save
```

### Setup Startup Script
```bash
pm2 startup
```

PM2 will show a command like:
```bash
sudo env PATH=$PATH:/home/ec2-user/.nvm/versions/node/v20.x.x/bin pm2 startup systemd -u ec2-user --hp /home/ec2-user
```

⚠️ IMPORTANT: Copy and run that EXACT command

### Verify
```bash
pm2 list
```
Should show nodeapp running ✅

## Part 8: Configure Nginx as Reverse Proxy

### Create Nginx Config
```bash
sudo nano /etc/nginx/conf.d/nodeapp.conf
```

Paste This Configuration:
```nginx
server {
  listen 80;
  server_name node.yourdomain.com YOUR_ELASTIC_IP;

  location / {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

⚠️ Replace:
- `node.yourdomain.com` with YOUR subdomain (or remove if no domain)
- `YOUR_ELASTIC_IP` with your actual IP
- `3000` with YOUR app port if different

Save: Ctrl + O → Enter → Ctrl + X

### Test Nginx Config
```bash
sudo nginx -t
```
Should show: syntax is ok, test is successful

### Reload Nginx
```bash
sudo systemctl reload nginx
```

## Part 9: Add DNS Record (If Using Domain)

In your domain DNS registrar:
```
Type: A
Host: @
Value: YOUR_ELASTIC_IP
TTL: 3600
```

Wait 5-10 minutes for DNS propagation

### Test in Browser
Open: `http://node.yourdomain.com` (or `http://YOUR_ELASTIC_IP`)
Should show: Your Node.js app response ✅

## Part 10: Update Security Group

### Add HTTP/HTTPS Rules

AWS Console → EC2 → Security Groups
Select your instance's security group
Edit inbound rules
Add these rules:

| Type | Port | Source |
|------|------|--------|
| HTTP | 80 | 0.0.0.0/0 |
| HTTPS | 443 | 0.0.0.0/0 |

Save rules

## Part 11: Setup SSL with Let's Encrypt

### Install Certbot
```bash
sudo dnf install certbot python3-certbot-nginx -y
```

### Obtain SSL Certificate
```bash
sudo certbot --nginx -d node.yourdomain.com
```

Follow prompts:
- Email address: Enter your email
- Terms of Service: Type Y
- Share email: Type N (or Y)

Certbot will auto-configure Nginx for HTTPS

### Verify HTTPS
Open: `https://node.yourdomain.com`
Should show: 🔒 Secure connection + Node.js app ✅

### Check Auto-Renewal
```bash
sudo certbot renew --dry-run
```
Should show: "Congratulations, all simulated renewals succeeded"

## Part 12: Create Deployment Script

### Create Deploy Script
```bash
nano /var/www/deploy-node.sh
```

Paste:
```bash
#!/bin/bash

echo "======================================"
echo "Starting Node.js Deployment"
echo "======================================"

cd /var/www/nodeapp

# Pull latest code
echo "→ Pulling latest code from Git..."
git pull origin main

# Check if pull was successful
if [ $? -eq 0 ]; then
  echo "✓ Code pulled successfully"
else
  echo "✗ Git pull failed!"
  exit 1
fi

# Install/update dependencies
echo "→ Installing dependencies..."
npm install --production

# Check if npm install was successful
if [ $? -eq 0 ]; then
  echo "✓ Dependencies installed"
else
  echo "✗ npm install failed!"
  exit 1
fi

# Restart PM2 app
echo "→ Restarting application..."
pm2 restart nodeapp

# Save PM2 state
pm2 save

echo "======================================"
echo "✓ Deployment Complete!"
echo "======================================"

# Show app status
pm2 status nodeapp
```

Save: Ctrl + O → Enter → Ctrl + X

### Make Executable
```bash
chmod +x /var/www/deploy-node.sh
```

### Use for Updates
```bash
/var/www/deploy-node.sh
```

## Part 13: PM2 Ecosystem Configuration (Advanced)

### Create Ecosystem File
```bash
cd /var/www/nodeapp
nano ecosystem.config.js
```

Paste:
```javascript
module.exports = {
  apps: [{
    name: 'nodeapp',
    script: './app.js',
    instances: 2,
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
    merge_logs: true,
    autorestart: true,
    watch: false,
    max_memory_restart: '1G',
    exp_backoff_restart_delay: 100
  }]
};
```

Save: Ctrl + O → Enter → Ctrl + X

### Create Logs Directory
```bash
mkdir -p /var/www/nodeapp/logs
```

### Restart with Ecosystem
```bash
pm2 delete nodeapp
pm2 start ecosystem.config.js
pm2 save
```

## Git Workflow Commands

### Pull Latest Changes
```bash
cd /var/www/nodeapp
git pull origin main
```

### Check Current Branch
```bash
git branch
```

### Switch Branch
```bash
git checkout branch-name
```

### Check Git Status
```bash
git status
```

### View Recent Commits
```bash
git log --oneline -5
```

### Discard Local Changes (be careful!)
```bash
git reset --hard HEAD
git pull origin main
```

## PM2 Commands Reference
```bash
# Start app
pm2 start app.js --name nodeapp

# Stop app
pm2 stop nodeapp

# Restart app
pm2 restart nodeapp

# Delete app
pm2 delete nodeapp

# View logs (real-time)
pm2 logs nodeapp

# View last 50 lines
pm2 logs nodeapp --lines 50

# Clear logs
pm2 flush

# Monitor resources
pm2 monit

# List all apps
pm2 list

# Show detailed info
pm2 show nodeapp

# Save current state
pm2 save

# Restart all apps
pm2 restart all

# Stop all apps
pm2 stop all
```

## Nginx Commands Reference
```bash
# Test config
sudo nginx -t

# Reload (no downtime)
sudo systemctl reload nginx

# Restart
sudo systemctl restart nginx

# Status
sudo systemctl status nginx

# View error logs
sudo tail -50 /var/log/nginx/error.log

# View access logs
sudo tail -50 /var/log/nginx/access.log

# Follow logs (real-time)
sudo tail -f /var/log/nginx/error.log
```

## Troubleshooting

### Git Pull Permission Denied (SSH)
```bash
# Test SSH connection
ssh -T git@github.com

# Check SSH key
cat ~/.ssh/id_ed25519.pub

# Regenerate if needed
ssh-keygen -t ed25519 -C "your_email@example.com"
# Add new key to GitHub
```

### Git Pull 403 Forbidden (Token)
```bash
# Token expired, generate new one
# Update remote URL with new token
git remote set-url origin https://NEW_TOKEN@github.com/username/repo.git
```

### "npm: command not found" in PM2
```bash
# Check Node path
which node
which npm

# Start with full path
pm2 start /home/ec2-user/.nvm/versions/node/v20.x.x/bin/npm --name nodeapp -- start

# Or use node directly
pm2 start app.js --name nodeapp
```

### "502 Bad Gateway" on Nginx
```bash
# Check if app is running
pm2 status

# Check if port is listening
sudo ss -tlnp | grep :3000

# Restart app
pm2 restart nodeapp

# Check Nginx logs
sudo tail -50 /var/log/nginx/error.log
```

### App Crashes After Git Pull
```bash
# View PM2 logs
pm2 logs nodeapp --lines 100

# Check for missing dependencies
cd /var/www/nodeapp
npm install

# Restart app
pm2 restart nodeapp
```

### SSL Certificate Failed
```bash
# Check DNS
nslookup node.yourdomain.com

# Check port 80 accessible
curl http://node.yourdomain.com

# Try verbose mode
sudo certbot --nginx -d node.yourdomain.com --verbose
```

### PM2 Not Starting on Reboot
```bash
# Check startup script
pm2 startup

# Run the command it shows
# Then save
pm2 save

# Test
sudo reboot
# After reboot: pm2 list
```

## File Structure
```
/var/www/nodeapp/
├── app.js                  # Main application (or server.js)
├── package.json            # Dependencies & scripts
├── package-lock.json       # Locked versions
├── node_modules/           # Installed packages
├── .env                    # Environment variables
├── .git/                   # Git repository data
├── ecosystem.config.js     # PM2 config
└── logs/                   # PM2 logs
    ├── err.log
    └── out.log

/etc/nginx/conf.d/
└── nodeapp.conf            # Nginx reverse proxy config

/home/ec2-user/.ssh/
├── id_ed25519             # Private SSH key (for Git)
└── id_ed25519.pub         # Public SSH key

~/.pm2/
├── logs/                   # PM2 system logs
└── pids/                   # Process IDs
```

## Security Checklist
- ✅ NODE_ENV=production
- ✅ Port 3000 NOT exposed publicly (only via Nginx)
- ✅ SSL certificate installed (HTTPS)
- ✅ SSH key secured (not shared)
- ✅ Token stored securely (if using)
- ✅ .env file not in Git (add to .gitignore)
- ✅ PM2 runs as ec2-user (non-root)
- ✅ Auto-restart enabled
- ✅ Logs monitored regularly
- ✅ Security Group configured properly

## Complete Deployment Workflow

### Initial Setup (One Time)

- ✅ Setup Git authentication (SSH or Token)
- ✅ Install Node.js, Nginx, PM2
- ✅ Clone repository
- ✅ Install dependencies
- ✅ Configure PM2 auto-start
- ✅ Configure Nginx reverse proxy
- ✅ Add DNS record
- ✅ Setup SSL certificate
- ✅ Create deployment script

### Future Updates (Every Time)

SSH into server
Run deployment script: `/var/www/deploy-node.sh`
Verify: `pm2 status` and test in browser

Cost: $0 (within free tier)

## 🗑️ CLEANUP (When Done)

### Stop App
```bash
pm2 delete nodeapp
pm2 save
pm2 unstartup systemd
```

### Remove Files
```bash
sudo rm -rf /var/www/nodeapp
sudo rm /var/www/deploy-node.sh
```

### Remove Nginx Config
```bash
sudo rm /etc/nginx/conf.d/nodeapp.conf
sudo systemctl reload nginx
```

### Remove SSL Certificate
```bash
sudo certbot delete --cert-name node.yourdomain.com
```

### Remove SSH Key from GitHub

GitHub → Settings → SSH and GPG keys
Delete the key

## Quick Test Checklist
- ✅ Git installed: `git --version`
- ✅ Git authenticated: `ssh -T git@github.com` or successful clone
- ✅ Node.js installed: `node -v`
- ✅ npm installed: `npm -v`
- ✅ PM2 installed: `pm2 -v`
- ✅ Nginx installed: `sudo systemctl status nginx`
- ✅ Repository cloned: `ls /var/www/nodeapp`
- ✅ Dependencies installed: `ls /var/www/nodeapp/node_modules`
- ✅ App running: `pm2 status`
- ✅ App accessible locally: `curl http://localhost:3000`
- ✅ Nginx configured: `sudo nginx -t`
- ✅ Security Group: HTTP/HTTPS open
- ✅ Domain works: `http://node.yourdomain.com`
- ✅ SSL works: `https://node.yourdomain.com` 🔒
- ✅ Auto-start configured: `pm2 startup` done
- ✅ Deployment script ready: `/var/www/deploy-node.sh`

## ✅ You now have:

- Git authentication configured (SSH or Token)
- Node.js v20 LTS on Amazon Linux
- App deployed from Git repository
- PM2 process manager with auto-restart
- Nginx reverse proxy
- SSL/HTTPS configured
- Automated deployment script for updates