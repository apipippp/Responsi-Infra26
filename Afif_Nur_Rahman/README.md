# ANALISIS KONFIGURASI DAN TROUBLESHOOTING

## Identitas Praktikan

| Keterangan  | Isi                                              |
| ----------- | ------------------------------------------------ |
| Nama        | Apip Nur Rahman                                  |
| NIM         | H1H024016                                        |
| Environment | VirtualBox Ubuntu 24.04                          |
| Repository  | Responsi-Infra26                                 |
| Tools       | Docker, Docker Compose, Nginx, PHP Apache, MySQL |

---

# 1. Deskripsi Singkat Project

Project ini merupakan sistem aplikasi berbasis Docker Compose yang terdiri dari beberapa service:

| Service | Container  | Fungsi                          |
| ------- | ---------- | ------------------------------- |
| `nginx` | `nginx-lb` | Load balancer dan reverse proxy |
| `web1`  | `web1`     | Web server backend pertama      |
| `web2`  | `web2`     | Web server backend kedua        |
| `web3`  | `web3`     | Web server backend ketiga       |
| `db`    | `mysql-db` | Database MySQL                  |

Aplikasi diakses melalui:

```bash
localhost:8080
```

Nginx berfungsi menerima request dari port `8080`, kemudian meneruskan request ke `web1`, `web2`, dan `web3` secara bergantian menggunakan metode round robin.

---

# 2. Konfigurasi Akhir yang Digunakan

## 2.1 File `docker-compose.yml`

```yaml
services:
  nginx:
    build:
      context: ./nginx
    container_name: nginx-lb
    ports:
      - "8080:80"
    depends_on:
      - web1
      - web2
      - web3
    networks:
      - frontend

  web1:
    build:
      context: ./web1
    container_name: web1
    environment:
      DB_HOST: db
      DB_NAME: responsi
      DB_USER: student
      DB_PASS: student123
    depends_on:
      - db
    networks:
      - frontend
      - backend

  web2:
    build:
      context: ./web2
    container_name: web2
    environment:
      DB_HOST: db
      DB_NAME: responsi
      DB_USER: student
      DB_PASS: student123
    depends_on:
      - db
    networks:
      - frontend
      - backend

  web3:
    build:
      context: ./web3
    container_name: web3
    environment:
      DB_HOST: db
      DB_NAME: responsi
      DB_USER: student
      DB_PASS: student123
    depends_on:
      - db
    networks:
      - frontend
      - backend

  db:
    build:
      context: ./db
    container_name: mysql-db
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: responsi
      MYSQL_USER: student
      MYSQL_PASSWORD: student123
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - backend

volumes:
  db-data:

networks:
  frontend:
  backend:
```

## 2.2 File `nginx/nginx.conf`

```nginx
events {}

http {
    upstream backend {
        server web1:80;
        server web2:80;
        server web3:80;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
        }
    }
}
```

## 2.3 Dockerfile Web Server

File `web1/Dockerfile`, `web2/Dockerfile`, dan `web3/Dockerfile` menggunakan konfigurasi berikut:

```dockerfile
FROM php:8.2-apache

WORKDIR /var/www/html

COPY . .

EXPOSE 80
```

## 2.4 Dockerfile Database

File `db/Dockerfile` menggunakan konfigurasi berikut:

```dockerfile
FROM mysql:8.0

COPY init.sql /docker-entrypoint-initdb.d/
```

---

# 3. Analisis Permasalahan dan Perbaikan

## 3.1 Kesalahan pada `docker-compose.yml`

### Masalah 1: Key `services` Tidak Menggunakan Tanda Titik Dua

**Konfigurasi salah:**

```yaml
services
  nginx:
    build:
      context: ./nginx
```

**Konfigurasi benar:**

```yaml
services:
  nginx:
    build:
      context: ./nginx
```

**Penyebab:**

File YAML membutuhkan tanda titik dua `:` setelah nama key. Jika `services` tidak diberi tanda `:`, Docker Compose tidak dapat membaca daftar service.

**Penyelesaian:**

Menambahkan tanda titik dua sehingga menjadi:

```yaml
services:
```

---

### Masalah 2: Format YAML dan Indentasi Tidak Valid

**Gejala:**

Saat menjalankan:

```bash
docker compose up -d --build
```

muncul error seperti:

```text
mapping values are not allowed in this context
```

**Penyebab:**

Struktur file `docker-compose.yml` tidak sesuai format YAML. YAML sangat bergantung pada indentasi, baris baru, dan tanda titik dua.

**Penyelesaian:**

File `docker-compose.yml` ditulis ulang dengan indentasi yang benar dan struktur service yang sesuai.

---

### Masalah 3: Build Context `web3` Salah

**Konfigurasi salah:**

```yaml
web3:
  build:
    context: ./web33
```

**Konfigurasi benar:**

```yaml
web3:
  build:
    context: ./web3
```

**Gejala:**

Saat build muncul error:

```text
path "/home/apipnurrr/Responsi-Infra26/web33" not found
```

**Penyebab:**

Folder `web33` tidak ada di dalam project. Folder yang tersedia adalah `web3`.

**Penyelesaian:**

Mengubah `context: ./web33` menjadi `context: ./web3`.

---

### Masalah 4: Volume Database Tidak Didefinisikan dengan Benar

**Konfigurasi salah:**

```yaml
db:
  volumes:
    - db-data:/var/lib/mysql

volumes:
  database-data:
```

**Konfigurasi benar:**

```yaml
db:
  volumes:
    - db-data:/var/lib/mysql

volumes:
  db-data:
```

**Gejala:**

Muncul error:

```text
service "db" refers to undefined volume db-data: invalid compose project
```

**Penyebab:**

Service `db` menggunakan volume `db-data`, tetapi volume yang dideklarasikan adalah `database-data`.

**Penyelesaian:**

Menyamakan nama volume menjadi `db-data`.

---

### Masalah 5: `DB_HOST` Tidak Sesuai

**Konfigurasi salah:**

```yaml
DB_HOST: mysql
```

**Konfigurasi benar:**

```yaml
DB_HOST: db
```

**Penyebab:**

Nama service database pada `docker-compose.yml` adalah `db`, bukan `mysql`. Dalam Docker Compose, komunikasi antar-container menggunakan nama service.

**Penyelesaian:**

Mengubah `DB_HOST` menjadi `db`.

---

### Masalah 6: Password Database Tidak Sesuai

**Konfigurasi salah:**

```yaml
DB_PASS: wrongpassword
```

**Konfigurasi benar:**

```yaml
DB_PASS: student123
```

**Penyebab:**

Password pada web server harus sama dengan `MYSQL_PASSWORD` pada service database.

**Penyelesaian:**

Mengubah `DB_PASS` menjadi `student123`.

---

### Masalah 7: Service `web3` Tidak Terhubung ke Network `frontend`

**Konfigurasi salah:**

```yaml
web3:
  networks:
    - backend
```

**Konfigurasi benar:**

```yaml
web3:
  networks:
    - frontend
    - backend
```

**Gejala:**

Nginx tidak dapat meneruskan request ke `web3`.

**Penyebab:**

Nginx berada pada network `frontend`. Jika `web3` tidak berada di network `frontend`, maka Nginx tidak bisa menjangkau container `web3`.

**Penyelesaian:**

Menambahkan network `frontend` pada service `web3`.

---

## 3.2 Kesalahan pada Dockerfile Web Server

### Masalah: Nama Image PHP Salah

**Konfigurasi salah pada `web1/Dockerfile`:**

```dockerfile
FROM php:8.2-apach
```

**Konfigurasi salah pada `web3/Dockerfile`:**

```dockerfile
FROM php:8.2-apche
```

**Konfigurasi benar:**

```dockerfile
FROM php:8.2-apache
```

**Gejala:**

Saat build muncul error:

```text
php:8.2-apach: not found
```

atau:

```text
php:8.2-apche: not found
```

**Penyebab:**

Terjadi typo pada nama image Docker. Image yang benar adalah `php:8.2-apache`.

**Penyelesaian:**

Mengubah base image pada Dockerfile menjadi:

```dockerfile
FROM php:8.2-apache
```

---

## 3.3 Kesalahan pada `nginx/nginx.conf`

### Masalah 1: File Masih Mengandung Format Markdown

**Konfigurasi salah:**

```text
File nginx.conf masih memiliki tanda markdown pembuka dan penutup code block.
Contohnya terdapat tulisan pembuka seperti nginx code block dan tanda penutup code block.
```

**Konfigurasi benar:**

```nginx
events {}

http {
    upstream backend {
        server web1:80;
        server web2:80;
        server web3:80;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
        }
    }
}
```

**Gejala:**

Container Nginx gagal berjalan dengan status:

```text
Exited (1)
```

Ketika aplikasi diakses:

```bash
curl localhost:8080
```

muncul error:

```text
Failed to connect to localhost port 8080
```

**Penyebab:**

Tanda markdown bukan bagian dari konfigurasi Nginx. Karena file `nginx.conf` masih mengandung tanda tersebut, Nginx gagal membaca konfigurasi.

**Penyelesaian:**

Menghapus seluruh tanda markdown sehingga isi file hanya berupa konfigurasi Nginx murni.

---

### Masalah 2: Nama Backend pada Upstream Salah

**Konfigurasi salah:**

```nginx
upstream backend {
    server web11:80;
    server web2:80;
    server web3:80;
}
```

**Konfigurasi benar:**

```nginx
upstream backend {
    server web1:80;
    server web2:80;
    server web3:80;
}
```

**Penyebab:**

Service `web11` tidak ada di `docker-compose.yml`. Service yang benar adalah `web1`.

**Penyelesaian:**

Mengubah `web11` menjadi `web1`.

---

### Masalah 3: Port Backend `web3` Salah

**Konfigurasi salah:**

```nginx
upstream backend {
    server web1:80;
    server web2:80;
    server web3:8080;
}
```

**Konfigurasi benar:**

```nginx
upstream backend {
    server web1:80;
    server web2:80;
    server web3:80;
}
```

**Penyebab:**

Container web server berbasis PHP Apache berjalan pada port `80`. Port `8080` digunakan oleh host untuk mengakses Nginx, bukan untuk mengakses web server backend.

**Penyelesaian:**

Mengubah port `web3` menjadi `80`.

---

## 3.4 Kesalahan pada `db/Dockerfile`

### Masalah: Instruksi Dockerfile Tidak Tepat

**Konfigurasi salah:**

```dockerfile
FROM mysql:8.0 COPY init.sql /docker-entrypoint-initdb.d/
```

**Konfigurasi benar:**

```dockerfile
FROM mysql:8.0

COPY init.sql /docker-entrypoint-initdb.d/
```

**Penyebab:**

Instruksi `FROM` dan `COPY` seharusnya dipisahkan. `FROM` digunakan untuk menentukan base image, sedangkan `COPY` digunakan untuk menyalin file `init.sql` ke dalam container.

**Penyelesaian:**

Memisahkan instruksi Dockerfile menjadi dua baris.

---

## 3.5 Kesalahan pada `db/init.sql`

### Masalah: File SQL Masih Mengandung Format Markdown

**Konfigurasi salah:**

```text
File init.sql masih mengandung tanda markdown pembuka dan penutup code block.
```

**Konfigurasi benar:**

```sql
CREATE TABLE students (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nim VARCHAR(20),
    nama VARCHAR(100)
);

INSERT INTO students (nim, nama) VALUES
('H1H024016', 'Apip Nur Rahman');
```

**Penyebab:**

File SQL akan dieksekusi langsung oleh MySQL. Jika terdapat tanda markdown, MySQL akan membacanya sebagai bagian dari perintah SQL dan dapat menyebabkan error.

**Penyelesaian:**

Menghapus tanda markdown dan memastikan file hanya berisi perintah SQL murni.

---

## 3.6 Kesalahan pada File `index.php`

### Masalah 1: Identitas Praktikan Belum Diisi

**Konfigurasi salah:**

```php
<?php
echo "<h1>WEB SERVER 1</h1>";
echo "<p>Nama Praktikan: </p>";
echo "<p>NIM: </p>";
echo "<p>Container: WEB-1</p>";
?>
```

**Konfigurasi benar:**

```php
<?php
echo "<h1>WEB SERVER 1</h1>";
echo "<hr>";
echo "<p>Nama Praktikan: Apip Nur Rahman</p>";
echo "<p>NIM: H1H024016</p>";
echo "<p>Container: WEB-1</p>";
?>
```

**Penyebab:**

Nama dan NIM belum diisi sehingga output web belum sesuai ketentuan.

**Penyelesaian:**

Mengisi nama dan NIM pada file `index.php`.

---

### Masalah 2: Label Container Salah

**Konfigurasi salah pada `web2/index.php`:**

```php
<?php
echo "<h1>WEB SERVER 2</h1>";
echo "<p>Container: WEB-WEB</p>";
?>
```

**Konfigurasi benar pada `web2/index.php`:**

```php
<?php
echo "<h1>WEB SERVER 2</h1>";
echo "<hr>";
echo "<p>Nama Praktikan: Apip Nur Rahman</p>";
echo "<p>NIM: H1H024016</p>";
echo "<p>Container: WEB-2</p>";
?>
```

**Konfigurasi salah pada `web3/index.php`:**

```php
<?php
echo "<h1>WEB SERVER 3</h1>";
echo "<p>Container: WEB-WOB</p>";
?>
```

**Konfigurasi benar pada `web3/index.php`:**

```php
<?php
echo "<h1>WEB SERVER 3</h1>";
echo "<hr>";
echo "<p>Nama Praktikan: Apip Nur Rahman</p>";
echo "<p>NIM: H1H024016</p>";
echo "<p>Container: WEB-3</p>";
?>
```

**Penyebab:**

Label container masih salah sehingga hasil pengujian load balancing tidak jelas.

**Penyelesaian:**

Mengubah label container menjadi `WEB-1`, `WEB-2`, dan `WEB-3`.

---

# 4. Hasil Pengujian

Setelah seluruh konfigurasi diperbaiki, container dijalankan dengan perintah:

```bash
docker compose up -d --build
```

Status container dicek menggunakan:

```bash
docker compose ps
```

Aplikasi diuji dengan perintah:

```bash
curl localhost:8080
curl localhost:8080
curl localhost:8080
curl localhost:8080
curl localhost:8080
```

Hasil pengujian menunjukkan bahwa aplikasi berhasil diakses melalui port `8080`.

Request yang dikirim ke `localhost:8080` diteruskan secara bergantian oleh Nginx ke:

```text
WEB-1
WEB-2
WEB-3
```

Hal ini menunjukkan bahwa Nginx load balancer berhasil berjalan dan dapat meneruskan request ke tiga web server backend.

---

# 5. Ringkasan Perbaikan

| File                 | Kesalahan                             | Perbaikan                                 |
| -------------------- | ------------------------------------- | ----------------------------------------- |
| `docker-compose.yml` | Key `services` tidak memakai `:`      | Diubah menjadi `services:`                |
| `docker-compose.yml` | Format YAML tidak valid               | Menulis ulang file dengan indentasi benar |
| `docker-compose.yml` | `web3` mengarah ke `./web33`          | Diubah menjadi `./web3`                   |
| `docker-compose.yml` | Volume `db-data` tidak didefinisikan  | Menambahkan `volumes: db-data:`           |
| `docker-compose.yml` | `DB_HOST` salah                       | Diubah menjadi `db`                       |
| `docker-compose.yml` | Password database tidak sesuai        | Disamakan menjadi `student123`            |
| `docker-compose.yml` | `web3` tidak masuk network `frontend` | Menambahkan network `frontend`            |
| `web1/Dockerfile`    | Image `php:8.2-apach` salah           | Diubah menjadi `php:8.2-apache`           |
| `web3/Dockerfile`    | Image `php:8.2-apche` salah           | Diubah menjadi `php:8.2-apache`           |
| `nginx/nginx.conf`   | Masih ada format markdown             | Tanda markdown dihapus                    |
| `nginx/nginx.conf`   | Backend `web11` salah                 | Diubah menjadi `web1`                     |
| `nginx/nginx.conf`   | Port `web3:8080` salah                | Diubah menjadi `web3:80`                  |
| `db/Dockerfile`      | Instruksi Dockerfile tidak tepat      | Memisahkan `FROM` dan `COPY`              |
| `db/init.sql`        | Masih ada format markdown             | Dibuat menjadi SQL murni                  |
| `index.php`          | Identitas belum diisi                 | Mengisi nama dan NIM                      |
| `index.php`          | Label container salah                 | Diubah menjadi `WEB-1`, `WEB-2`, `WEB-3`  |

---

# 6. Kesimpulan

Berdasarkan hasil troubleshooting, sistem awal belum dapat berjalan karena terdapat beberapa kesalahan konfigurasi pada Docker Compose, Dockerfile, Nginx, database, dan file aplikasi.

Setelah dilakukan perbaikan, seluruh container berhasil berjalan. Aplikasi dapat diakses melalui `localhost:8080`, dan Nginx berhasil meneruskan request ke tiga web server backend secara bergantian.

Dengan demikian, sistem sudah berjalan sesuai tujuan responsi, yaitu menjalankan aplikasi multi-container menggunakan Docker Compose dengan Nginx sebagai load balancer.
