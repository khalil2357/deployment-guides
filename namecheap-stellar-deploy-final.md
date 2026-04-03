# Laravel 11 → Namecheap Stellar Shared Hosting
## Final Deployment Guide (SSH + Private GitHub Repo)

---

## Overview of the Directory Structure

Namecheap Stellar is **cPanel shared hosting**. Your web root is `public_html/`.
Laravel's web-accessible folder is `public/`. The rest of the app must live **outside** `public_html/` for security.

```
/home/yourusername/
├── public_html/              ← cPanel web root (yoursite.com points here)
│   ├── index.php             ← copied once, then manually edited — NEVER overwrite
│   ├── .htaccess             ← copied once — NEVER overwrite
│   ├── build/                ← compiled CSS/JS — updated on every deploy
│   │   ├── manifest.json
│   │   └── assets/
│   ├── favicon.ico
│   ├── robots.txt
│   └── storage/              ← symlink → ~/laravel/storage/app/public
├── laravel/                  ← full Laravel app (private, not web-accessible)
│   ├── app/
│   ├── bootstrap/
│   ├── config/
│   ├── database/
│   ├── public/               ← source of truth for assets
│   ├── resources/
│   ├── routes/
│   ├── storage/
│   └── vendor/
├── bin/
│   └── composer              ← Composer binary
└── deploy.sh                 ← one-command deploy script
```

---

## Prerequisites Checklist

Confirm these in cPanel before starting:

| Requirement | Minimum | Where to check |
|---|---|---|
| PHP version | **8.2+** | cPanel → Select PHP Version |
| PHP extensions | See below | cPanel → MultiPHP Extensions Manager |
| SSH access | Enabled | Namecheap Account → Hosting → Manage → SSH Access |
| Disk space | 500 MB+ | cPanel → Disk Usage |

**Required PHP extensions** — enable all in cPanel → MultiPHP Extensions Manager:

`bcmath` `ctype` `curl` `dom` `fileinfo` `json` `mbstring` `openssl` `pcre` `pdo` `pdo_mysql` `tokenizer` `xml` `zip` `intl`

---

## STEP 1 — Connect via SSH

```bash
ssh yourusername@yoursite.com -p 21098
# Namecheap shared hosting always uses port 21098
```

Verify you are in the right place:
```bash
pwd
# Should show: /home/yourusername
```

---

## STEP 2 — Set PHP Version to 8.2+

Do this in cPanel **before** running Composer.

1. cPanel → **Select PHP Version** (or MultiPHP Manager)
2. Set your domain to **PHP 8.2** or **8.3**
3. Click **Apply**
4. Go to **PHP Extensions** tab → enable all extensions listed above
5. Click **Save**

Verify PHP version in SSH:
```bash
php -v
# Must show: PHP 8.2.x or 8.3.x
```

If SSH still shows an old version, add the correct PHP to your PATH:
```bash
echo 'export PATH=/opt/cpanel/ea-php82/root/usr/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
php -v   # now shows 8.2
```

---

## STEP 3 — Install Composer on the Server

```bash
cd ~
curl -sS https://getcomposer.org/installer | php
mkdir -p ~/bin
mv composer.phar ~/bin/composer
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# Verify
composer --version
```

---

## STEP 4 — Generate a Deploy Key for the Private GitHub Repo

### 4a — Generate SSH keypair on the server

```bash
ssh-keygen -t ed25519 -C "deploy@yoursite.com" -f ~/.ssh/github_deploy
# Press Enter twice for no passphrase
```

### 4b — Copy the public key

```bash
cat ~/.ssh/github_deploy.pub
# Copy the entire output — starts with: ssh-ed25519 AAAA...
```

### 4c — Add it to GitHub as a Deploy Key

1. Go to your GitHub repo → **Settings** → **Deploy keys**
2. Click **Add deploy key**
3. Title: `Namecheap Stellar Production`
4. Key: paste the public key you copied above
5. **Allow write access**: leave **unchecked** (read-only is enough)
6. Click **Add key**

### 4d — Configure SSH to use this key for GitHub

```bash
cat >> ~/.ssh/config << 'EOF'
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_deploy
    IdentitiesOnly yes
EOF

chmod 600 ~/.ssh/config
```

### 4e — Test the connection

```bash
ssh -T git@github.com
# Expected: Hi yourusername! You've successfully authenticated...
```

---

## STEP 5 — Clone the Repository

```bash
cd ~
git clone git@github.com:yourusername/your-repo-name.git laravel
cd ~/laravel

# Confirm correct branch
git branch -a
git checkout main   # or master / production
```

---

## STEP 6 — Build Frontend Assets Locally (on your machine — NOT the server)

The server has no npm. You build locally and commit the compiled output to Git.

**On your local machine:**
```bash
cd your-project
npm install
npm run build
```

This creates:
```
public/build/
├── manifest.json
└── assets/
    ├── app-xxxxxx.css
    └── app-xxxxxx.js
```

**Allow Git to track the build output** — open `.gitignore` and comment out or delete this line:
```gitignore
# /public/build    ← comment out or delete this line
```

**Commit and push:**
```bash
git add public/build/
git commit -m "Add compiled frontend assets for production"
git push origin main
```

**Back on the server**, pull the changes:
```bash
cd ~/laravel
git pull origin main
```

> Every time you change Tailwind classes, JS, or any frontend code — rebuild locally, commit, push, then deploy.

---

## STEP 7 — Install PHP Dependencies

```bash
cd ~/laravel

composer install --optimize-autoloader --no-dev
```

> This may take 2–5 minutes on shared hosting.

---

## STEP 8 — Configure the Environment File

```bash
cd ~/laravel
cp .env.example .env
nano .env
```

**Set these values:**

```env
APP_NAME="Your Company Name"
APP_ENV=production
APP_KEY=                         # leave blank — generate in next step
APP_DEBUG=false
APP_URL=https://yoursite.com

LOG_CHANNEL=single
LOG_LEVEL=error

DB_CONNECTION=mysql
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=yourusername_dbname  # cPanel prefixes DB names with your cPanel username
DB_USERNAME=yourusername_dbuser  # cPanel prefixes DB users too
DB_PASSWORD=your_db_password

CACHE_DRIVER=file
SESSION_DRIVER=file
QUEUE_CONNECTION=sync            # sync = no background worker needed on shared hosting

MAIL_MAILER=smtp
MAIL_HOST=mail.yoursite.com
MAIL_PORT=465
MAIL_USERNAME=info@yoursite.com
MAIL_PASSWORD=your_email_password
MAIL_ENCRYPTION=ssl
MAIL_FROM_ADDRESS=info@yoursite.com
MAIL_FROM_NAME="${APP_NAME}"

FILESYSTEM_DISK=public
```

Save: `Ctrl+O` → Enter → `Ctrl+X`

**Generate the application key:**
```bash
php artisan key:generate
```

---

## STEP 9 — Create the MySQL Database in cPanel

1. cPanel → **MySQL Databases**
2. Create database: e.g. `myproject` → becomes `yourusername_myproject`
3. Create user: e.g. `dbuser` → becomes `yourusername_dbuser` → set a strong password
4. **Add user to database** → grant **All Privileges**
5. Update `.env` with these exact prefixed names

---

## STEP 10 — Run Migrations & Seeders

```bash
cd ~/laravel

php artisan migrate --force

# If you have seeders for initial data
php artisan db:seed --force
```

---

## STEP 11 — Set Permissions

```bash
chmod -R 755 ~/laravel/storage
chmod -R 755 ~/laravel/bootstrap/cache
```

---

## STEP 12 — Set Up `public_html/` — First Time Only

This step is done **once** on initial deployment. Never repeat it — it would overwrite your edited `index.php`.

### 12a — Copy all public assets to public_html

```bash
cp -r ~/laravel/public/. ~/public_html/
```

### 12b — Edit index.php to point to the Laravel app

```bash
nano ~/public_html/index.php
```

Replace the entire file content with:

```php
<?php

use Illuminate\Http\Request;

define('LARAVEL_START', microtime(true));

if (file_exists($maintenance = __DIR__.'/../laravel/storage/framework/maintenance.php')) {
    require $maintenance;
}

require __DIR__.'/../laravel/vendor/autoload.php';

$app = require_once __DIR__.'/../laravel/bootstrap/app.php';

$app->handleRequest(Request::capture());
```

Save: `Ctrl+O` → Enter → `Ctrl+X`

### 12c — Verify .htaccess is correct

```bash
cat ~/public_html/.htaccess
```

It should contain:
```apache
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{HTTP:Authorization} .
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]
</IfModule>
```

If missing, copy it from Laravel:
```bash
cp ~/laravel/public/.htaccess ~/public_html/.htaccess
```

### 12d — Fix the storage symlink

After copying `public/` to `public_html/`, the `storage` symlink is broken because it was a relative path. Recreate it as an absolute path:

```bash
rm ~/public_html/storage
ln -s ~/laravel/storage/app/public ~/public_html/storage

# Verify
ls -la ~/public_html/storage
# Should show: storage -> /home/yourusername/laravel/storage/app/public
```

### 12e — Create the storage link inside Laravel

```bash
php artisan storage:link
```

---

## STEP 13 — Optimise for Production

```bash
cd ~/laravel

php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan event:cache
```

---

## STEP 14 — Verify the Site is Live

Visit `https://yoursite.com` in your browser.

**If DNS hasn't propagated yet**, edit your local machine's hosts file to preview immediately:

**Windows** — open `C:\Windows\System32\drivers\etc\hosts` as Administrator:
```
123.456.789.10    yoursite.com
123.456.789.10    www.yoursite.com
```

**Mac/Linux:**
```bash
sudo nano /etc/hosts
# Add:
123.456.789.10    yoursite.com
123.456.789.10    www.yoursite.com
```

Replace `123.456.789.10` with your server IP from cPanel → General Information → Shared IP Address.

Remove these lines once DNS propagation is complete.

---

## STEP 15 — SSL Certificate

1. cPanel → **SSL/TLS** → **AutoSSL** → run it for your domain
2. Wait 2–5 minutes for the certificate to issue
3. Verify `https://yoursite.com` shows a padlock

Force HTTPS in Laravel — add to `app/Providers/AppServiceProvider.php`:

```php
public function boot(): void
{
    if (config('app.env') === 'production') {
        \URL::forceScheme('https');
    }
}
```

---

## STEP 16 — Set Up Cron Job for Laravel Scheduler

cPanel → **Cron Jobs** → Add New Cron Job:

```
* * * * *    /opt/cpanel/ea-php82/root/usr/bin/php /home/yourusername/laravel/artisan schedule:run >> /dev/null 2>&1
```

---

## STEP 17 — Create the Deploy Script

Save this as `~/deploy.sh` on the server — run it for every future deployment:

```bash
nano ~/deploy.sh
```

```bash
#!/bin/bash
# ~/deploy.sh
# Usage: bash ~/deploy.sh
# Run this on the server after every git push from your local machine.

set -e  # stop immediately on any error

LARAVEL_DIR="$HOME/laravel"
PUBLIC_DIR="$HOME/public_html"
PHP="/opt/cpanel/ea-php82/root/usr/bin/php"

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "  Deploying Laravel App"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""

echo "▶  Enabling maintenance mode..."
$PHP $LARAVEL_DIR/artisan down --retry=5

echo "▶  Pulling latest code from GitHub..."
cd $LARAVEL_DIR
git pull origin main

echo "▶  Installing PHP dependencies..."
$PHP $HOME/bin/composer install --optimize-autoloader --no-dev --no-interaction

echo "▶  Running database migrations..."
$PHP $LARAVEL_DIR/artisan migrate --force

echo "▶  Copying compiled frontend assets (build/)..."
cp -r $LARAVEL_DIR/public/build $PUBLIC_DIR/

echo "▶  Syncing other public assets (favicon, robots.txt, etc.)..."
rsync -a --exclude='index.php' \
         --exclude='.htaccess' \
         --exclude='storage' \
         --exclude='build' \
         $LARAVEL_DIR/public/. $PUBLIC_DIR/

echo "▶  Fixing storage symlink..."
rm -f $PUBLIC_DIR/storage
ln -s $LARAVEL_DIR/storage/app/public $PUBLIC_DIR/storage

echo "▶  Setting permissions..."
chmod -R 755 $LARAVEL_DIR/storage
chmod -R 755 $LARAVEL_DIR/bootstrap/cache

echo "▶  Rebuilding Laravel caches..."
$PHP $LARAVEL_DIR/artisan config:cache
$PHP $LARAVEL_DIR/artisan route:cache
$PHP $LARAVEL_DIR/artisan view:cache
$PHP $LARAVEL_DIR/artisan event:cache

echo "▶  Taking site out of maintenance mode..."
$PHP $LARAVEL_DIR/artisan up

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "  ✅  Deploy complete!"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
```

```bash
chmod +x ~/deploy.sh
```

---

## Your Complete Workflow After Initial Setup

### When you change PHP/Blade code only:

```bash
# Local machine
git add .
git commit -m "Your message"
git push origin main

# Server
ssh yourusername@yoursite.com -p 21098
bash ~/deploy.sh
```

### When you change CSS, JS, or Tailwind classes:

```bash
# Local machine — build first
npm run build

git add .
git commit -m "Rebuild frontend assets"
git push origin main

# Server
ssh yourusername@yoursite.com -p 21098
bash ~/deploy.sh
```

---

## Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| 500 Internal Server Error | Wrong paths in `index.php` | Re-check Step 12b |
| Blank white page | `APP_DEBUG=false` hides errors | Temporarily set `APP_DEBUG=true`, reload, fix, set back |
| "No application encryption key" | `APP_KEY` missing | `php artisan key:generate` |
| DB connection refused | Wrong credentials in `.env` | DB name and username must have cPanel prefix |
| `laravel.log` permission error | Storage not writable | `chmod -R 755 ~/laravel/storage` |
| Images not loading | Storage symlink broken | Re-run Step 12d |
| CSS/JS 404 | Build not copied | `cp -r ~/laravel/public/build ~/public_html/` |
| Vite manifest not found | `public/build` not in Git | Remove `/public/build` from `.gitignore`, rebuild and push |
| `index.php` got overwritten | Ran `cp -r public/.` again | Re-apply the index.php edit from Step 12b |
| Old cached version showing | Caches stale | `php artisan optimize:clear` |
| CSS looks broken after deploy | Old Vite hashed filenames cached | Hard refresh browser `Ctrl+Shift+R` |

---

## Quick Reference

```bash
# SSH into server
ssh yourusername@yoursite.com -p 21098

# Deploy after a git push
bash ~/deploy.sh

# Copy only the build folder manually
cp -r ~/laravel/public/build ~/public_html/

# View live error logs
tail -f ~/laravel/storage/logs/laravel.log

# Clear all caches
cd ~/laravel && php artisan optimize:clear

# Check PHP version
php -v

# Check storage symlink
ls -la ~/public_html/storage

# Put site in / out of maintenance manually
cd ~/laravel
php artisan down
php artisan up
```
