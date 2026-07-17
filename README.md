# Passbolt CE dengan Docker

Repositori ini menyediakan konfigurasi Docker Compose untuk menjalankan Passbolt Community Edition (CE) sesuai alur instalasi resmi Passbolt.

## Prasyarat

Pastikan sistem Anda sudah memenuhi syarat berikut:

- Docker Engine terinstal
- Docker Compose plugin tersedia
- Anda dapat menjalankan perintah Docker tanpa `sudo`
- Server SMTP yang berfungsi untuk notifikasi dan email pemulihan
- Layanan NTP yang berjalan agar autentikasi GPG tidak bermasalah
- Nama domain atau hostname publik, misalnya `https://passbolt.example.com`

> Instalasi dengan Docker dianggap sudah lanjut. Pastikan Anda terbiasa dengan Docker sebelum menjalankannya.

## 1. Download file Compose resmi

Untuk mengikuti langkah resmi Passbolt, unduh file Compose dan checksum-nya:

```bash
curl -LO https://github.com/passbolt/passbolt_docker/releases/latest/download/docker-compose-ce-SHA512SUM.txt
```

Verifikasi integritas file:

```bash
sha512sum -c docker-compose-ce-SHA512SUM.txt
```

Hasil yang diharapkan:

```text
docker-compose-ce.yaml: OK
```

## 2. Clone repositori dan siapkan environment

```bash
git clone <repo-url>
cd passbolt-deployment
cp .env.example .env
```

Edit file `.env` dan ubah nilai berikut:

```env
APP_FULL_BASE_URL=https://passbolt.example.com
MYSQL_DATABASE=passbolt
MYSQL_USER=passbolt
MYSQL_PASSWORD=change_me
DATASOURCES_DEFAULT_HOST=db
DATASOURCES_DEFAULT_USERNAME=passbolt
DATASOURCES_DEFAULT_PASSWORD=change_me
DATASOURCES_DEFAULT_DATABASE=passbolt

EMAIL_DEFAULT_FROM_NAME=Passbolt
EMAIL_DEFAULT_FROM=no-reply@example.com
EMAIL_TRANSPORT_DEFAULT_HOST=smtp.example.com
EMAIL_TRANSPORT_DEFAULT_PORT=587
EMAIL_TRANSPORT_DEFAULT_USERNAME=change_me
EMAIL_TRANSPORT_DEFAULT_PASSWORD=change_me
EMAIL_TRANSPORT_DEFAULT_TLS=true
```

Pastikan Anda menggunakan password yang kuat dan URL publik yang benar. Jangan pernah commit file `.env`; file ini sudah diabaikan oleh Git melalui `.gitignore`.

## 3. Jalankan container

```bash
docker compose up -d
```

Tunggu beberapa menit sampai container siap, lalu cek status:

```bash
docker compose ps
docker compose logs -f
```

## 4. Buat user admin pertama

Setelah container berjalan, buat user admin pertama dengan perintah berikut:

```bash
docker compose exec passbolt su -m -c "/usr/share/php/passbolt/bin/cake passbolt register_user -u YOUR_EMAIL -f YOUR_NAME -l YOUR_LASTNAME -r admin" -s /bin/sh www-data
```

Perintah ini akan menghasilkan tautan pendaftaran yang dapat Anda buka di browser untuk menyelesaikan setup.

## 5. Akses aplikasi

Buka URL yang Anda set di `APP_FULL_BASE_URL`.

Langkah awal yang biasanya harus dilakukan:

1. Buka halaman web Passbolt
2. Selesaikan setup awal
3. Buat user admin pertama jika belum dilakukan

## Perintah berguna

```bash
# Hentikan stack
docker compose down

# Restart stack
docker compose restart

# Perbarui image
docker compose pull
docker compose up -d
```

## Persistensi data

Data database dan material kunci disimpan di volume Docker, sehingga tidak hilang saat container di-restart.

## Catatan keamanan

- Gunakan password yang kuat dan unik
- Jangan simpan secret di repository publik
- Jika Anda menambahkan sertifikat atau kunci privat, simpan di lokasi aman dan jangan commit ke Git
- Untuk lingkungan produksi, disarankan menggunakan tag versi tertentu daripada `latest`

Referensi resmi: https://www.passbolt.com/docs/hosting/install/ce/docker/
