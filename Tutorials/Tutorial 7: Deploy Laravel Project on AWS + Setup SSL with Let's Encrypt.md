# Tutorial 7: Deploy Laravel Project on AWS + Setup SSL with Let's Encrypt

## What You'll Learn

- Install Composer
- Install Laravel (new or from Git)
- Configure Nginx for Laravel
- Setup MySQL database
- Setup SSL certificate (HTTPS)
- Deploy a working Laravel application

## Prerequisites

- ✅ Nginx + PHP installed (Tutorial 6 completed)
- ✅ Domain/subdomain pointing to your server
- ✅ Elastic IP assigned
- ⚠️ IMPORTANT: SSL requires a domain. You cannot use IP address.

## Part 1: Install Required Software

### Install Git
```bash
sudo apt install git -y
```

### Install Composer
```bash
cd ~
curl -sS https://getcomposer.org/installer -o composer-setup.php
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
rm composer-setup.php
```

### Verify Installation
```bash
git --version
composer --version
# Should show versions
```

## Part 2: Install MySQL Database

### Install MySQL Server
```bash
sudo apt install mysql-server -y
```

### Secure MySQL Installation
```bash
sudo mysql_secure_installation
```

Follow prompts:
- Set root password? Y → Enter password (save this!)
- Remove anonymous users? Y
- Disallow root login remotely? Y
- Remove test database? Y
- Reload privilege tables? Y

### Create Database for Laravel
```bash
sudo mysql -u root -p
# Enter your root password
```

In MySQL console:
```sql
CREATE DATABASE laravel_db;
CREATE USER 'laravel_user'@'localhost' IDENTIFIED BY 'your_secure_password';
GRANT ALL PRIVILEGES ON laravel_db.* TO 'laravel_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### Install PHP MySQL Extension
```bash
sudo apt install php-mysql -y
sudo systemctl restart php8.5-fpm
```

## Part 3A: Deploy NEW Laravel Project

### Create New Laravel Project
```bash
cd /var/www
sudo composer create-project laravel/laravel myapp
# Takes 2-3 minutes
```

### Configure Environment
```bash
sudo nano /var/www/myapp/.env
```

Update these lines:
```env
APP_NAME="My AWS App"
APP_ENV=production
APP_DEBUG=false
APP_URL=https://aws.yourdomain.com

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_db
DB_USERNAME=laravel_user
DB_PASSWORD=your_secure_password
```

Save: Ctrl + O → Enter → Ctrl + X

### Run Migrations
```bash
cd /var/www/myapp
sudo -u www-data php artisan migrate
```

Continue to Part 4

## Part 3B: Deploy EXISTING Laravel Project from Git

### Clone Your Repository
```bash
cd /var/www
sudo git clone https://github.com/yourusername/your-repo.git myapp
# Replace with your actual Git URL
```

### Install Dependencies
```bash
cd /var/www/myapp
sudo composer install --optimize-autoloader --no-dev
# --no-dev = Production mode (excludes dev packages)
```

### Setup Environment File
```bash
# Copy example env
sudo cp .env.example .env

# Edit it
sudo nano .env
```

Update:
```env
APP_NAME="Your App Name"
APP_ENV=production
APP_DEBUG=false
APP_URL=https://aws.yourdomain.com

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_db
DB_USERNAME=laravel_user
DB_PASSWORD=your_secure_password
```

Save: Ctrl + O → Enter → Ctrl + X

### Generate Key & Setup
```bash
cd /var/www/myapp

# Generate application key
sudo -u www-data php artisan key:generate

# Create storage link
sudo -u www-data php artisan storage:link

# Run migrations
sudo -u www-data php artisan migrate --force

# Seed database (if your project has seeders)
sudo -u www-data php artisan db:seed --force

# Clear and cache config
sudo -u www-data php artisan config:cache
sudo -u www-data php artisan route:cache
sudo -u www-data php artisan view:cache
```

Continue to Part 4

## Part 4: Set Permissions
```bash
sudo chown -R www-data:www-data /var/www/myapp
sudo chmod -R 755 /var/www/myapp
sudo chmod -R 775 /var/www/myapp/storage
sudo chmod -R 775 /var/www/myapp/bootstrap/cache
```

## Part 5: Configure Nginx for Laravel

### Create Laravel Nginx Config
```bash
sudo nano /etc/nginx/sites-available/laravel
```

Paste This Configuration:
```nginx
server {
    listen 80;
    server_name aws.yourdomain.com;
    
    root /var/www/myapp/public;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.5-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

⚠️ Replace:
- `aws.yourdomain.com` with YOUR actual subdomain
- `php8.5-fpm.sock` with YOUR PHP version (check: `ls /var/run/php/`)

Save: Ctrl + O → Enter → Ctrl + X

### Disable Old Site, Enable Laravel
```bash
sudo rm /etc/nginx/sites-enabled/mysite
sudo ln -s /etc/nginx/sites-available/laravel /etc/nginx/sites-enabled/
```

### Test & Reload Nginx
```bash
sudo nginx -t
sudo systemctl reload nginx
```

### Test in Browser
Open: `http://aws.yourdomain.com`
You should see: Laravel welcome page ✅

## Part 6: Setup SSL with Let's Encrypt

### Install Certbot
```bash
sudo apt install certbot python3-certbot-nginx -y
```

### Obtain SSL Certificate
```bash
sudo certbot --nginx -d aws.yourdomain.com
```

You'll be asked:
- Email address: Enter your email (for renewal notifications)
- Terms of Service: Type Y
- Share email with EFF: Type N (or Y if you want)

Certbot will:
- Verify domain ownership
- Generate SSL certificate
- Automatically configure Nginx for HTTPS
- Setup auto-renewal

Takes 30-60 seconds

### Update .env for HTTPS
```bash
sudo nano /var/www/myapp/.env
```

Change:
```env
APP_URL=https://aws.yourdomain.com
```

Save and clear cache:
```bash
cd /var/www/myapp
sudo -u www-data php artisan config:clear
```

### Verify SSL Works
Open: `https://aws.yourdomain.com` (note the https)
You should see: 🔒 Secure connection + Laravel page ✅

### Check Auto-Renewal
```bash
sudo certbot renew --dry-run
```
Should show: "Congratulations, all simulated renewals succeeded"

## Part 7: Test Laravel Application

### Create Test Route
```bash
sudo nano /var/www/myapp/routes/web.php
```

Add This Route (after existing routes):
```php
Route::get('/test', function () {
    return response()->json([
        'status' => 'success',
        'message' => 'Laravel is working on AWS!',
        'php_version' => phpversion(),
        'laravel_version' => app()->version(),
        'database' => DB::connection()->getPdo() ? 'Connected' : 'Not Connected'
    ]);
});
```

Save: Ctrl + O → Enter → Ctrl + X

### Test in Browser
Open: `https://aws.yourdomain.com/test`
You should see: JSON response with database connected ✅

## Part 8: Create Deployment Script (For Git Projects)

For future updates of your Git project:
```bash
sudo nano /var/www/deploy.sh
```

Paste:
```bash
#!/bin/bash

echo "Starting deployment..."

cd /var/www/myapp

# Pull latest code
echo "Pulling latest code..."
sudo git pull origin main

# Install/update dependencies
echo "Installing dependencies..."
sudo composer install --optimize-autoloader --no-dev

# Clear cache
echo "Clearing cache..."
sudo -u www-data php artisan config:clear
sudo -u www-data php artisan cache:clear
sudo -u www-data php artisan view:clear

# Run migrations
echo "Running migrations..."
sudo -u www-data php artisan migrate --force

# Cache everything
echo "Caching config..."
sudo -u www-data php artisan config:cache
sudo -u www-data php artisan route:cache
sudo -u www-data php artisan view:cache

# Fix permissions
echo "Fixing permissions..."
sudo chown -R www-data:www-data /var/www/myapp
sudo chmod -R 755 /var/www/myapp
sudo chmod -R 775 /var/www/myapp/storage
sudo chmod -R 775 /var/www/myapp/bootstrap/cache

echo "Deployment complete!"
```

Save and make executable:
```bash
sudo chmod +x /var/www/deploy.sh
```

Use it anytime you update your code:
```bash
sudo /var/www/deploy.sh
```

## Common Laravel Commands
```bash
# Navigate to Laravel directory
cd /var/www/myapp

# Clear all cache
sudo -u www-data php artisan optimize:clear

# Run as www-data user (important!)
sudo -u www-data php artisan [command]

# View routes
sudo -u www-data php artisan route:list

# Check Laravel version
php artisan --version

# Run migrations
sudo -u www-data php artisan migrate

# Rollback last migration
sudo -u www-data php artisan migrate:rollback

# Fresh migrations (drop all tables)
sudo -u www-data php artisan migrate:fresh

# Generate app key
sudo -u www-data php artisan key:generate

# Create storage link
sudo -u www-data php artisan storage:link

# Run seeders
sudo -u www-data php artisan db:seed
```

## MySQL Commands Reference
```bash
# Login to MySQL
sudo mysql -u root -p

# Login as Laravel user
mysql -u laravel_user -p laravel_db
```

In MySQL console:
```sql
-- Show databases
SHOW DATABASES;

-- Use database
USE laravel_db;

-- Show tables
SHOW TABLES;

-- View table structure
DESCRIBE users;

-- Query data
SELECT * FROM users;

-- Exit
EXIT;
```

## SSL Certificate Management

### Check Certificate Expiry
```bash
sudo certbot certificates
```

### Renew Manually (optional - auto-renews)
```bash
sudo certbot renew
```

### Test Auto-Renewal
```bash
sudo certbot renew --dry-run
```
Certificates auto-renew every 60 days

### Add More Domains to Certificate
```bash
sudo certbot --nginx -d domain1.com -d www.domain1.com
```

## Troubleshooting

### "500 Internal Server Error"
```bash
# Check Laravel logs
sudo tail -50 /var/www/myapp/storage/logs/laravel.log

# Check Nginx error log
sudo tail -50 /var/log/nginx/error.log

# Check permissions
sudo chown -R www-data:www-data /var/www/myapp
sudo chmod -R 775 /var/www/myapp/storage
sudo chmod -R 775 /var/www/myapp/bootstrap/cache

# Clear cache
cd /var/www/myapp
sudo -u www-data php artisan config:clear
sudo -u www-data php artisan cache:clear
```

### "Could not find driver" (SQLite error)
```bash
# Install MySQL extension
sudo apt install php-mysql -y
sudo systemctl restart php8.5-fpm

# Verify .env uses mysql not sqlite
sudo nano /var/www/myapp/.env
# DB_CONNECTION=mysql (NOT sqlite)

# Clear cache
sudo -u www-data php artisan config:clear
```

### "Access denied for user"
```bash
# Verify MySQL credentials
sudo mysql -u root -p

# In MySQL:
SELECT User, Host FROM mysql.user;
SHOW GRANTS FOR 'laravel_user'@'localhost';

# Recreate user if needed
DROP USER 'laravel_user'@'localhost';
CREATE USER 'laravel_user'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON laravel_db.* TO 'laravel_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### "Composer: command not found"
```bash
# Reinstall Composer
cd ~
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
chmod +x /usr/local/bin/composer
```

### SSL Certificate Failed
```bash
# Check domain points to correct IP
nslookup aws.yourdomain.com

# Check port 80 is accessible (required for verification)
curl http://aws.yourdomain.com

# Check Security Group allows HTTP (port 80)
# Try again with verbose
sudo certbot --nginx -d aws.yourdomain.com --verbose
```

### "Permission Denied" on Storage
```bash
# Fix storage permissions
cd /var/www/myapp
sudo chmod -R 775 storage bootstrap/cache
sudo chown -R www-data:www-data storage bootstrap/cache
```

### Git Pull Permission Denied
```bash
# If you need to pull as specific user
cd /var/www/myapp
sudo -u www-data git pull origin main

# Or change ownership temporarily
sudo chown -R ubuntu:ubuntu /var/www/myapp
git pull origin main
sudo chown -R www-data:www-data /var/www/myapp
```

### CSS/JS Not Loading
```bash
# Clear cache and recompile
cd /var/www/myapp
sudo -u www-data php artisan config:clear
sudo -u www-data php artisan cache:clear
sudo -u www-data php artisan view:clear

# If using npm assets
npm install
npm run build
```

## File Structure
```
/var/www/myapp/
├── app/                # Application code
├── bootstrap/          # Bootstrap files
│   └── cache/         # Cached files (775)
├── config/             # Configuration
├── database/           # Migrations, seeds
├── public/             # Web root (Nginx points here)
│   └── index.php      # Entry point
├── resources/          # Views, assets
├── routes/             # Route definitions
│   └── web.php        # Web routes
├── storage/            # Logs, cache, uploads (775)
│   ├── app/
│   ├── framework/
│   └── logs/
├── .env               # Environment config (sensitive!)
├── .env.example       # Example env file
├── composer.json      # PHP dependencies
└── artisan            # CLI tool

/etc/letsencrypt/live/aws.yourdomain.com/
├── fullchain.pem      # SSL certificate
├── privkey.pem        # Private key
└── chain.pem          # Certificate chain
```

## Security Checklist
- ✅ APP_DEBUG=false in production
- ✅ APP_ENV=production in .env
- ✅ SSL certificate installed (HTTPS)
- ✅ Storage permissions: 775, owned by www-data
- ✅ .env file not in public directory
- ✅ Auto-renewal enabled for SSL
- ✅ HTTP redirects to HTTPS automatically
- ✅ MySQL root accessible only locally
- ✅ Strong database passwords
- ✅ Git credentials secure (use SSH keys or tokens)

Cost: $0 (within free tier)
SSL Certificate: FREE forever with Let's Encrypt

## 🗑️ CLEANUP (When Done)

### Remove Laravel
```bash
sudo rm -rf /var/www/myapp
```

### Remove Database
```bash
sudo mysql -u root -p
```
```sql
DROP DATABASE laravel_db;
DROP USER 'laravel_user'@'localhost';
EXIT;
```

### Remove SSL Certificate
```bash
sudo certbot delete --cert-name aws.yourdomain.com
```

### Remove Nginx Config
```bash
sudo rm /etc/nginx/sites-enabled/laravel
sudo rm /etc/nginx/sites-available/laravel
sudo systemctl reload nginx
```

### Uninstall Software (optional)
```bash
sudo apt remove mysql-server certbot python3-certbot-nginx -y
sudo apt autoremove -y
```

## Quick Test Checklist
- ✅ Git installed: `git --version`
- ✅ Composer installed: `composer --version`
- ✅ MySQL running: `sudo systemctl status mysql`
- ✅ Database created: `sudo mysql -u root -p` → `SHOW DATABASES;`
- ✅ Laravel installed: `ls /var/www/myapp`
- ✅ Dependencies installed: `ls /var/www/myapp/vendor`
- ✅ .env configured: `cat /var/www/myapp/.env`
- ✅ Permissions correct: `ls -la /var/www/myapp/storage`
- ✅ Nginx configured: `sudo nginx -t`
- ✅ Laravel loads: `http://aws.yourdomain.com`
- ✅ SSL works: `https://aws.yourdomain.com` 🔒
- ✅ HTTP redirects to HTTPS
- ✅ Database connected: `/test` endpoint
- ✅ Auto-renewal works: `sudo certbot renew --dry-run`

## Deployment Workflow Summary

### For New Projects:
```
composer create-project laravel/laravel myapp
Configure .env
Run migrations
Set permissions
Configure Nginx
Setup SSL
```

### For Existing Git Projects:
```
git clone [repo] myapp
composer install
Copy & configure .env
php artisan key:generate
Run migrations
Set permissions
Configure Nginx
Setup SSL
Use deploy.sh for future updates
```

## ✅ You now have:

- Laravel 11.x running on AWS
- MySQL database configured
- SSL/HTTPS with Let's Encrypt
- Git integration (for existing projects)
- Deployment script for updates
- Production-ready setup