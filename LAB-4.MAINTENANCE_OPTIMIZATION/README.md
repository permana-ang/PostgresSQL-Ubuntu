# LAB 4. MAINTENANCE & OPTIMIZATION
## 4.1 Dashboard
## 4.2 Vacuum
## 4.3 Databases Optimization
## 4.4 Index


# 4.1 Dashboard
Install pgAdmin pada windows/macOS/linux melalui link berikut:

https://www.pgadmin.org/download/

Setelah berhasil didownload, pasang aplikasi tersebut dan lakukan koneksi ke postgreSQL.

# 4.2 Vacuum

Buat database baru untuk lab pengujian:
```bash
CREATE DATABASE test_vacuum;  
\c test_vacuum
```

Buat tabel di dalam database:
```bash
CREATE TABLE data_table (id SERIAL PRIMARY KEY, data TEXT);
```

Masukkan data ke dalam tabel:
```bash
INSERT INTO data_table (data)  
SELECT md5(random()::text)  
FROM generate_series(1, 10000000);
```

Lihat informasi dari tabel yang dibuat:
```bash
SELECT relname AS table_name,  
n_live_tup AS live_tuples,  
n_dead_tup AS dead_tuples,  
last_vacuum,  
last_autovacuum,  
last_analyze,  
last_autoanalyze  
FROM pg_stat_user_tables  
WHERE relname = 'data_table';
```

Lihat ukuran tabel:
```bash
SELECT pg_size_pretty(pg_total_relation_size('data_table'));
```

Lihat ukuran bloat dari tabel:
```bash
SELECT relname AS table_name,  
pg_size_pretty(pg_total_relation_size(relid)) AS total_size,  
pg_size_pretty(pg_relation_size(relid)) AS table_size,  
pg_size_pretty(pg_total_relation_size(relid) -  
pg_relation_size(relid)) AS bloat  
FROM pg_catalog.pg_statio_user_tables  
ORDER BY pg_total_relation_size(relid) DESC;
```

Jalankan query dengan kondisi tertentu:
```bash
EXPLAIN ANALYZE SELECT * FROM data_table WHERE id = 5000000;
```

Hapus sebagian data dari tabel:
```bash
DELETE FROM data_table WHERE id % 5 = 0;
```

Coba jalankan vacuum untuk membersihkan data yang tidak terpakai:
```bash
VACUUM data_table;  
VACUUM FULL data_table;
```

Lihat kembali ukuran tabel:
```bash
SELECT pg_size_pretty(pg_total_relation_size('data_table'));
```

# 4.3 Databases Optimization
Ada banyak hal yang bisa dilakukan untuk mengoptimalkan database, diantaranya:

Index (Kunci Utama)

Query plan & Analyze

Efisiensi Query

Hindari subquery, pakai join

Limit untuk pagination

Hindari select *

Gunakan where

Autovacuum

Partitioning

Connection Poolin (pgBouncer)

Caching Aplikasi (Redis, Memcached)

Materialized View (view tapi disimpan, tidak query ulang)


# 4.4 Index
Kunci paling sederhana dari optimasi database adalah Indexing. Ada beberapa jenis index yang bisa dibuat, yaitu:

BTREE (default)

GIN (pencarian array, json, full text)

BRIN (data besar, berurutan, time series)

HASH (pencarian persis, nomor telepon/email/username)


Buat tabel untuk lab pengujian:
```bash
CREATE TABLE customers (
id SERIAL PRIMARY KEY,
name TEXT,
email TEXT,
city TEXT
);
```

Masukkan data ke dalam tabel:
```bash
INSERT INTO customers (name, email, city)
SELECT
'Name_' || i,
'user_' || i || '@example.com',
CASE
WHEN i % 5 = 0 THEN 'Jakarta'
WHEN i % 5 = 1 THEN 'Bandung'
WHEN i % 5 = 2 THEN 'Surabaya'
WHEN i % 5 = 3 THEN 'Medan'
ELSE 'Makassar'
END
FROM generate_series(1, 1000000) AS s(i);
```

Tampilkan data dengan kondisi tertentu dan analisis proses query:
```bash
SELECT * FROM customers WHERE city = 'Bandung';
EXPLAIN ANALYZE SELECT * FROM customers WHERE city = 'Bandung';
```

Buat index untuk tabel tersebut:
```bash
CREATE INDEX idx_customers_city ON customers(city);
```

Tampilkan kembali data yang sama dan analisis proses query:
```bash
SELECT * FROM customers WHERE city = 'Bandung';
EXPLAIN ANALYZE SELECT * FROM customers WHERE city = 'Bandung';
```






