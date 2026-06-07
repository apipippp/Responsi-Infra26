# Responsi Praktikum Infrastruktur Teknologi Informasi

## Deskripsi

Pada responsi ini, Anda diberikan sebuah proyek yang terdiri dari beberapa service yang saling terhubung menggunakan Docker Compose.

Tugas Anda adalah melakukan analisis, troubleshooting, dan perbaikan terhadap konfigurasi yang tersedia hingga seluruh service dapat berjalan sesuai dengan spesifikasi yang ditentukan.

Proyek yang diberikan **tidak berada dalam kondisi siap digunakan**. Oleh karena itu, Anda diharapkan melakukan identifikasi masalah, analisis penyebab, serta melakukan perbaikan yang diperlukan.

---

## Langkah Awal

### 1. Fork Repository

Lakukan fork terhadap repository yang diberikan oleh asisten praktikum ke akun GitHub masing-masing.

### 2. Clone Repository Fork

Clone repository hasil fork ke environment kerja yang digunakan.

Contoh:

```bash
git clone https://github.com/USERNAME/responsi-infra.git
cd responsi-infra
```

Pastikan Anda mengerjakan responsi menggunakan repository hasil fork milik sendiri.

---

## Arsitektur Sistem

Sistem terdiri dari beberapa komponen:

* Nginx Load Balancer
* Web Server 1
* Web Server 2
* Web Server 3
* Database Server

Secara umum arsitektur yang diharapkan adalah sebagai berikut:

```
                +-------------+
                |    NGINX    |
                |Load Balancer|
                +------+------+
                       |
          +------------+------------+
          |            |            |
       +--+--+      +--+--+      +--+--+
       |Web 1|      |Web 2|      |Web 3|
       +--+--+      +--+--+      +--+--+
          \            |            /
           \           |           /
            +----------+----------+
                       |
                  +----+----+
                  |Database |
                  +---------+
```

---

## Tujuan

Pastikan seluruh service dapat berjalan dengan baik dan memenuhi ketentuan berikut:

1. Seluruh container berhasil dijalankan menggunakan Docker Compose.
2. Nginx dapat diakses melalui browser pada port yang telah ditentukan.
3. Nginx berfungsi sebagai load balancer untuk seluruh web server.
4. Load balancing berjalan menggunakan metode Round Robin.
5. Ketiga web server dapat diakses melalui Nginx.
6. Seluruh web server dapat mengakses database yang sama.
7. Informasi identitas praktikan ditampilkan pada masing-masing web server.

---

## Identitas Praktikan

Ganti placeholder identitas yang tersedia pada aplikasi dengan data berikut:

* Nama Lengkap
* NIM

Identitas harus muncul pada seluruh web server.

---

## Pengujian

### Menjalankan Service

```bash
docker compose up -d
```

### Memeriksa Status Container

```bash
docker ps
```

Pastikan seluruh service yang diperlukan berjalan dengan baik.

### Mengakses Aplikasi

Aplikasi harus dapat diakses melalui:

```
http://localhost:8080
```

### Verifikasi Load Balancing

Lakukan refresh halaman beberapa kali.

Output harus bergantian menampilkan:

```
WEB-1
WEB-2
WEB-3
```

beserta identitas praktikan.

### Verifikasi Database

Pastikan seluruh web server dapat mengakses sumber data yang sama.

---

## Analisis dan Dokumentasi

Selain melakukan perbaikan konfigurasi, setiap praktikan wajib membuat dokumentasi analisis.

Buat file baru dengan nama:

```
Analisis.md
```

File tersebut harus berisi penjelasan mengenai setiap permasalahan yang ditemukan selama proses troubleshooting.

Format yang digunakan:

```markdown
# Analisis Perbaikan

## Permasalahan 1

### Gejala
Jelaskan gejala yang ditemukan.

### Penyebab
Jelaskan akar penyebab masalah.

### Solusi
Jelaskan langkah perbaikan yang dilakukan.

---

## Permasalahan 2

### Gejala
...

### Penyebab
...

### Solusi
...
```

Tambahkan seluruh permasalahan yang berhasil ditemukan dan diperbaiki.

Penilaian tidak hanya berdasarkan hasil akhir sistem yang berjalan, tetapi juga berdasarkan kualitas analisis yang diberikan.

---

## Submission

1. Pastikan seluruh perubahan telah tersimpan pada repository fork masing-masing.
2. Commit seluruh perubahan yang dilakukan.
3. Push ke repository GitHub masing-masing.
4. Pastikan file `ANALISIS.md` ikut ter-push ke repository.
5. Submit URL repository sesuai instruksi asisten praktikum.

Contoh:

```bash
git add .
git commit -m "Fix infrastructure configuration"
git push origin main
```

---

## Ketentuan

* Dilarang mengubah struktur folder utama proyek.
* Dilarang menghapus service yang telah disediakan.
* Dilarang mengganti teknologi yang digunakan.
* Diperbolehkan menggunakan dokumentasi resmi dan tools pendukung selama responsi berlangsung.
* Fokus utama penilaian adalah kemampuan analisis, troubleshooting, dan pemahaman konfigurasi sistem.

Selamat mengerjakan.
