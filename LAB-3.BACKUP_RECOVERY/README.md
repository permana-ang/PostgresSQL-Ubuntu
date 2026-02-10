# LAB 3. BACKUP & RECOVERY
## 3.1 Logical
### 3.1.1 Backup Logical
### 3.1.2 Restore Logical
## 3.2 Physical
### 3.2.1 Offline Mode
### 3.2.2 Online Mode
## 3.3 Archiving WAL
### 3.3.1 Konfigurasi Archiving WAL


# 3.1 Logical
## 3.1.1 Backup Logical
Coba jalankan perintah backup database berikut:
```bash
# Backup database tertentu  
pg_dump -f training_db.sql -U postgres training_db  
# Backup schema  
pg_dump -n training_schema -f training_schema.sql -U postgres training_db  
# Backup data saja  
pg_dump -a --insert --disable-triggers -f training_data.sql -U postgres training_db  
# Backup tabel tertentu  
pg_dump -t training_schema.courses -f training_courses.sql -U postgres training_db  
# Backup full database dan format dikompres  
pg_dump -Ft -f training_db.tar -U postgres training_db
```

Backup cluster database:
```bash
# Backup full database Cluster  
pg_dumpall > main_cluster.sql
```

3.1.2 Restore Logical
Jalankan perintah berikut untuk restorasi database:
```bash
# buat database dahulu  
createdb -O admin training_db;  
# Restore database training_db saja  
psql -f training_db.sql training_db;  
# Restore semua databases di dalam cluster  
psql -f main_cluster.sql
```

Restore database dari format kompres
```bash
pg_restore -d training_db training_db_full.tar
```

# 3.2 Physical
## 3.2.1 Offline Mode
Jalankan perintah backup mode offline berikut:
```bash
# Backup  
tar -cf main-cluster.tar /var/lib/postgresql/17/main  
# Coba extract dan lihat isinya  
tar --strip-components=4 -xf main-cluster.tar
```

3.2.2 Online Mode
Jalankan perintah backup mode online berikut:
```bash
# Config Archiving WAL files  
vim /etc/postgresql/17/main/postgresql.conf  
```

Konfigurasi parameter berikut:
```bash
# postgresql.conf  
wal_level = replica
````

Lakukan backup:
```bash
# Base Backup Using pg_basebackup Tool  
pg_basebackup -h localhost -D backup_db  
```

Periksa data hasil backup:
```bash
# Verify Backup  
pg_verifybackup backup_db/
```

# 3.3 Archiving WAL
## 3.3.1 Konfigurasi Archiving WAL
Buat folder baru untuk mengarsipkan WAL:
```bash
mkdir -p /backup/wal/main
```

Ubah folder menjadi milik user postgres:
```bash
chown -R postgres:postgres /backup/
```

Konfigurasi postgresql untuk mengarsipkan WAL:
```bash
vim /etc/postgresql/17/main/postgresql.conf
```

Sesuaikan beberapa parameter seperti berikut:
```bash
archive_mode = on  
archive_command = 'cp %p /backup/wal/main/%f'
```

Periksa hasil WAL yang diarsipkan:
```bash
ls /backup/wal/main
```



































