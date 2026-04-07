# Laravel Herd — Multi PHP Version Management
> Complete Setup Guide for Windows | Git Bash · PowerShell · CMD

---

## Table of Contents

1. [Why Laravel Herd?](#1-why-laravel-herd)
2. [Install Laravel Herd](#2-install-laravel-herd)
3. [Configure XAMPP to Avoid Conflicts](#3-configure-xampp-to-avoid-conflicts)
4. [Add Your Projects Path to Herd](#4-add-your-projects-path-to-herd)
5. [Install Multiple PHP Versions](#5-install-multiple-php-versions)
6. [CLI PHP Version Switching](#6-cli-php-version-switching)
   - [6.1 .php-version Files](#61-set-up-php-version-files)
   - [6.2 Git Bash Setup](#62-git-bash-setup-primary)
   - [6.3 PowerShell Setup](#63-powershell-setup)
   - [6.4 CMD Setup](#64-cmd-setup)
7. [Project .env Configuration](#7-project-env-configuration)
8. [Daily Workflow](#8-daily-workflow)
9. [Quick Reference](#9-quick-reference)
10. [File Structure Summary](#10-file-structure-summary)

---

## 1. Why Laravel Herd?

Managing multiple Laravel projects that need different PHP versions on Windows is painful with XAMPP alone. Herd solves this cleanly.

| Feature | XAMPP Only | XAMPP + Herd |
|---|---|---|
| Multiple PHP versions | Manual config edits | One click in UI |
| Per-project PHP version | Not possible easily | `herd isolate` command |
| Auto switch on `cd` | Not possible | Via `.php-version` file |
| Port conflicts | Everything on port 80 | Herd owns 80, XAMPP on 8080 |
| Virtual hosts | Manual `httpd-vhosts.conf` | Auto-detected by folder name |
| `.test` domains | Manual `hosts` file edits | Automatic via Herd DNS |
| SSL certificates | Complex setup | Built-in, one click |

---

## 2. Install Laravel Herd

### Step 1 — Download

```
https://herd.laravel.com/windows
```

Run the installer. Herd will set up Nginx, PHP, and its DNS resolver automatically.

### Step 2 — Verify Installation

```bash
herd --version
herd status
```

Herd runs in the **system tray** after installation.

---

## 3. Configure XAMPP to Avoid Conflicts

Both XAMPP Apache and Herd Nginx want **port 80**. Move XAMPP Apache to port **8080**.

### Step 1 — Edit `httpd.conf`

Open: `D:\xampp\apache\conf\httpd.conf`

```apache
# Change from:
Listen 80
ServerName localhost:80

# To:
Listen 8080
ServerName localhost:8080
```

### Step 2 — Edit `httpd-vhosts.conf`

Open: `D:\xampp\apache\conf\extra\httpd-vhosts.conf`

```apache
# Change all VirtualHost entries from:
<VirtualHost *:80>

# To:
<VirtualHost *:8080>
```

### Step 3 — Restart XAMPP Apache

In XAMPP Control Panel → **Stop** then **Start** Apache.

XAMPP sites are now at `http://localhost:8080`

> **Note:** XAMPP MySQL keeps running on port 3306 without any changes. Only Apache conflicts with Herd.

---

## 4. Add Your Projects Path to Herd

### Step 1 — Open Herd General Settings

Herd tray icon → **General** → **Herd Paths** → **Add path**

Add your htdocs folder:

```
D:\xampp\htdocs
```

All sub-folders inside `htdocs` are now automatically available as `.test` sites.

### Step 2 — Verify Sites are Detected

```bash
herd links
```

Expected output:
```
ps-pgw-admin        ->  D:\xampp\htdocs\ps-pgw-admin
emerald-ecommerce   ->  D:\xampp\htdocs\emerald-ecommerce
```

Each project folder becomes a `.test` domain automatically:

```
http://ps-pgw-admin.test
http://emerald-ecommerce.test
```

### Step 3 — Flush DNS

```bash
ipconfig /flushdns
```

---

## 5. Install Multiple PHP Versions

### Step 1 — Install via Herd UI

Herd tray → **PHP** → **Install PHP Version**

Install whichever versions your projects need, for example:
- PHP 7.4 — for legacy projects
- PHP 8.0 — if needed
- PHP 8.4 — latest default

Herd stores all versions here:
```
C:\Users\HP\.config\herd\bin\php74\php.exe
C:\Users\HP\. config\herd\bin\php84\php.exe
```

To find exact paths on your machine:

```powershell
Get-ChildItem -Path C:\ -Filter "php.exe" -Recurse -ErrorAction SilentlyContinue
```

### Step 2 — Isolate PHP per Site (Web)

```bash
# Navigate into your project
cd D:\xampp\htdocs\ps-pgw-admin

# Set PHP 7.4 for this site
herd isolate php@7.4

# Verify
herd which
```

> **Important:** `herd isolate` only affects the **web server (Nginx)**. The CLI `php` command needs separate setup — covered in the next section.

---

## 6. CLI PHP Version Switching

`herd isolate` changes PHP for Nginx only. For CLI (running `composer`, `php artisan`, etc.) you need to manage PHP via terminal profiles.

---

### 6.1 Set Up `.php-version` Files

Create a `.php-version` file in each project that needs a non-default PHP version.

```bash
# For legacy project — PHP 7.4
echo "74" > /d/xampp/htdocs/ps-pgw-admin/.php-version

# Modern projects — no file needed, uses default 8.4
```

---

### 6.2 Git Bash Setup (Primary)

#### Create `.bashrc`

```bash
touch ~/.bashrc
notepad ~/.bashrc
```

Paste the full content below:

```bash
# ─────────────────────────────────────────────────
# PHP Version Management
# ─────────────────────────────────────────────────

# Default PHP version
DEFAULT_PHP="84"

# Load default PHP on startup
export PATH="/c/Users/HP/.config/herd/bin/php${DEFAULT_PHP}:$PATH"

# Manual switch functions
php74() {
    export PATH="/c/Users/HP/.config/herd/bin/php74:$PATH"
    echo "Switched to PHP 7.4"
    php -v
}

php84() {
    export PATH="/c/Users/HP/.config/herd/bin/php84:$PATH"
    echo "Switched to PHP 8.4"
    php -v
}

# Auto switch based on .php-version file
auto_php_switch() {
    if [ -f .php-version ]; then
        version=$(cat .php-version)
        export PATH="/c/Users/HP/.config/herd/bin/php${version}:$PATH"
        echo "Auto switched to PHP $version"
    else
        export PATH="/c/Users/HP/.config/herd/bin/php${DEFAULT_PHP}:$PATH"
    fi
}

# Trigger auto switch when cd-ing into a folder
cd() {
    builtin cd "$@"
    auto_php_switch
}

# Trigger auto switch when terminal opens inside a folder
auto_php_switch
```

#### Create `.bash_profile`

```bash
touch ~/.bash_profile
notepad ~/.bash_profile
```

Paste:

```bash
# Load .bashrc
if [ -f ~/.bashrc ]; then
    source ~/.bashrc
fi
```

#### Reload Without Restarting

```bash
source ~/.bashrc
```

---

### 6.3 PowerShell Setup

#### Open PowerShell Profile

```powershell
notepad $PROFILE
```

Profile location:
```
C:\Users\HP\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1
```

Paste the content below:

```powershell
# Default PHP 8.4
$env:PATH = "C:\Users\HP\.config\herd\bin\php84;" + $env:PATH

# Switch to PHP 7.4
function Use-PHP74 {
    $env:PATH = "C:\Users\HP\.config\herd\bin\php74;" + $env:PATH
    Write-Host "Switched to PHP 7.4" -ForegroundColor Green
    php -v
}

# Switch to PHP 8.4
function Use-PHP84 {
    $env:PATH = "C:\Users\HP\.config\herd\bin\php84;" + $env:PATH
    Write-Host "Switched to PHP 8.4" -ForegroundColor Green
    php -v
}
```

#### Reload Without Restarting

```powershell
. $PROFILE
```

---

### 6.4 CMD Setup

Create `C:\Users\HP\php-switch.bat`:

```batch
@echo off
if "%1"=="74" (
    set PATH=C:\Users\HP\.config\herd\bin\php74;%PATH%
    echo Switched to PHP 7.4
    php -v
) else if "%1"=="84" (
    set PATH=C:\Users\HP\.config\herd\bin\php84;%PATH%
    echo Switched to PHP 8.4
    php -v
) else (
    echo Usage: call php-switch 74  or  call php-switch 84
)
```

Add `C:\Users\HP` to System PATH so you can run it from anywhere.

```cmd
call php-switch 74
call php-switch 84
```

---

### Manual Switch Summary

| Terminal | Switch to 7.4 | Switch to 8.4 |
|---|---|---|
| Git Bash | `php74` | `php84` |
| PowerShell | `Use-PHP74` | `Use-PHP84` |
| CMD | `call php-switch 74` | `call php-switch 84` |

> **Note:** Switching is per terminal session. Each new window starts with the default (8.4). Auto-switch via `.php-version` fires on terminal open and on every `cd`.

---

## 7. Project `.env` Configuration

```env
# CORRECT for Herd
APP_URL=http://your-project.test
ASSET_URL=http://your-project.test

# WRONG — causes 404 on all assets
# ASSET_URL=http://your-project.test/public   ← DO NOT include /public

# Database — XAMPP MySQL still works fine on 3306
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=your_database
DB_USERNAME=root
DB_PASSWORD=
```

> **Why no `/public` in ASSET_URL?**
> Herd's Nginx already points the webroot to the `/public` folder. So `http://project.test` already serves from `/public`. Adding `/public` in the URL doubles it → 404 on all assets.

---

## 8. Daily Workflow

### Working on a Legacy Project (PHP 7.4)

```bash
# Open Git Bash inside project folder
# PHP auto-switches to 7.4 from .php-version file

cd /d/xampp/htdocs/ps-pgw-admin
# Output: Auto switched to PHP 7.4

php -v                  # PHP 7.4.x
composer install
php artisan migrate
php artisan serve
```

### Working on a Modern Project (PHP 8.4)

```bash
# No .php-version file = uses default 8.4

cd /d/xampp/htdocs/emerald-ecommerce

php -v                  # PHP 8.4.x
composer install
php artisan migrate
```

### After Changing `php.ini` in Herd

```bash
herd restart
ipconfig /flushdns
```

### Clearing Laravel Cache

```bash
php artisan config:clear
php artisan cache:clear
php artisan view:clear
```

---

## 9. Quick Reference

### Herd CLI Commands

```bash
herd links              # List all detected sites
herd isolate php@7.4    # Set PHP 7.4 for current site (web only)
herd unisolate          # Remove isolation, use global PHP
herd which              # Show PHP version and driver for current site
herd restart            # Restart Nginx + PHP-FPM
herd status             # Check all service status
herd open               # Open current site in browser
herd logs               # Tail Nginx error logs
```

### PHP / Laravel Compatibility

| Laravel | Min PHP | Max PHP | Notes |
|---|---|---|---|
| 6.x | 7.2 | 8.0 | EOL |
| 7.x / 8.x | 7.3 | 8.1 | Use PHP 7.4 |
| 9.x | 8.0 | 8.2 | Active |
| 10.x | 8.1 | 8.2 | Active |
| 11.x | 8.2 | 8.4 | Current LTS |

### Troubleshooting

| Problem | Fix |
|---|---|
| All sites go to same page | Stop XAMPP Apache or move it to port 8080 |
| `php -v` shows wrong version | Run `source ~/.bashrc` or open a new terminal |
| Assets 404 error | Remove `/public` from `ASSET_URL` in `.env` |
| `composer install` fails | Run `php -v` first to confirm correct PHP version |
| Site not detected | Run `herd links` to verify, then `herd restart` |
| `herd isolate` not working for CLI | CLI needs `.php-version` file, not `herd isolate` |
| `.bashrc` not loading | Make sure `.bash_profile` sources `.bashrc` |
| Port conflict with XAMPP | Move XAMPP Apache to port 8080 |

---

## 10. File Structure Summary

```
D:\xampp\htdocs\
    ps-pgw-admin\
        .php-version              ← contains: 74
        .env                      ← APP_URL=http://ps-pgw-admin.test
        public\
        ...
    emerald-ecommerce\
        .env                      ← APP_URL=http://emerald-ecommerce.test
        public\
        ...

C:\Users\HP\
    .bashrc                       ← Git Bash PHP switching config
    .bash_profile                 ← sources .bashrc
    php-switch.bat                ← CMD switching script
    Documents\WindowsPowerShell\
        Microsoft.PowerShell_profile.ps1   ← PowerShell config

C:\Users\HP\.config\herd\bin\
    php74\php.exe
    php84\php.exe
```

---

> **Summary:** Laravel Herd manages web PHP per-site via `herd isolate`. CLI PHP is managed via `.php-version` files + terminal profiles (`.bashrc` for Git Bash, `$PROFILE` for PowerShell, `.bat` for CMD). XAMPP MySQL runs normally. XAMPP Apache moves to port 8080.
