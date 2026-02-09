# LAB 1. SETUP POSTGRESQL

## 1.1 Instalasi
## 1.2 Akses Databases
## 1.3 Setting Environment Variable


## 1.1 Instalasi

Jalankan perintah berikut sebagai user **root**:

```bash
sudo apt install curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
. /etc/os-release
sudo sh -c "echo 'deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $VERSION_CODENAME-pgdg main' > /etc/apt/sources.list.d/pgdg.list"
sudo apt update
```

### Instalasi PostgreSQL

```bash
sudo apt -y install postgresql-17
````

## Verifikasi setelah instalasi:

```bash
systemctl status postgresql
````

## Lokasi data directory:

```bash
ls /var/lib/postgresql/17/main/
````

## Lokasi file konfigurasi:

```bash
ls /etc/postgresql/17/main/
````

## Lokasi file log:

```bash
tail -f /var/log/postgresql/postgresql-17-main.log
````





