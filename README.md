# Panduan Deploy Aplikasi PSAT2425

Dokumen ini menjelaskan langkah lengkap untuk melakukan deploy aplikasi **PSAT2425** pada server berbasis Linux (Ubuntu/Debian) menggunakan skrip UserData otomatis. Aplikasi ini menggunakan Apache2 sebagai webserver, PHP sebagai bahasa backend, serta MySQL sebagai database yang terkoneksi melalui konfigurasi environment file `.env`.

---

## Prasyarat

Sebelum mulai, pastikan hal-hal berikut sudah tersedia:

- **Instance Linux (misal AWS EC2 Ubuntu/Debian)** yang sudah siap digunakan dan memiliki akses internet.
- **Database MySQL/MariaDB** yang sudah aktif dan dapat diakses oleh server (contoh: RDS di AWS).
- **Akses SSH** ke instance (untuk pengecekan dan troubleshooting).
- Repository aplikasi tersedia di GitHub:  
  `https://github.com/paknux/crudsiswa.git`

---

## 1. Persiapan Deploy Menggunakan UserData

Jika menggunakan AWS EC2, Anda dapat memanfaatkan **UserData** untuk menjalankan skrip otomatis saat instance baru dibuat.

### Skrip UserData Lengkap

```bash
#!/bin/bash
sudo apt update -y
sudo apt install -y apache2 php php-mysql libapache2-mod-php mysql-client git
sudo rm -rf /var/www/html/{*,.*} || true
sudo git clone https://github.com/paknux/crudsiswa.git /var/www/html
sudo chmod -R 777 /var/www/html
echo DB_USER=admin > /var/www/html/.env
echo DB_PASS=P4ssw0rd123 >> /var/www/html/.env
echo DB_NAME=rdszlatan47 >> /var/www/html/.env
echo DB_HOST=rdszlatan47.cgmvlmwfqd1j.us-east-1.rds.amazonaws.com >> /var/www/html/.env
sudo systemctl restart apache2
```

---

## 2. Penjelasan Skrip UserData

### 2.1 Update Sistem dan Install Paket

- `sudo apt update -y`  
  Memperbarui daftar paket untuk mendapatkan versi terbaru.

- `sudo apt install -y apache2 php php-mysql libapache2-mod-php mysql-client git`  
  Menginstal:
  - **apache2**: web server.
  - **php**: interpreter PHP.
  - **php-mysql & libapache2-mod-php**: modul PHP untuk koneksi MySQL dan integrasi dengan Apache.
  - **mysql-client**: tool untuk tes koneksi ke database.
  - **git**: untuk meng-clone repository aplikasi.

### 2.2 Bersihkan Folder Web Server

```bash
sudo rm -rf /var/www/html/{*,.*} || true
```

Membersihkan isi direktori `/var/www/html` agar kosong sebelum aplikasi di-copy. `|| true` agar tidak error jika folder kosong.

### 2.3 Clone Repository Aplikasi

```bash
sudo git clone https://github.com/paknux/crudsiswa.git /var/www/html
```

Mengambil source code aplikasi dari GitHub ke folder web server.

### 2.4 Set Permission Folder

```bash
sudo chmod -R 777 /var/www/html
```

Memberikan akses penuh agar Apache dan PHP dapat membaca dan menulis file. **Catatan:** untuk produksi, sesuaikan permission agar lebih aman (misal 755).

### 2.5 Buat File Konfigurasi `.env`

```bash
echo DB_USER=admin > /var/www/html/.env
echo DB_PASS=P4ssw0rd123 >> /var/www/html/.env
echo DB_NAME=rdszlatan47 >> /var/www/html/.env
echo DB_HOST=rdszlatan47.cgmvlmwfqd1j.us-east-1.rds.amazonaws.com >> /var/www/html/.env
```

Membuat file konfigurasi environment `.env` yang berisi variabel koneksi database. Aplikasi akan membaca konfigurasi ini untuk terhubung ke database.

### 2.6 Restart Apache2

```bash
sudo systemctl restart apache2
```

Menerapkan konfigurasi dan memastikan web server berjalan dengan setting terbaru.

---

## 3. Verifikasi Deploy

Setelah instance selesai booting dan UserData selesai dijalankan:

1. **Cek status Apache**

   ```bash
   sudo systemctl status apache2
   ```

   Pastikan Apache aktif dan berjalan tanpa error.

2. **Buka aplikasi via browser**

   Akses IP Public instance di browser, contoh:  
   `http://<IP-PUBLIC-INSTANCE>/`

   Pastikan aplikasi `crudsiswa` tampil dengan benar.

3. **Cek koneksi database**

   Dari server, tes koneksi database MySQL:

   ```bash
   mysql -h rdszlatan47.cgmvlmwfqd1j.us-east-1.rds.amazonaws.com -u admin -p
   ```

   Masukkan password: `P4ssw0rd123`  
   Pastikan dapat login tanpa masalah.

---

## 4. Troubleshooting

- **Apache tidak jalan?**  
  Cek log error di: `/var/log/apache2/error.log`

- **Halaman kosong atau error PHP?**  
  Pastikan PHP sudah terinstall dengan benar dan modul php-mysql terpasang.

- **Database gagal koneksi?**  
  - Pastikan security group database mengizinkan akses dari IP server.  
  - Cek username dan password di file `.env`.  
  - Pastikan database sudah aktif dan menerima koneksi remote.

- **Permission Error?**  
  Pastikan folder `/var/www/html` memiliki permission yang cukup agar Apache dan PHP dapat mengakses.

- **UserData tidak berjalan saat instance pertama kali dibuat?**  
  - Pastikan format UserData benar (gunakan bash script lengkap).  
  - Cek logs instance `/var/log/cloud-init-output.log` untuk debug.

---

## 5. Keamanan dan Best Practice

- Jangan gunakan permission `777` pada production, gunakan permission minimal yang diperlukan.
- Simpan credential database di tempat yang aman, gunakan AWS Secrets Manager atau Parameter Store jika memungkinkan.
- Gunakan HTTPS untuk mengamankan akses aplikasi.
- Rutin update sistem dan patch keamanan.

---

## 6. Referensi

- [Dokumentasi Apache2](https://httpd.apache.org/docs/)
- [Dokumentasi PHP](https://www.php.net/manual/en/install.php)
- [GitHub Repository crudsiswa](https://github.com/paknux/crudsiswa)
- [AWS EC2 User Data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)
- [MySQL Client](https://dev.mysql.com/doc/refman/8.0/en/mysql-client.html)

---

**Selamat!** Aplikasi PSAT2425 Anda sudah berhasil dideploy menggunakan skrip otomatis. Jika ada kendala, silakan hubungi admin atau buat issue pada repository GitHub.
