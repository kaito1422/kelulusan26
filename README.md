# 🎓 Pengumuman Kelulusan SMP Annawawi

Aplikasi web untuk pengumuman kelulusan siswa berbasis **Laravel 13** + **Tailwind CSS 4**. Siswa dapat mengecek kelulusan menggunakan **NISN** dan **Tanggal Lahir**, serta mendownload **Surat Kelulusan (SK)** dan **Sertifikat TKA** dalam format PDF.

---

## ✨ Fitur

- ✅ **Cek Kelulusan** — Siswa memasukkan NISN + Tanggal Lahir (DDMMYY)
- ✅ **Hasil Detail** — Menampilkan status, nama, kelas, nilai, dll.
- ✅ **Download PDF** — Download Surat Kelulusan (SK) dan Sertifikat TKA
- ✅ **Cetak/Print** — Halaman hasil yang siap cetak
- ✅ **Admin Panel** — Login, CRUD siswa, import CSV, upload file PDF
- ✅ **Upload File** — Upload PDF/ZIP/RAR, otomatis cocokkan dengan NISN
- ✅ **Google Drive Fallback** — Jika file tidak ada di lokal, redirect ke Google Drive
- ✅ **Responsive** — Tampilan mobile-friendly dengan Tailwind CSS
- ✅ **Production Ready** — Siap deploy ke hosting

---

## 🛠️ Tech Stack

| Component | Technology |
|---|---|
| Framework | Laravel 13 |
| Frontend | Tailwind CSS 4 + Vite |
| Database | MySQL / MariaDB |
| PDF | DomPDF / Barryvdh DomPDF |
| Auth | Session-based (NISN + DOB) |
| Admin | Custom middleware auth |

---

## ⚙️ Installation

### Prerequisites
- PHP 8.1+
- Composer 2.x
- MySQL/MariaDB
- Node.js 18+ (optional, for asset building)

### Quick Start
```bash
# 1. Clone the project
git clone <your-repo> kelulusan
cd kelulusan

# 2. Install PHP dependencies
composer install

# 3. Environment setup
cp .env.example .env
php artisan key:generate

# 4. Edit .env with your database credentials
#    DB_DATABASE, DB_USERNAME, DB_PASSWORD

# 5. Database migration & seed
php artisan migrate
php artisan db:seed

# 6. Storage link
php artisan storage:link

# 7. Build frontend assets
npm install
npm run build

# 8. Start development server
php artisan serve
```

Visit `http://localhost:8000` 🎉

---

## 📁 Project Structure

```
kelulusan/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── AdminController.php        # Admin CRUD + file upload
│   │   │   └── AnnouncementController.php # Public check + download
│   │   └── Middleware/
│   │       └── AdminMiddleware.php        # Admin auth guard
│   ├── Imports/
│   │   └── StudentsImport.php             # CSV import (Laravel Excel)
│   └── Models/
│       └── Student.php                    # Student model
├── database/
│   ├── migrations/                         # Table structures
│   └── seeders/
│       └── StudentSeeder.php              # Sample data
├── resources/
│   └── views/
│       ├── welcome.blade.php              # Public home (check form)
│       ├── result.blade.php               # Result page
│       └── admin/
│           ├── login.blade.php            # Admin login
│           ├── dashboard.blade.php        # Admin dashboard
│           ├── create.blade.php           # Add student
│           ├── edit.blade.php             # Edit student
│           ├── import.blade.php           # CSV import
│           └── upload-files.blade.php     # Upload PDF/ZIP/RAR
├── routes/
│   └── web.php                            # All routes
└── storage/
    ├── app/
    │   ├── sk/                            # Uploaded SK files
    │   └── stka/                          # Uploaded STKA files
    └── app/public/                        # Public storage (symlinked)
```

---

## 🚀 Deployment

See **[DEPLOYMENT.md](DEPLOYMENT.md)** for complete deployment instructions.

### Quick Deploy Checklist
- [ ] Upload all files to hosting
- [ ] Run `composer install --no-dev --optimize-autoloader`
- [ ] Configure `.env` (database, app URL, admin credentials)
- [ ] Run `php artisan key:generate`
- [ ] Run `php artisan migrate --force && php artisan db:seed --force`
- [ ] Run `php artisan storage:link`
- [ ] Set correct permissions (storage, bootstrap/cache)
- [ ] Run `php artisan config:cache && php artisan route:cache && php artisan view:cache`

---

## 🔐 Admin Access

| URL | Path |
|---|---|
| Login | `/admin/login` |
| Dashboard | `/admin/dashboard` |
| Add Student | `/admin/create` |
| Import CSV | `/admin/import` |
| Upload Files | `/admin/upload-files` |

**Default credentials** (change immediately!):
```
Username: admin
Password: admin123
```

Configure in `.env`:
```env
ADMIN_USERNAME=admin
ADMIN_PASSWORD=admin123
```

---

## 👨‍🎓 Student Check

1. Visit `https://yourdomain.com/`
2. Enter **NISN** (e.g., `0078123456`)
3. Enter **Tanggal Lahir** in DDMMYY format (e.g., `120508` for 12 May 2008)
4. Click **CEK KELULUSAN**

---

## 📄 CSV Import Format

| nisn | name | birth_date | class | status | scores |
|---|---|---|---|---|---|
| 1234567890 | John Doe | 2008-01-01 | IX-A | LULUS | 85 |

Upload via: `/admin/import`

---

## 📁 File Upload

Upload **Surat Kelulusan (SK)** and **Sertifikat TKA** via:
- Single PDF files (named by NISN, e.g., `1234567890.pdf`)
- ZIP/RAR archives containing multiple PDFs

Files are stored in:
- `storage/app/sk/{nisn}.pdf`
- `storage/app/stka/{nisn}.pdf`

Served publicly at:
- `/storage/sk/{nisn}.pdf`
- `/storage/stka/{nisn}.pdf`

---

## 📝 License

This project is licensed under the MIT License.