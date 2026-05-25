# Tutorial 6: Install Nginx + PHP on Ubuntu EC2 + Connect Domain

## What You'll Learn

- Install Nginx web server
- Install PHP 8.5
- Configure Nginx to serve PHP
- Connect subdomain to your server
- Test with a PHP website

## Prerequisites

✅ EC2 instance running (t3.micro)
✅ SSH access working
✅ Elastic IP assigned
✅ Domain ready (we'll use subdomain)

## Part 1: Install Nginx

### Connect to Your Server

```bash
ssh -i ~/.ssh/my-aws-key.pem ubuntu@YOUR_ELASTIC_IP
```

### Update System

```bash
sudo apt update
sudo apt upgrade -y
```

### Install Nginx

```bash
sudo apt install nginx -y
```

### Start & Enable Nginx

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

### Check Status

```bash
sudo systemctl status nginx
# Press 'q' to quit
```

### Test in Browser

Open: http://YOUR_ELASTIC_IP

You should see: "Welcome to nginx!" page ✅

## Part 2: Install PHP

### Install PHP + Extensions

```bash
sudo apt install php-fpm php-cli php-common php-mysql php-zip php-gd php-mbstring php-curl php-xml php-bcmath -y
```

### Verify PHP Installation

```bash
php -v
# Should show: PHP 8.x
```

### Find PHP-FPM Socket

```bash
ls /var/run/php/
# Note the version: php8.5-fpm.sock or similar
```

### Check PHP-FPM Status

```bash
sudo systemctl status php8.5-fpm
# Press 'q' to quit
# Replace 8.5 with your version if different
```

## Part 3: Configure Nginx for PHP

### Remove Default Nginx Config

```bash
sudo rm /etc/nginx/sites-enabled/default
```

### Create New Site Config

```bash
sudo nano /etc/nginx/sites-available/mysite
```

Paste This Configuration:

```nginx
server {
    listen 80;
    server_name YOUR_ELASTIC_IP;

    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.5-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

⚠️ **IMPORTANT:**

- Replace `YOUR_ELASTIC_IP` with your actual Elastic IP
- Replace `php8.5-fpm.sock` with YOUR version from `ls /var/run/php/`

**Save:** Ctrl + O → Enter → Ctrl + X

### Enable Site

```bash
sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
```

### Test Nginx Config

```bash
sudo nginx -t
# Should show: syntax is ok, test is successful
```

### Reload Nginx

```bash
sudo systemctl reload nginx
```

## Part 4: Test PHP

### Remove Default HTML Files

```bash
sudo rm /var/www/html/*.html
```

### Create PHP Info File

```bash
sudo nano /var/www/html/info.php
```

Add This Code:

```php
<?php
phpinfo();
?>
```

**Save:** Ctrl + O → Enter → Ctrl + X

### Set Permissions

```bash
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
```

### Test in Browser

Open: http://YOUR_ELASTIC_IP/info.php

You should see: Full PHP configuration page ✅

### Delete Info File (Security)

```bash
sudo rm /var/www/html/info.php
```

## Part 5: Create Sample Website

### Create Index Page

```bash
sudo nano /var/www/html/index.php
```

Add This Code:

```php
<!DOCTYPE html>
<html>
<head>
    <title>My AWS Website</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 50px auto;
            padding: 20px;
            background: #f4f4f4;
        }
        .container {
            background: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        h1 { color: #333; }
        .info {
            background: #e8f5e9;
            padding: 15px;
            border-radius: 5px;
            margin: 20px 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🚀 Welcome to My AWS Website!</h1>
        <div class="info">
            <p><strong>Server IP:</strong> <?php echo $_SERVER['SERVER_ADDR']; ?></p>
            <p><strong>Your IP:</strong> <?php echo $_SERVER['REMOTE_ADDR']; ?></p>
            <p><strong>PHP Version:</strong> <?php echo phpversion(); ?></p>
            <p><strong>Server Time:</strong> <?php echo date('Y-m-d H:i:s'); ?></p>
        </div>
        <p>✅ Nginx is running</p>
        <p>✅ PHP is working</p>
        <p>✅ Server is live on AWS EC2</p>
    </div>
</body>
</html>
```

**Save:** Ctrl + O → Enter → Ctrl + X

### Test in Browser

Open: http://YOUR_ELASTIC_IP

You should see: Your custom website with server info ✅

## Part 6: Connect Subdomain

### Step 1: Add DNS Record

Login to your domain registrar (Namecheap, GoDaddy, etc.)

**Add A Record:**

| Field | Value |
|-------|-------|
| Type | A |
| Host | aws (creates aws.yourdomain.com) |
| Value | YOUR_ELASTIC_IP |
| TTL | 3600 (1 hour) |

**Example:**

- Type: A
- Host: aws (creates aws.yourdomain.com)
- Value: 23.20.211.207 (your Elastic IP)
- TTL: 3600 (1 hour)

Save changes

### Step 2: Update Nginx Config

```bash
sudo nano /etc/nginx/sites-available/mysite
```

Change this line:

```nginx
server_name YOUR_ELASTIC_IP;
```

To this:

```nginx
server_name aws.yourdomain.com YOUR_ELASTIC_IP;
```

Replace `yourdomain.com` with your actual domain

**Save:** Ctrl + O → Enter → Ctrl + X

### Step 3: Test & Reload

```bash
sudo nginx -t
sudo systemctl reload nginx
```

### Step 4: Wait for DNS Propagation

Time: 5-60 minutes

### Step 5: Verify DNS

```bash
# On your local machine (not server)
nslookup aws.yourdomain.com
# Should show your Elastic IP
```

### Step 6: Test in Browser

Open: http://aws.yourdomain.com

You should see: Your website via domain! ✅

## Common Commands Reference

```bash
# Restart Nginx
sudo systemctl restart nginx

# Reload Nginx (faster, no downtime)
sudo systemctl reload nginx

# Check Nginx status
sudo systemctl status nginx

# Test Nginx config
sudo nginx -t

# Restart PHP-FPM (replace version)
sudo systemctl restart php8.5-fpm

# Check Nginx error logs
sudo tail -f /var/log/nginx/error.log

# Check Nginx access logs
sudo tail -f /var/log/nginx/access.log

# Check PHP-FPM logs
sudo journalctl -u php8.5-fpm -n 50

# Check what's listening on port 80
sudo ss -tlnp | grep :80

# Test from server itself
curl http://localhost
```

## Troubleshooting

### "502 Bad Gateway"

```bash
# Check PHP-FPM is running
sudo systemctl status php8.5-fpm

# Restart PHP-FPM
sudo systemctl restart php8.5-fpm

# Check PHP socket exists
ls /var/run/php/
```

### "403 Forbidden"

```bash
# Fix ownership
sudo chown -R www-data:www-data /var/www/html

# Fix permissions
sudo chmod -R 755 /var/www/html

# Check file exists
ls -la /var/www/html/
```

### "Connection Refused"

Check Security Group has HTTP (port 80) open to 0.0.0.0/0
Check Nginx is running

```bash
sudo systemctl status nginx
```

### Domain Not Working

```bash
# Check DNS propagation
nslookup aws.yourdomain.com

# Check Nginx server_name is correct
sudo nano /etc/nginx/sites-available/mysite
```

### PHP Not Processing (Shows Code)

```bash
# Check PHP-FPM socket path matches in Nginx config
ls /var/run/php/
sudo nano /etc/nginx/sites-available/mysite

# Restart both
sudo systemctl restart php8.5-fpm
sudo systemctl reload nginx
```

## File Structure

```
/var/www/html/          # Website root directory
├── index.php           # Homepage

/etc/nginx/
├── sites-available/
│   └── mysite          # Your site config
└── sites-enabled/
    └── mysite          # Symlink to sites-available

/var/run/php/
└── php8.5-fpm.sock     # PHP-FPM socket
```

## Security Group Checklist

Verify these rules exist in `learning-sg`:

| Type | Port | Source | Why |
|------|------|--------|-----|
| SSH | 22 | My IP | Your access only |
| HTTP | 80 | 0.0.0.0/0 | Public website |
| HTTPS | 443 | 0.0.0.0/0 | SSL (next tutorial) |

## Cost

$0 (within free tier)

## 🗑️ CLEANUP (When Done)

### Remove Website Files

```bash
sudo rm -rf /var/www/html/*
```

### Remove Nginx Config

```bash
sudo rm /etc/nginx/sites-enabled/mysite
sudo rm /etc/nginx/sites-available/mysite
sudo systemctl reload nginx
```

### Restore Default Nginx Page

```bash
sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/
sudo systemctl reload nginx
```

### Uninstall (Optional - if done with all web tutorials)

```bash
sudo apt remove nginx php-* -y
sudo apt autoremove -y
```

## Quick Test Checklist

✅ Nginx installed: `sudo systemctl status nginx`
✅ PHP installed: `php -v`
✅ PHP-FPM running: `sudo systemctl status php8.5-fpm`
✅ Website works via IP: http://YOUR_ELASTIC_IP
✅ PHP processes correctly (not showing code)
✅ Permissions correct: `ls -la /var/www/html/`
✅ Security Group has port 80 open
✅ Domain connected (if applicable): http://aws.yourdomain.com
✅ DNS resolves: `nslookup aws.yourdomain.com`

✅ You now have:

- Nginx web server running
- PHP 8.5 installed and configured
- Sample website live
- Subdomain pointing to your server (optional)
