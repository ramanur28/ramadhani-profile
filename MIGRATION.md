# 🚀 Panduan Migrasi Server Otomatis (The Ramadhani Website & Analytics)

Dokumen ini menjelaskan cara melakukan **backup, restore, dan migrasi server 1-klik** untuk website The Ramadhani dan database Umami Analytics.

---

## 🛠️ File Otomasi yang Tersedia

| File Script | Fungsi | Cara Menjalankan |
| :--- | :--- | :--- |
| **`migrate.sh`** | **One-Click Migration**: Otomatis backup VPS lama, transfer data, dan restore di VPS baru. | `./migrate.sh <IP_LAMA> <IP_BARU> root ~/.ssh/id_rsa` |
| **`backup-server.sh`** | Membuat arsip terkompresi dari database PostgreSQL Umami, file `.env`, konfigurasi Caddy, dan media uploads. | `./backup-server.sh` |
| **`restore-server.sh`** | Mengekstrak arsip backup, menginisialisasi Docker di VPS baru, merestore database, dan menyalakan container. | `./restore-server.sh <file_backup.tar.gz>` |

---

## 📋 Metode 1: Migrasi Otomatis 1-Klik (Direkomendasikan)

Cukup jalankan satu perintah ini dari komputer lokal Anda:

```bash
./migrate.sh 103.175.217.71 <IP_VPS_BARU> root /path/ke/ssh_key
```

Script akan secara otomatis:
1. Mengekspor database Umami dari container `ramadhani-umami-db` di VPS lama.
2. Mengemas database, konfigurasi `.env`, dan media ke dalam satu arsip `.tar.gz`.
3. Mengirim arsip ke VPS baru.
4. Menjalankan Docker Compose di VPS baru dan mengimpor seluruh data analytics.
5. Menyiapkan Caddy untuk penerbitan SSL HTTPS otomatis.

---

## 📋 Metode 2: Migrasi Manual (Langkah demi Langkah)

### Langkah 1: Buat Backup di VPS Lama
Masuk ke VPS lama dan jalankan:
```bash
cd /var/www/ramadhani
./backup-server.sh
```
*File backup akan dibuat di folder `~/ramadhani_backups/ramadhani_migration_YYYYMMDD_HHMMSS.tar.gz`.*

### Langkah 2: Transfer ke VPS Baru
```bash
scp ~/ramadhani_backups/ramadhani_migration_*.tar.gz root@IP_VPS_BARU:~/
```

### Langkah 3: Restore di VPS Baru
Di VPS baru, jalankan:
```bash
mkdir -p /var/www/ramadhani
scp root@IP_VPS_LAMA:/var/www/ramadhani/restore-server.sh /var/www/ramadhani/
chmod +x /var/www/ramadhani/restore-server.sh

/var/www/ramadhani/restore-server.sh ~/ramadhani_migration_*.tar.gz
```

---

## 🌐 Langkah Terakhir: Update DNS A Record
Setelah proses migrasi selesai, ubah IP pada DNS Management (Cloudflare / Domain Registrar):

- **Type A**: `@` (ramadhani.cloud) &rarr; `IP_VPS_BARU`
- **Type A**: `www` &rarr; `IP_VPS_BARU`
- **Type A**: `analytics` &rarr; `IP_VPS_BARU`

Selesai! Seluruh website, artikel, gambar, dan data analitik Umami telah berpindah ke server baru secara utuh tanpa ada data yang hilang.

---

## 💻 Metode 3: Memindahkan Data dari Server Produksi ke Komputer Lokal

Jika Anda ingin mengunduh (*pull*) database Umami dan file gambar terbaru dari server produksi ke komputer lokal:

Cukup jalankan script otomasi:
```bash
./sync-from-server.sh <IP_VPS> root /path/ke/ssh_key
```

*Contoh:*
```bash
./sync-from-server.sh 103.175.217.71 root ~/.ssh/id_rsa
```

Script akan:
1. Mengekspor database PostgreSQL Umami dari VPS langsung ke folder `./backups/` di komputer Anda.
2. Mengunduh semua file gambar yang diunggah dari CMS ke `./public/images/uploads/`.
