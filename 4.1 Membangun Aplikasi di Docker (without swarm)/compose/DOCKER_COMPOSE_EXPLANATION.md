# Penjelasan docker-compose.yaml

Dokumen ini menjelaskan konfigurasi file `docker-compose.yaml` untuk aplikasi microservice yang menggunakan Docker Compose tanpa Docker Swarm.

## Struktur Umum

File `docker-compose.yaml` menggunakan format YAML. File ini mendefinisikan 5 services, 3 networks, dan 3 volumes.

> **Catatan**: Docker Compose v2 tidak lagi memerlukan atribut `version` di file docker-compose.yaml. Atribut tersebut sudah obsolete dan akan diabaikan.

## Services

### 1. Database Service (`db`)

```yaml
db:
  container_name: db
  build:
    context: ./database
    dockerfile: Dockerfile
  restart: always
  environment:
    POSTGRES_USER: docker
    POSTGRES_PASSWORD: docker
    POSTGRES_DB: musicapp
  volumes:
    - db:/var/lib/postgresql/data
  networks:
    - db-network
```

**Penjelasan:**
- **container_name**: Nama container yang akan dibuat (opsional, jika tidak ada Docker akan generate nama otomatis)
- **build**: Konfigurasi untuk membangun image dari Dockerfile
  - `context`: Direktori yang berisi Dockerfile dan file yang diperlukan
  - `dockerfile`: Nama file Dockerfile
- **restart**: Kebijakan restart container (`always` = restart otomatis jika container berhenti)
- **environment**: Variabel lingkungan untuk PostgreSQL
  - `POSTGRES_USER`: Username untuk database
  - `POSTGRES_PASSWORD`: Password untuk database
  - `POSTGRES_DB`: Nama database yang akan dibuat
- **volumes**: Mount volume untuk persistensi data
  - `db:/var/lib/postgresql/data`: Volume bernama `db` di-mount ke `/var/lib/postgresql/data` di dalam container
- **networks**: Jaringan yang digunakan oleh container
  - `db-network`: Container terhubung ke jaringan `db-network`

### 2. Backend Service (`backend`)

```yaml
backend:
  container_name: backend
  environment:
    # server configuration
    HOST: 0.0.0.0
    PORT: 5000
    # database configuretion
    PGHOST: db
    PGPORT: 5432
    PGUSER: docker
    PGPASSWORD: docker
    PGDATABASE: musicapp
    # JWT token
    ACCESS_TOKEN_KEY: ...
    REFRESH_TOKEN_KEY: ...
    ACCESS_TOKEN_AGE: 1800
  build:
    context: ./backend
    dockerfile: Dockerfile
  depends_on:
    - db
  volumes:
    - 'backend:/usr/src/backend'
  networks:
    - db-network
    - api-network
```

**Penjelasan:**
- **environment**: Variabel lingkungan untuk aplikasi backend
  - `HOST: 0.0.0.0`: Bind ke semua interface network
  - `PORT: 5000`: Port yang digunakan oleh aplikasi backend
  - `PGHOST: db`: Host database (menggunakan nama service `db`)
  - `PGPORT`, `PGUSER`, `PGPASSWORD`, `PGDATABASE`: Konfigurasi koneksi database
  - `ACCESS_TOKEN_KEY`, `REFRESH_TOKEN_KEY`: Kunci untuk JWT token
  - `ACCESS_TOKEN_AGE`: Usia token dalam detik (1800 = 30 menit)
- **depends_on**: Dependency service - backend akan menunggu `db` siap sebelum start
- **networks**: Terhubung ke 2 jaringan:
  - `db-network`: Untuk komunikasi dengan database
  - `api-network`: Untuk komunikasi dengan nginx-backend

### 3. Frontend Service (`frontend`)

```yaml
frontend:
  container_name: frontend
  build:
    context: ./frontend
    dockerfile: Dockerfile
  depends_on:
    - backend
  volumes:
    - 'frontend:/usr/src/frontend'
  networks:
    - web-network
```

**Penjelasan:**
- **depends_on**: Frontend menunggu `backend` siap sebelum start
- **networks**: Terhubung ke `web-network` untuk komunikasi dengan nginx-frontend
- **volumes**: Volume untuk data frontend

### 4. Nginx Frontend Service (`nginx-frontend`)

```yaml
nginx-frontend:
  container_name: nginx-frontend
  depends_on:
    - frontend
  ports:
    - '80:80'
  build:
    context: ./nginx
    dockerfile: Dockerfile-frontend
  networks:
    - web-network
```

**Penjelasan:**
- **ports**: Port mapping dari host ke container
  - `'80:80'`: Port 80 di host di-mapping ke port 80 di container
- **depends_on**: Nginx-frontend menunggu `frontend` siap
- **networks**: Terhubung ke `web-network` untuk komunikasi dengan frontend

### 5. Nginx Backend Service (`nginx-backend`)

```yaml
nginx-backend:
  container_name: nginx-backend
  depends_on:
    - backend
  ports:
    - '8080:8080'
  build:
    context: ./nginx
    dockerfile: Dockerfile-backend
  networks:
    - web-network
    - api-network
```

**Penjelasan:**
- **ports**: Port mapping dari host ke container
  - `'8080:8080'`: Port 8080 di host di-mapping ke port 8080 di container
- **networks**: Terhubung ke 2 jaringan:
  - `web-network`: Untuk komunikasi dengan frontend (jika diperlukan)
  - `api-network`: Untuk komunikasi dengan backend

## Networks

```yaml
networks:
  db-network:
    driver: bridge
  web-network:
    driver: bridge
  api-network:
    driver: bridge
```

**Penjelasan:**
- **driver: bridge**: Menggunakan bridge network driver
  - Bridge network adalah network default Docker yang memungkinkan container pada host yang sama untuk berkomunikasi
  - Tidak seperti overlay network (untuk Swarm), bridge network hanya bekerja pada satu host
  - Container dalam network yang sama dapat saling berkomunikasi menggunakan nama service

**Pemisahan Network:**
- **db-network**: Hanya untuk komunikasi antara backend dan database (isolasi keamanan)
- **api-network**: Untuk komunikasi antara nginx-backend dan backend
- **web-network**: Untuk komunikasi antara frontend, nginx-frontend, dan nginx-backend

## Volumes

```yaml
volumes:
  db:
    driver: local
  backend:
    driver: local
  frontend:
    driver: local
```

**Penjelasan:**
- **driver: local**: Menggunakan local driver untuk menyimpan data di host machine
- **db**: Volume untuk data PostgreSQL (persistensi data database)
- **backend**: Volume untuk data backend (opsional, untuk development)
- **frontend**: Volume untuk data frontend (opsional, untuk development)

**Lokasi Volume:**
Volume local biasanya disimpan di:
- Linux: `/var/lib/docker/volumes/<volume-name>/_data`
- macOS/Windows: Di dalam Docker VM

**Catatan Penting:**
- Data di volume akan tetap ada meskipun container dihapus
- Untuk menghapus volume, gunakan: `docker compose down -v`
- Backup volume secara berkala untuk data penting

## Alur Komunikasi

1. **Client → Nginx Frontend (Port 80)**
   - Client mengakses aplikasi melalui browser pada port 80
   - Request diterima oleh nginx-frontend

2. **Nginx Frontend → Frontend**
   - Nginx-frontend mem-forward request ke container frontend
   - Komunikasi melalui `web-network`

3. **Frontend → Nginx Backend (Port 8080)**
   - Jika frontend perlu data dari backend, request dikirim ke nginx-backend pada port 8080
   - Komunikasi melalui `web-network` atau `api-network`

4. **Nginx Backend → Backend**
   - Nginx-backend mem-forward request ke container backend
   - Komunikasi melalui `api-network`

5. **Backend → Database**
   - Backend melakukan query ke database
   - Komunikasi melalui `db-network`

## Perbedaan dengan Versi Swarm

| Aspek | Docker Compose (File ini) | Docker Swarm |
|-------|---------------------------|--------------|
| **Network Driver** | `bridge` | `overlay` |
| **Volume Driver** | `local` | `local` dengan NFS |
| **Deploy Section** | Tidak ada | Ada (replicas, placement, restart_policy) |
| **Container Name** | Opsional, bisa di-set | Tidak bisa di-set (auto-generated) |
| **Image Registry** | Build lokal | Push ke registry |
| **Scaling** | Manual (edit file) | Otomatis dengan replicas |

## Command yang Sering Digunakan

```bash
# Build semua images
docker compose build

# Build image tertentu
docker compose build <service-name>

# Menjalankan semua services
docker compose up -d

# Menjalankan service tertentu
docker compose up -d <service-name>

# Melihat status services
docker compose ps

# Melihat logs
docker compose logs
docker compose logs -f <service-name>

# Stop services
docker compose down

# Stop dan hapus volumes
docker compose down -v

# Restart service
docker compose restart <service-name>

# Scale service (jika diperlukan)
docker compose up -d --scale backend=2
```

## Troubleshooting

### Container tidak bisa start
- Cek logs: `docker compose logs <service-name>`
- Verifikasi Dockerfile dan dependencies
- Pastikan port tidak digunakan: `lsof -i :80` atau `lsof -i :8080`

### Database connection error
- Pastikan service `db` sudah running: `docker compose ps`
- Verifikasi environment variables di backend
- Cek network: `docker network inspect <network-name>`

### Network tidak terhubung
- Verifikasi semua service dalam network yang sama
- Gunakan nama service (bukan IP) untuk komunikasi
- Cek dengan: `docker network inspect <network-name>`

### Volume tidak persist
- Verifikasi volume dibuat: `docker volume ls`
- Cek mount point: `docker volume inspect <volume-name>`
- Pastikan tidak menggunakan `docker compose down -v` jika ingin mempertahankan data

