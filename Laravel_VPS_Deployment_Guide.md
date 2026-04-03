# 🚀 Laravel VPS Deployment Guide

> **MobaXterm · Nginx · MySQL · Certbot SSL · GitHub**
> 
> Generic Template — Replace all placeholders with your actual project values before running any command.

---

## 📋 Requirements

| Component | Requirement |
|-----------|-------------|
| VPS OS | Ubuntu 20.04 / 22.04 / 24.04 |
| Web Server | Nginx |
| PHP Version | 8.1 / 8.2 / 8.3 *(must match your project)* |
| Database | MySQL 8.0+ |
| Node.js | 18+ *(required if project uses Vite)* |
| SSL | Let's Encrypt via Certbot (free) |
| SSH Client | MobaXterm (Windows) |
| Version Control | GitHub / GitLab / Bitbucket |

---

## 🔖 Placeholders — Replace Before You Start

| Placeholder | What to Replace With | Example |
|-------------|----------------------|---------|
| `YOUR_VPS_IP` | Your VPS server IP address | `192.168.1.100` |
| `yourdomain.com` | Your main domain name | `mysite.com` |
| `yoursubdomain` | Subdomain prefix (if applicable) | `app` or `portal` |
| `your_project` | Project folder name on server | `my_laravel_app` |
| `YOUR_GITHUB_URL` | GitHub repository clone URL | `https://github.com/user/repo.git` |
| `your_db_name` | MySQL database name | `myapp_prod` |
| `your_db_user` | MySQL database username | `myapp_user` |
| `your_db_password` | Strong MySQL password | `MyStr0ngPass!` |
| `your_php_version` | Server PHP version | `8.2` |
| `your@email.com` | Email for SSL certificate | `admin@mysite.com` |

> ⚠️ **Replace EVERY placeholder above with your actual values before running any command.**

---

## Step 1 — Add DNS Record in Your Domain Provider

Log in to your domain provider (Hostinger, Namecheap, GoDaddy, Cloudflare, etc.) and add a DNS **A record**:

| Field | Value |
|-------|-------|
| Type | `A` |
| Name / Host | `yoursubdomain` *(or `@` for root domain)* |
| Points To | `YOUR_VPS_IP` |
| TTL | `3600` *(or Auto)* |

> ⚠️ Wait **10–30 minutes** for DNS propagation before running SSL. Verify at [dnschecker.org](https://dnschecker.org)

> ℹ️ For root domain use `@` as the Name. For a subdomain use only the prefix — e.g. `app` for `app.yourdomain.com`

---

## Step 2 — Connect to VPS via MobaXterm

Open MobaXterm and start a new SSH session:

| Field | Value |
|-------|-------|
| Host | `YOUR_VPS_IP` |
| Username | `root` *(or your sudo user)* |
| Port | `22` |

> ℹ️ If you use SSH key authentication, load your private key in **MobaXterm → Settings → SSH → SSH Keys**

---

## Step 3 — Verify Server Requirements

Run these commands to confirm required software is installed:

```bash
# Check installed versions
php -v
nginx -v
mysql --version
composer --version
node -v
npm -v
git --version
certbot --version
```

> ⚠️ If any command is not found, install it before continuing. Your PHP version **must match** your project's `composer.json` requirement.

---

## Step 4 — Clone Project from GitHub

Navigate to the web root and clone your repository:

```bash
cd /var/www/
git clone YOUR_GITHUB_URL your_project
cd /var/www/your_project
```

> ℹ️ Replace `YOUR_GITHUB_URL` with your actual GitHub repo URL and `your_project` with your chosen folder name.

> ⚠️ If your repo is **private**, set up SSH key authentication first (see the [Git SSH Setup](#git-ssh-key-setup) section).

---

## Step 5 — Install PHP Dependencies (Composer)

```bash
composer install --no-dev --optimize-autoloader
```

> ✅ If `composer.lock` exists in your repo, exact package versions are guaranteed across environments.

> ℹ️ The `--no-dev` flag skips development-only packages, keeping the production server clean and secure.

---

## Step 6 — Build Frontend Assets (Vite / Mix)

> ℹ️ Only required if your project uses **Vite** or **Laravel Mix** for frontend assets.

```bash
npm install
npm run build
```

> ❌ This generates the `/public/build/` folder. Without this step **all CSS and JS will be broken** in production.

> ⚠️ Do **NOT** upload or commit the `node_modules/` folder to the server. Only the compiled `public/build/` output is needed.

---

## Step 7 — Create MySQL Database & User

Log in to MySQL as root:

```bash
mysql -u root -p
```

Run these SQL commands:

```sql
CREATE DATABASE your_db_name;
CREATE USER 'your_db_user'@'localhost' IDENTIFIED BY 'your_db_password';
GRANT ALL PRIVILEGES ON your_db_name.* TO 'your_db_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

> ⚠️ Use a strong password with uppercase, lowercase, numbers and special characters.

---

## Step 8 — Configure the .env File

```bash
cp .env.example .env
nano .env
```

Set these critical values inside `.env`:

```env
APP_NAME="Your App Name"
APP_ENV=production
APP_DEBUG=false
APP_URL=https://yoursubdomain.yourdomain.com

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=your_db_name
DB_USERNAME=your_db_user
DB_PASSWORD=your_db_password
```

> ℹ️ Press `CTRL+X` then `Y` then `Enter` to save and exit the nano editor.

> ❌ **NEVER** set `APP_DEBUG=true` in production. It exposes sensitive server information publicly.

---

## Step 9 — Run Laravel Artisan Commands

Run all required artisan commands **in order**:

```bash
# Generate the application encryption key
php artisan key:generate

# Run database migrations
php artisan migrate --force

# Cache configuration, routes and views for performance
php artisan config:cache
php artisan route:cache
php artisan view:cache

# Create the storage symlink
php artisan storage:link
```

> ⚠️ `php artisan key:generate` must run **AFTER** the `.env` file is configured, not before.

> ℹ️ If you have database seeders to run: `php artisan db:seed --force`

---

## Step 10 — Set File Permissions

```bash
chown -R www-data:www-data /var/www/your_project
chmod -R 755 /var/www/your_project
chmod -R 775 /var/www/your_project/storage
chmod -R 775 /var/www/your_project/bootstrap/cache
```

> ⚠️ `www-data` is the Nginx/PHP-FPM user. Without correct permissions Laravel cannot write logs, cache or uploaded files.

---

## Step 11 — Check PHP Version for Nginx Config

```bash
php -v
```

Note the version — you will use it in the next step for the PHP-FPM socket path:

| PHP Version | FPM Socket Path |
|-------------|-----------------|
| PHP 8.1 | `unix:/var/run/php/php8.1-fpm.sock` |
| PHP 8.2 | `unix:/var/run/php/php8.2-fpm.sock` |
| PHP 8.3 | `unix:/var/run/php/php8.3-fpm.sock` |

---

## Step 12 — Create Nginx Virtual Host Configuration

Create a new Nginx config file:

```bash
nano /etc/nginx/sites-available/yoursubdomain.yourdomain.com
```

Paste the following **HTTP-only** configuration *(replace `your_php_version`, `your_project` and domain)*:

```nginx
server {
    listen 80;
    server_name yoursubdomain.yourdomain.com;
    root /var/www/your_project/public;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/phpyour_php_version-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }

    error_log  /var/log/nginx/yourproject_error.log;
    access_log /var/log/nginx/yourproject_access.log;
}
```

> ℹ️ Press `CTRL+X` then `Y` then `Enter` to save.

Enable the site, test and reload Nginx:

```bash
# Remove the default Nginx site (it hijacks unmatched domains)
rm /etc/nginx/sites-enabled/default

# Enable your new site
ln -s /etc/nginx/sites-available/yoursubdomain.yourdomain.com /etc/nginx/sites-enabled/

# Test configuration — must return OK before reloading
nginx -t

# Reload Nginx
systemctl reload nginx
```

> ❌ `nginx -t` must show: **syntax is ok** AND **test is successful**. Fix any errors before reloading.

---

## Step 13 — Install SSL Certificate (Let's Encrypt)

> ⚠️ Only run this **AFTER** DNS has fully propagated. Verify your domain resolves to `YOUR_VPS_IP` first.

```bash
certbot --nginx -d yoursubdomain.yourdomain.com
```

When prompted, choose **Option 2** to redirect all HTTP traffic to HTTPS automatically.

After Certbot succeeds, open the config and replace all content with the full HTTPS configuration:

```bash
nano /etc/nginx/sites-available/yoursubdomain.yourdomain.com
```

```nginx
# Redirect all HTTP to HTTPS
server {
    listen 80;
    server_name yoursubdomain.yourdomain.com;
    return 301 https://$host$request_uri;
}

# Main HTTPS server block
server {
    listen 443 ssl;
    server_name yoursubdomain.yourdomain.com;
    root /var/www/your_project/public;
    index index.php index.html;

    ssl_certificate     /etc/letsencrypt/live/yoursubdomain.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yoursubdomain.yourdomain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/phpyour_php_version-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }

    error_log  /var/log/nginx/yourproject_error.log;
    access_log /var/log/nginx/yourproject_access.log;
}
```

Test and restart Nginx:

```bash
nginx -t
systemctl restart nginx
```

> ℹ️ SSL certificates from Let's Encrypt expire after **90 days**. Certbot auto-renews them. Verify with: `certbot renew --dry-run`

---

## Step 14 — Final Verification

Check all services are running:

```bash
# Check Nginx
systemctl status nginx

# Check PHP-FPM (replace version number)
systemctl status phpyour_php_version-fpm

# Check MySQL
systemctl status mysql

# Watch Laravel logs live for any errors
tail -f /var/www/your_project/storage/logs/laravel.log
```

> ✅ Open your browser in **Incognito / Private** mode and visit `https://yoursubdomain.yourdomain.com` — the padlock icon should appear.

---

## 🔄 Future Updates — Pull & Deploy

Every time you push new code to GitHub, SSH into the server and run:

```bash
cd /var/www/your_project

# Pull latest code
git pull origin main

# Update PHP dependencies (only if composer.lock changed)
composer install --no-dev --optimize-autoloader

# Rebuild frontend assets (only if frontend files changed)
npm run build

# Run any new migrations
php artisan migrate --force

# Clear and rebuild caches
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

> ⚠️ If `git pull` fails with *"local changes would be overwritten"*, run: `git checkout -- path/to/file` then pull again.

---

## 🔑 Git SSH Key Setup (Recommended)

Setting up SSH key authentication eliminates username/password prompts on every `git pull`. This is the professional standard approach.

### Step A — Generate SSH Key on Server

```bash
ssh-keygen -t ed25519 -C "your@email.com"
# Press Enter 3 times (accept defaults, no passphrase)
```

### Step B — Copy the Public Key

```bash
cat ~/.ssh/id_ed25519.pub
# Copy the entire output starting with: ssh-ed25519 AAAA...
```

### Step C — Add Key to GitHub

Go to **GitHub Profile → Settings → SSH and GPG Keys → New SSH Key** → Paste your key → Save.

### Step D — Switch Remote URL to SSH

```bash
cd /var/www/your_project
git remote set-url origin git@github.com:yourusername/your-repo.git
```

### Step E — Test the Connection

```bash
ssh -T git@github.com
# Expected: Hi yourusername! You've successfully authenticated!
```

> ✅ After SSH setup, `git pull` will never ask for a username or password again.

---

## 🔧 Troubleshooting Common Issues

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| Wrong site shows on your domain | Default Nginx site is enabled | `rm /etc/nginx/sites-enabled/default` then reload Nginx |
| 500 Internal Server Error | File permissions or `.env` misconfiguration | Check storage permissions and `APP_KEY` in `.env` |
| CSS / JS not loading | `npm run build` was not run | Run `npm run build` in project folder |
| Certbot timeout error | Port 80 blocked by VPS firewall | Open port 80 and 443 in your VPS provider firewall panel |
| `git pull` asks for password | Using HTTPS remote URL | Switch to SSH remote URL (see [Git SSH Setup](#git-ssh-key-setup)) |
| `git pull` blocked by local changes | Server file was manually edited | Run `git checkout -- path/to/file` then pull again |
| DB connection refused | Wrong DB credentials in `.env` | Verify `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD` in `.env` |
| `php artisan` fails | `composer install` not run yet | Run `composer install --no-dev --optimize-autoloader` |

---

## 📁 Quick Reference — Important Paths & Commands

| Item | Path / Command |
|------|---------------|
| Project root | `/var/www/your_project` |
| Public web root | `/var/www/your_project/public` |
| Laravel logs | `/var/www/your_project/storage/logs/laravel.log` |
| Nginx sites-available | `/etc/nginx/sites-available/` |
| Nginx sites-enabled | `/etc/nginx/sites-enabled/` |
| Nginx error log | `/var/log/nginx/yourproject_error.log` |
| SSL certificates | `/etc/letsencrypt/live/yourdomain.com/` |
| Reload Nginx | `systemctl reload nginx` |
| Restart Nginx | `systemctl restart nginx` |
| Test Nginx config | `nginx -t` |
| Restart PHP-FPM | `systemctl restart phpX.X-fpm` |
| Check all SSL certs | `certbot certificates` |
| Renew SSL manually | `certbot renew` |
| MySQL login | `mysql -u root -p` |
