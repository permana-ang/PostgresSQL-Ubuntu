# LAB 5. REPLICATION & LOAD BALANCING
## 5.1 Streaming Replication
### 5.1.1 Asynchronous
### 5.1.2 Synchronous
## 5.2 Logical Replication
## 5.3 Failover
## 5.4 Load Balancing

# 5.1 Streaming Replication
## 5.1.1 Asynchronous

Lakukan di Server 1

Masuk dengan user postgres pada server linux:
```bash
su - postgres
```

Buat cluster baru:
```bash
initdb -D /var/lib/postgresql/17/mydb
```

Buka konfigurasi cluster baru:
```bash
vim /var/lib/postgresql/17/mydb/postgresql.conf
```

Konfigurasi parameter berikut:
```bash
listen_addresses = 'localhost'  
port = 5401
```

Jalankan cluster:
```bash
pg_ctl -D /var/lib/postgresql/17/mydb/ -l /var/log/postgresql/mydb.log start
```

Remote cluster baru:
```bash
psql -p 5401
```

Atur password user postgres-nya:
```bash
ALTER USER postgres PASSWORD 'postgres';
```

Buat user baru dengan izin akses superuser:
```bash
create role admin login password 'password';  
alter role admin superuser;
```

Buka konfigurasi HBA:
```bash
vim /var/lib/postgresql/17/mydb/pg_hba.conf
```

Izinkan user yang telah dibuat untuk melakukan koneksi secara remote:
```bash
host all admin 0.0.0.0/0 md5
```

Restart cluster setelah konfigurasi:
```bash
pg_ctl -D /var/lib/postgresql/17/mydb/ -l /var/log/postgresql/mydb.log restart
```

Remote cluster baru dengan user yang telah dibuat:
```bash
psql -U admin -d postgres -p 5401 -h localhost
```

Buat database baru:
```bash
create database test_replica;  
\c test_replica;
```

Buat tabel untuk pengujian:
```bash
create table tb_data(id int primary key, name varchar);
```

Masukkan data ke dalam tabel:
```bash
insert into tb_data values(generate_series(1,10),'data'||generate_series(1,10));
```

Periksa isi tabel:
```bash
select * from tb_data;
```

Buka kembali konfigurasi postgresql:
```bash
vim /var/lib/postgresql/17/mydb/postgresql.conf
```

Konfigurasi parameter berikut:
```bash
# config wal replica  
wal_level = replica  
# config archive mode  
archive_mode = on  
archive_command = 'cp %p /backup/wal/mydb/%f'  
# config cluster name
cluster_name = 'psql1'
```

Buat folder baru untuk WAL:
```bash
mkdir -p /backup/wal/mydb
```
Buat user untuk replikasi:
```bash
createuser --replication -P replica -p 5401
```

Buka konfigurasi HBA:
```bash
vim /var/lib/postgresql/17/mydb/pg_hba.conf
```

Tambahkan konfigurasi agar user yang digunakan untuk replikasi bisa melakukan koneksi:
```bash
host replication replica 172.23.0.0/20 md5  
host all replica 172.23.0.0/20 md5
```

Restart cluster:
```bash
pg_ctl -D /var/lib/postgresql/17/mydb/ -l /var/log/postgresql/mydb.log restart
```

Lakukan di Server 2 Lakukan backup server 1 agar disimpan di server 2:
```bash
cd /var/lib/postgresql/17/  
pg_basebackup -R -h 172.23.x.x -U replica --port 5401 -D mydb -P
```

Buka konfigurasi:
```bash
vim /var/lib/postgresql/17/mydb/postgresql.conf
```

Konfigurasi parameter berikut:
```bash
# change listen and port  
listen_addresses = '*'  
port = 5401  
# config wal replica  
wal_level = replica  
# config archive mode  
archive_mode = on  
archive_command = 'cp %p /backup/wal/mydb/%f'  
# config standby  
primary_conninfo = 'user=replica host=172.23.x.x password=password port=5401 application_name=psql2'  
hot_standby = on  
# config cluster name  
cluster_name = 'psql2'
```

### Pada primary_conninfo atur host mengarah ke server 1.

Buat folder untuk archive wal:
```bash
mkdir -p /backup/wal/mydb
```

Jalankan cluster pada server 2:
```bash
pg_ctl -D /var/lib/postgresql/17/mydb/ -l /var/log/postgresql/mydb.log start
```

Lakukan di server 1 Koneksi ke cluster server 1:
```bash
psql -U postgres -h localhost -p 5401 -d test_replica
```

Periksa status replikasi:
```bash
\x
select * from pg_stat_replication;
```

Lakukan di server 2 Koneksi ke cluster server 2:
```bash
psql -U postgres -h localhost -p 5401 -d test_replica
```

Periksa status replikasi:
```bash
\x
select * from pg_stat_wal_receiver;
```

Coba masukkan data melalui cluster pada server 1:
```bash
\x  
insert into tb_data values(generate_series(11,20),'data'||generate_series(11,20));  
insert into tb_data values(generate_series(21,30),'data'||generate_series(21,30));  
insert into tb_data values(generate_series(31,40),'data'||generate_series(31,40));
```

Lakukan di server 1 dan server 2 Lihat hasilnya pada kedua server:
```bash
select * from tb_data;
```

# 5.1.2 Synchronous

Lakukan di server 1 Buka konfigurasi pada cluster di server 1:
```bash
vim /var/lib/postgresql/17/mydb/postgresql.conf
```

Konfigurasi parameter berikut:
```bash
# config sync  
synchronous_commit = on  
synchronous_standby_names = '*'
```

Restart cluster di server 1:
```bash
pg_ctl -D /var/lib/postgresql/17/mydb/ -l /var/log/postgresql/mydb.log restart
```

Lakukan di server 1 Periksa status replikasi di server 1:
```bash
\x
select * from pg_stat_replication;
\x
```

Lakukan di server 2 Periksa status replikasi di server 2:
```bash
\x
select * from pg_stat_wal_receiver;
\x
```

Lakukan di server 1 Coba masukkan data baru melalui server 1:
```bash
# insert data on table  
insert into tb_data values(generate_series(41,50),'data'||generate_series(41,50));  
# show data on table  
select * from tb_data;
```

Periksa data pada kedua cluster:
```bash
# show data on table  
select * from tb_data;
```

# 5.2 Logical Replication
Logical replication adalah jenis replikasi di PostgreSQL pada tingkat tabel. Antar cluster bisa replikasi namun hanya tabel tertentu saja.


Salah satu menjadi PUBLICATION dan yang lain, yang mereplikasi data menjadi SUBSCRIPTION. Untuk cara konfigurasi bisa dilihat pada dokumentasinya.


# 5.3 Failover
Jika terjadi kegagalan server PostgreSQL primary, maka perlu dilakukan failover secara manual (bisa dibuatkan script).

Lakukan di server 1 Simulasikan server 1 mengalami kegagalan cluster/server:

Lakukan di server 1 Simulasikan server 1 mengalami kegagalan cluster/server:

```bash
pg_ctl -D /var/lib/postgresql/17/mydb/ -l /var/log/postgresql/mydb.log stop
```

Lakukan di server 2 Buka konfigurasi di server 2:
```bash
nano /var/lib/postgresql/17/mydb/postgresql.conf
```

Beri komentar pada konfigurasi berikut:
```bash
# primary_conninfo = 'user=replica host=172.23.x.x password=replica port=5401 application_name=psql2'
```
Maksudnya konfigurasi tersebut dimatikan dengan pagar.

Setelah itu hapus file signal agar cluster database bisa menerima proses tulis:
```bash
rm /var/lib/postgresql/17/mydb/standby.signal
```

Jalankan restart cluster untuk menerapkan konfigurasi:
```bash
pg_ctl -D /var/lib/postgresql/17/mydb/ -l /var/log/postgresql/mydb.log restart
```

5.4 Load Balancing

Install load balancer postgreSQL yaitu pgpool2:
```bash
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'  
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -  
sudo apt-get update  
sudo apt-get -y install pgpool2 libpgpool2 postgresql-17-pgpool2
```

Lakukan di server 1 Buat role baru melalui server 1 dan berikan permissions:
```bash
psql -p 5401 -h localhost
```

```bash
CREATE ROLE pgpool WITH LOGIN password 'password';  
GRANT pg_monitor TO pgpool;
\q
```

Buka konfigurasi HBA pada server:
```bash
vim /var/lib/postgresql/17/mydb/pg_hba.conf
```

Beri izin jaringan agar bisa terhubung:
```bash
host all pgpool 172.23.0.0/20 md5
```

Restart cluster database:
```bash
pg_ctl -D /var/lib/postgresql/17/mydb/ restart -l /var/log/postgresql/mydb.log
```

Lakukan di server 2 Buka konfigurasi HBA pada server:
```bash
vim /var/lib/postgresql/17/mydb/pg_hba.conf
```

Beri izin jaringan agar bisa terhubung:
```bash
host all pgpool 172.23.0.0/20 md5
```

Restart cluster database:
```bash
pg_ctl -D /var/lib/postgresql/17/mydb/ restart -l /var/log/postgresql/mydb.log
```

Buka konfigurasi pgpool:
```bash
vim /etc/pgpool2/pgpool.conf
```

Konfigurasi beberapa parameter seperti berikut:
```bash
backend_clustering_mode = 'streaming_replication'  
listen_addresses = '*'  
port = 5400  
backend_hostname0 = '172.23.x.x'  
backend_port0 = 5401  
backend_weight0 = 0  
backend_data_directory0 = '/var/lib/postgresql/17/mydb/'  
backend_application_name0 = 'psql1-nama'  
backend_hostname1 = '172.23.x.x'  
backend_port1 = 5401  
backend_weight1 = 1  
backend_data_directory1 = '/var/lib/postgresql/17/mydb/'  
backend_application_name1 = 'psql2-nama'  
enable_pool_hba = on  
log_statement = on  
log_per_node_statement = on  
sr_check_user = 'replica'  
sr_check_password = 'password'  
health_check_period = 10  
health_check_user = 'pgpool'  
health_check_password = 'password'
```

Buka konfigurasi HBA pada pgpool:
```bash
vim /etc/pgpool2/pool_hba.conf
```

Izinkan jaringan agar bisa terhubung:
```bash
host all pgpool 172.23.0.0/20 md5  
host all replica 172.23.0.0/20 md5  
host all admin 172.23.0.0/20 md5
```

Buka konfigurasi pgpool password:
```bash
vim /etc/pgpool2/pool_passwd
```

Isi user dan password agar bisa terhubung ke pgpool seperti berikut:
```bash
pgpool:password
admin:password
```

Jalankan service pgpool2 dan restart:
```bash
systemctl start pgpool2.service  
systemctl restart pgpool2.service
```

Pastikan service pgpool2 berjalan:
```bash
journalctl -f -u pgpool2.service
```

Periksa hasil konfigurasi:
```bash
psql -h 172.23.x.x -U pgpool --port 5400 -c "show pool_nodes;" postgres  
psql -h 172.23.x.x -U pgpool --port 5400 -c "show pool_processes;" postgres  
psql -h 172.23.x.x -U pgpool --port 5400 -c "show pool_health_check_stats;" postgres
```
Pastikan kedua cluster muncul.

Lakukan uji coba koneksi ke pgpool:
```bash
psql -U admin -d test_replica -p 5400 -h localhost
```

Masukkan data baru ke tabel yang telah dibuat:
```bash
insert into tb_data values(generate_series(61,70),'data'||generate_series(61,70));  
update tb_data set name = 'test2' where id = 61;  
delete from tb_data where id < 65;  
select * from tb_data;
```













































































