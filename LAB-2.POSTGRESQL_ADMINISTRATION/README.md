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

## 2.4.2 User & Roles

Buat user dengan nama admin dengan password:
```bash
create role admin login password 'password';
```

Beri izin akses superuser:
```bash
alter role admin superuser;
```

Ubah kepemilikan database training_db menjadi milik admin:
```bash
alter database training_db owner to admin;
```

Coba keluar dan koneksi kembali dengan user yang telah dibuat:
```bash
psql -h localhost -U admin -p 5432 -d training_db
```

## 2.4.3 Tablespace
Periksa ketersediaan tablespace:
```bash
\db
```

Keluar dari prompt psql dan buat folder dengan user root dan ubah kepemilikan menjadi milik user linux postgres:
```bash
mkdir /data
chown postgres:postgres /data
```

Koneksi kembali dengan user admin:
```bash
psql -h localhost -U admin -p 5432 -d training_db
```

Buat tablespace dengan lokasi folder yang telah dibuat:
```bash
create tablespace data location '/data';
```

Buat database dengan tablespace yang telah dibuat:
```bash
create database training_data tablespace data;
```

Hapus tablespace:
```bash
drop tablespace data;
```

## 2.4.4 Databases Schema

Buat schema baru:
```bash
CREATE SCHEMA training_schema;
```

Periksa schema yang telah dibuat:
```bash
\dn
```

Periksa schema yang digunakan secara default:
```bash
SELECT current_schema();
```

Periksa schema default dengan cara lain:
```bash
SHOW search_path;
```

Atur default schema ke schema baru yang telah dibuat:
```bash
set search_path to training_schema;
```

Periksa tabel yang ada di dalam schema baru:
```bash
\dt
```

## 2.4.5 Table
Buat tabel baru:

```bash
CREATE TABLE training_schema.participants (
    id_participant SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(15)
);
```

Buat tabel baru:
```bash
CREATE TABLE training_schema.instructors (
    id_instructor SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(15),
    specialization VARCHAR(100)
);
```

Buat tabel baru:
```bash
CREATE TABLE training_schema.courses (
    id_course SERIAL PRIMARY KEY,
    course_name VARCHAR(100) NOT NULL,
    description TEXT,
    start_date DATE,
    end_date DATE,
    id_instructor INTEGER REFERENCES training_schema.instructors(id_instructor)
);
```

Buat tabel baru:
```bash
CREATE TABLE training_schema.materials (
    id_material SERIAL PRIMARY KEY,
    material_name VARCHAR(100) NOT NULL,
    description TEXT,
    id_course INTEGER REFERENCES training_schema.courses(id_course)
);
```

Buat tabel baru:
```bash
CREATE TABLE training_schema.registrations (
    id_registration SERIAL PRIMARY KEY,
    id_participant INTEGER REFERENCES training_schema.participants(id_participant),
    id_course INTEGER REFERENCES training_schema.courses(id_course),
    registration_date DATE DEFAULT CURRENT_DATE
);
```

Periksa tabel yang telah dibuat:
```bash
\dt
```

Lihat struktur tabel:
```bash
\d training_schema.participants
```

Lihat struktur tabel lebih detail:
```bash
\d+ training_schema.participants
```

Masukkan data ke dalam tabel:
```bash
INSERT INTO training_schema.participants (name, email, phone) VALUES
    ('Budi', '<andi@gmail.com>', '123456789'),
    ('Andi', '<budi@gmail.com>', '987654321');
```

Masukkan data ke dalam tabel:
```bash
INSERT INTO training_schema.instructors (name, email, phone, specialization) VALUES
    ('Rafi Riadi', '<rafi.riadi@gmail.com>', '111222333', 'Data Science'),
    ('John Doe', '<jane.doe@gmail.com>', '444555666', 'Machine Learning');
```

Masukkan data ke dalam tabel:
```bash
INSERT INTO training_schema.courses (course_name, description, start_date, end_date, id_instructor) VALUES
    ('Data Science 101', 'Introduction to Data Science', '2024-07-01', '2024-07-15', 1),
    ('Machine Learning Basics', 'Fundamentals of Machine Learning', '2024-08-01', '2024-08-10', 2);
```

Masukkan data ke dalam tabel:
```bash
INSERT INTO training_schema.materials (material_name, description, id_course) VALUES
    ('Introduction Slides', 'Slides for introduction', 1),
    ('Machine Learning Overview', 'Overview of Machine Learning', 2);
```

Masukkan data ke dalam tabel:
```bash
INSERT INTO training_schema.registrations (id_participant, id_course) VALUES
    (1, 1),
    (2, 2);
```

Lihat data di dalam tabel:
```bash
SELECT * FROM training_schema.participants;
```

Lihat data di dalam tabel:
```bash
SELECT * FROM training_schema.instructors;
```

Lihat data di dalam tabel:
```bash
SELECT * FROM training_schema.courses;
```

Lihat data di dalam tabel:
```bash
SELECT * FROM training_schema.materials;
```

Lihat data di dalam tabel:
```bash
SELECT * FROM training_schema.registrations;
```

## 2.4.6 Data
Ubah atau perbarui data yang ada di dalam tabel:
```bash
UPDATE training_schema.participants
SET phone = '1122334455'
WHERE name = 'Budi';

UPDATE training_schema.instructors
SET specialization = 'Deep Learning'
WHERE name = 'Jhon Doe';
```

Hapus data yang ada di dalam tabel:
```bash
DELETE FROM training_schema.registrations
WHERE id_participant = 2 AND id_course = 2;
```

## 2.4.7 View
Buat view dengan menggabungkan beberapa tabel:
```bash
CREATE VIEW training_schema.participant_courses AS
SELECT
    p.name AS participant_name,
    c.course_name,
    r.registration_date
FROM
    training_schema.registrations r
    JOIN training_schema.participants p ON r.id_participant = p.id_participant
    JOIN training_schema.courses c ON r.id_course = c.id_course;
```

Periksa data dari view yang telah dibuat:
```bash
SELECT * FROM training_schema.participant_courses;
```

# 2.5 Databases Security
## 2.5.1 Host Based Access (HBA)
Masuk ke dalam cluster database:
```bash
psql
```

Periksa lokasi file HBA:
```bash
show hba_file;
```

Buka file HBA:
```bash
vim /etc/postgresql/17/main/pg_hba.conf
```

Tambahkan konfigurasi berikut di baris paling bawah atau bersama baris serupa yang telah ada:
```bash
# TYPE DATABASE USER ADDRESS METHOD
host training_db admin 0.0.0.0/0 md5
```

Remote database dari server lain atau dari client:
```bash
psql -h 172.23.x.x -U admin -p 5432 -d training_db
```

## 2.5.2 Row Level Security
Buat dua user untuk mencoba fitur keamanan tingkat row/baris:
```bash
create user andi with password 'andi123';  
create user budi with password 'budi123';
```

Pastikan dua user baru telah berhasil dibuat:
```bash
select rolname from pg_roles;
```

Berikan beberapa izin kepada dua user baru tersebut:
```bash
grant connect on database training_db to andi, budi;  
grant usage on schema training_schema to andi, budi;  
grant select on all tables in schema training_schema to andi, budi;
```

Lihat informasi dari schema yang nantinya akan digunakan:
```bash
SELECT table_name  
FROM information_schema.tables  
WHERE table_schema = 'training_schema';
```

Coba gunakan user yang telah dibuat untuk melihat data dari sebuah tabel:
```bash
psql -h localhost -d training_db -U andi -p 5432 -c "select * from training_schema.courses;"  
psql -h localhost -d training_db -U budi -p 5432 -c "select * from training_schema.courses;"
```

Periksa status row level security:
```bash
SELECT schemaname, tablename, reloftype, relrowsecurity, relforcerowsecurity  
FROM pg_tables  
JOIN pg_class ON pg_tables.tablename = pg_class.relname  
WHERE schemaname = 'training_schema' AND tablename = 'courses';
```

Hidupkan fitur row level security pada tabel:
```bash
ALTER TABLE training_schema.courses ENABLE ROW LEVEL SECURITY;
```

Buat policy yang mengatur user agar hanya bisa melihat data dengan id sesuai nilai tag yang diberikan:
```bash
CREATE POLICY participant_policy ON training_schema.courses  
FOR SELECT USING (id_course = current_setting('app.current_participant')::INTEGER);
```

Beri tag pada user dengan value sesuai data yang ada pada kolom id:
```bash
alter role andi set app.current_participant = '1';  
alter role budi set app.current_participant = '2';
```

Coba gunakan user untuk melihat isi tabel:
```bash
psql -h localhost -d postgres -U andi -p 5432 -c "select * from training_schema.courses;"  
psql -h localhost -d postgres -U budi -p 5432 -c "select * from training_schema.courses;"
```

Pastikan masing-masing user hanya bisa melihat data yang id-nya sesuai nilai tag yang telah diberikan.

## 2.5.3 Data Encryption
Periksa status ekstensi pgcrypto:
```bash
select * from pg_available_extensions;
```

Periksa dengan cara lain:
```bash
\dx
```

Buat ekstensi pgcrypto aktif dengan perintah berikut:
```bash
CREATE EXTENSION pgcrypto;
```

Buat tabel baru:
```bash
CREATE TABLE users (  
id SERIAL PRIMARY KEY,  
email TEXT NOT NULL UNIQUE,  
password TEXT NOT NULL  
);
```

Masukkan data ke dalam tabel yang telah dibuat dengan enkripsi pada kolom password:
```bash
INSERT INTO users (email, password) VALUES (  
'rafiryd@mail.com',  
crypt('katasandi123', gen_salt('md5'))  
);
```

Periksa data yang telah dimasukkan:
```bash
select * from users;
```

Cara untuk menemukan user dengan password yang sesuai:
```bash
select * from users where password = crypt('katasandi123', password);
```



















































































































