# LAB 2. POSTGRESQL ADMINISTRATION

## 2.1 Operasi Dasar (CLI)
## 2.2 Databases Cluster
## 2.3 Konfigurasi
## 2.4 Databases Management
### 2.4.1 Databases
### 2.4.2 User & Roles
### 2.4.3 Tablespace
### 2.4.4 Databases Schema
### 2.4.5 Table
### 2.4.6 Data
### 2.4.7 View
## 2.5 Databases Security
### 2.5.1 Host Based Access (HBA)
### 2.5.2 Row Level Security
### 2.5.3 Data Encryption


# 2.1 Operasi Dasar (CLI)
Download sampel sebelum memulai lab:

https://github.com/IDN-Training/postgresql-training/blob/main/sample/training_data.sql

Setelah berhasil didownload ke dalam server, import file SQL tersebut:

```bash
psql -f training_data.sql -d postgres -U postgres -h localhost
```
Masuk ke database cluster:
```bash
psql
```
Coba jalankan perintah-perintah berikut untuk mendapatkan informasi seputar databases postgreSQL:
```bash
-- List all databases
\l

-- List all schemas
\dn

-- Use schema
set search_path to training_schema;

-- List all table
\dt

-- Describe course table
\d participants

-- Describe the course table including description
\d+ participants

-- List all tablespaces
\db

-- Execute SQL Statement
select * from participants;

-- Execute SQL Statement, to save output to a file
\o participants_data.txt
SELECT * FROM training_schema.participants;
\o

-- history
\s

-- show current connection
\conninfo

-- show port
show port;

-- show data directory
show data_directory;
```

# 2.2 Databases Cluster

Pada postgreSQL kita bisa membuat banyak cluster dalam satu server. Antar cluster bisa berjalan dengan port yang berbeda.

Untuk membuatnya, login dengan user postgres pada server linux:
```bash
su postgres
```
Inisialisasi atau buat cluster dengan perintah berikut:
```bash
initdb -D training
```
Keterangan: training adalah nama cluster, sekaligus folder yang akan dibuat

Buka konfigurasi database cluster dan ganti port yang digunakan:

```bash
vim training/postgresql.conf
```

Ubah pada parameter port seperti berikut:
```bash
port = 5433
```
Kemudian jalankan cluster dengan perintah berikut:
```bash
# jalankan database cluster
pg_ctl -D training/ -l /var/log/postgresql/training.log start
# periksa status database cluster
pg_ctl -D training/ status
```
Keterangan: opsi -l digunakan untuk menentukan dimana log cluster akan ditulis
Masuk ke cluster dengan perintah berikut:
```bash
# access database cluster
psql -p 5433
# show current connection
\conninfo
# show port
show port;
# show data directory
show data_directory;
```
Kemudian matikan cluster dengan perintah berikut:
```bash
# stop database cluster
pg_ctl -D training/ stop
# periksa status database cluster
pg_ctl -D training/ status
```

# 2.3 Konfigurasi

Konfigurasi postgreSQL bisa dilakukan dengan dua cara. Pertama, konfigurasi langsung dijalankan pada prompt postgreSQL. Kedua, konfigurasi pada file postgresql.conf. Dan disarankan untuk menggunakan cara yang kedua, karena lebih lengkap, permanen, dan tercatat dengan baik.

Konfigurasi langsung pada prompt postgreSQL:

```bash
show all;
show port;
show work_mem;
show data_directory;
show config_file;
set work_mem = 8096;
alter system set work_mem=8192;
select pg_reload_conf();
```

Konfigurasi pada file:
```bash
vim /etc/postgresql/17/main/postgresql.conf
```

Sesuaikan beberapa parameter berikut:
```bash
# Koneksi dan port
listen_addresses = '*'
port = 5432
max_connections = 200
# Keamanan dan Autentikasi
superuser_reserved_connections = 10
authentication_timeout = 10s
# Write Ahead Log
max_wal_size = 1GB
min_wal_size = 80MB
# Pengaturan Log
log_directory = 'log'
log_min_duration_statement = 5000
log_connections = on
```

# 2.4 Databases Management
## 2.4.1 Databases

Masuk ke prompt psql terlebih dahulu. Kemudian lihat database:
```bash
\l
```

Buat database:
```bash
create database training;
```

Ubah nama database:
```bash
ALTER DATABASE training
RENAME TO training_db;
```

Membuat database dari template atau database yang sudah ada:
```bash
create database training_db_new with template training_db;
```

Gunakan database:
```bash
\c training_db;
```

Hapus database:
```bash
drop database training_db_new;
```

# 2.4.2 User & Roles















