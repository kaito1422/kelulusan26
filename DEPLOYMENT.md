# 🚀 Deployment Guide - Aplikasi Pengumuman Kelulusan SMP Annawawi

## 📋 Requirements

| Requirement | Version |
|---|---|
| PHP | 8.1+ (8.5 recommended) |
| MySQL/MariaDB | 5.7+ / 10.4+ |
| Web Server | Apache with mod_rewrite or Nginx |
| Composer | 2.x |
| Node.js (optional) | 18+ (only needed for asset rebuilding) |
| Extensions | `php-mysql`, `php-zip`, `php-xml`, `php-mbstring`, `php-curl`, `php-gd` |

---

## 📦 Step 1: Upload Files to Hosting

Upload the **entire project** (except `node_modules` and `.env`) to your hosting via FTP/SFTP or Git.

### Via Git:
```bash
git clone <repository-url> /path/to/hosting/public_html
cd /path/to/hosting/public_html
```

### Via FTP:
1. Zip the project: `zip -r kelulusan.zip . -x "node_modules/*" ".env" "storage/*" 2>/dev/null`
2. Upload and unzip to your hosting root

---

## 🔧 Step 2: Environment Configuration

```bash
# Copy environment file
cp .env.example .env

# Generate application key
php artisan key:generate
```

Edit `.env` with your hosting configuration:

```env
APP_NAME="Pengumuman Kelulusan SMP Annawawi"
APP_ENV=production
APP_DEBUG=false
APP_URL=https://yourdomain.com    # ← Change this!

# Database
DB_CONNECTION=mysql
DB_HOST=localhost                  # ← Usually localhost on shared hosting
DB_PORT=3306
DB_DATABASE=your_database_name    # ← Your MySQL database name
DB_USERNAME=your_database_user    # ← Your MySQL username
DB_PASSWORD=your_database_pass    # ← Your MySQL password

# Admin credentials (change these!)
ADMIN_USERNAME=admin
ADMIN_PASSWORD=admin123           # ← Change after first login!

# Google Drive folders (optional - for file fallback)
# GOOGLE_API_KEY=
# GOOGLE_DRIVE_SK_FOLDER_ID=
# GOOGLE_DRIVE_STKA_FOLDER_ID=
```

---

## 📦 Step 3: Install Dependencies

```bash
# Install PHP dependencies
composer install --no-dev --optimize-autoloader

# (Optional) Install Node dependencies and build assets
# npm install && npm run build
```

> **Note:** The `public/build` directory already contains pre-built assets. You only need to rebuild if you modify CSS/JS files.

---

## 🗄️ Step 4: Database Setup

### Option A: Create database from cPanel/phpMyAdmin
1. Login to your hosting cPanel
2. Open **MySQL Databases**
3. Create a new database and user
4. Update `.env` with the database credentials

### Option B: Via command line (if available)
```sql
CREATE DATABASE kelulusan CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### Run Migrations & Seeders
```bash
php artisan migrate --force
php artisan db:seed --force
```

This creates the tables and inserts 3 sample students:
| NISN | Name | Password (DOB) | Status |
|---|---|---|---|
| 0078123456 | Ahmad Fauzi | 120508 | ✅ LULUS |
| 0078123457 | Siti Aisyah | 210908 | ✅ LULUS |
| 0078123458 | Muhammad Rizki | 150108 | ✅ LULUS |

> ⚠️ **Delete or update sample data before going live!**

---

## 🔗 Step 5: Storage Link & Permissions

```bash
# Create symbolic link for public storage
php artisan storage:link

# Set proper permissions (IMPORTANT!)
chmod -R 775 storage bootstrap/cache
chmod -R 775 public/build
```

### Directory Permissions Checklist:
| Path | Permission |
|---|---|
| `storage/` | 775 |
| `storage/app/sk` | 775 (for uploaded SK files) |
| `storage/app/stka` | 775 (for uploaded STKA files) |
| `storage/framework/` | 775 |
| `storage/logs/` | 775 |
| `bootstrap/cache/` | 775 |
| `public/build/` | 755 |

---

## ⚡ Step 6: Optimizations

```bash
# Cache everything for maximum performance
php artisan config:cache
php artisan route:cache
php artisan view:cache

# (Optional) Clear cache for maintenance
# php artisan config:clear
# php artisan route:clear
# php artisan view:clear
```

---

## 🌐 Step 7: Web Server Configuration

### Apache (.htaccess already configured)
The included `public/.htaccess` handles URL rewriting. Ensure **mod_rewrite** is enabled:
```apache
# In httpd.conf or cPanel
RewriteEngine On
```

**If your hosting points to a subdirectory:**
Create a `.htaccess` in the root with:
```apache
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteRule ^(.*)$ public/$1 [L]
</IfModule>
```

### Nginx Configuration
```nginx
server {
    listen 80;
    server_name yourdomain.com;
    root /path/to/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

---

## 🧪 Step 8: Testing

After deployment, verify:

| URL | Expected |
|---|---|
| `https://yourdomain.com/` | ✅ Welcome page with check form |
| `https://yourdomain.com/result/1` | ✅ Result page (if student exists) |
| `https://yourdomain.com/admin/login` | ✅ Admin login form |
| `https://yourdomain.com/admin/dashboard` | ✅ Admin dashboard (after login) |

Test NISN check:
```bash
# Using curl
curl -X POST https://yourdomain.com/check \
  -d "nisn=0078123456&password=120508"
```

---

## 📁 File Upload Setup

The application supports uploading **PDF** files (SK and STKA certificates):

### Local File Storage
After upload, files are stored in:
- `storage/app/sk/{nisn}.pdf` - Surat Kelulusan
- `storage/app/stka/{nisn}.pdf` - Sertifikat TKA

Files are served via the `public/storage` symlink:
- `/storage/sk/{nisn}.pdf`
- `/storage/stka/{nisn}.pdf`

### Google Drive Fallback (Optional)
If no local file exists, the app redirects to Google Drive folders.
Configure in `.env`:
```env
GOOGLE_API_KEY=your_api_key
GOOGLE_DRIVE_SK_FOLDER_ID=folder_id_here
GOOGLE_DRIVE_STKA_FOLDER_ID=folder_id_here
```

---

## 🔐 Security Checklist

- [ ] Change `ADMIN_PASSWORD` in `.env` after first login
- [ ] Set `APP_DEBUG=false` in production
- [ ] Use HTTPS (SSL certificate)
- [ ] Remove sample student data
- [ ] Set strong database password
- [ ] Restrict file permissions (775 for storage, 755 for others)
- [ ] Enable `APP_ENV=production`

---

## 📊 Maintenance Commands

```bash
# Add new student
php artisan tinker
> App\Models\Student::create([
    'nisn' => '1234567890',
    'name' => 'Student Name',
    'birth_date' => '2008-01-01',
    'class' => 'IX-A',
    'status' => 'LULUS',
    'scores' => '85',
  ]);

# Bulk import via CSV
# Use admin panel: /admin/import

# Backup database
mysqldump -u user -p kelulusan > backup.sql

# Clear application cache
php artisan optimize:clear
```

---

## ❗ Troubleshooting

| Problem | Solution |
|---|---|
| `500 Server Error` | Check storage permissions: `chmod -R 775 storage bootstrap/cache` |
| `No application key` | Run `php artisan key:generate` |
| `Page Expired (419)` | Session issue - check storage permissions |
| `SQLSTATE[HY000]` | Wrong DB credentials in `.env` |
| `Class not found` | Run `composer dump-autoload` |
| `Vite manifest not found` | Run `npm install && npm run build` |
| `_token mismatch` | Clear sessions: `rm -rf storage/framework/sessions/*` |
| `Storage link broken` | Run `php artisan storage:link` |
| `404 on pages` | Ensure Apache mod_rewrite is enabled / Nginx config is correct |

---

## 📝 Quick Deploy Checklist

- [ ] Upload all files to hosting
- [ ] Run `composer install --no-dev --optimize-autoloader`
- [ ] Copy `.env.example` to `.env` and configure
- [ ] Run `php artisan key:generate`
- [ ] Create database and update `.env`
- [ ] Run `php artisan migrate --force`
- [ ] Run `php artisan db:seed --force`
- [ ] Run `php artisan storage:link`
- [ ] Set correct permissions
- [ ] Run `php artisan config:cache && php artisan route:cache && php artisan view:cache`
- [ ] Test all pages
- [ ] Change admin password
- [ ] Done! 🎉

---

## 🔄 Updating

```bash
# Pull latest changes
git pull

# Update dependencies
composer install --no-dev --optimize-autoloader

# Run new migrations
php artisan migrate --force

# Rebuild cache
php artisan optimize:clear
php artisan config:cache
php artisan route:cache
php artisan view:cache