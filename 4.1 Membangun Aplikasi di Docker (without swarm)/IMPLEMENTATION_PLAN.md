# Rencana Implementasi: Membangun Aplikasi Microservice dengan Docker Compose

## Tujuan
Membangun dan menjalankan aplikasi microservice menggunakan Docker Compose tanpa Docker Swarm pada satu mesin.

## Prasyarat
- Docker dan Docker Compose terinstall
- Git terinstall
- Minimal 4GB RAM tersedia
- Port 6000 (frontend) dan 5000 (backend) tersedia

## Arsitektur Aplikasi

Aplikasi terdiri dari 5 services:
1. **nginx-frontend**: Web server untuk frontend (Port 6000)
2. **frontend**: Aplikasi Next.js
3. **nginx-backend**: Web server untuk backend (Port 5000)
4. **backend**: Aplikasi Node.js/Express
5. **database**: PostgreSQL database

![Arsitektur Aplikasi](./img/app-arch.png)

### Jaringan
- **web-network**: Jaringan untuk frontend dan nginx-frontend (bridge)
- **api-network**: Jaringan untuk backend dan nginx-backend (bridge)
- **db-network**: Jaringan untuk backend dan database (bridge)

### Volume
- **db**: Volume untuk data PostgreSQL (local)
- **backend**: Volume untuk data backend (local)
- **frontend**: Volume untuk data frontend (local)

## Langkah-Langkah Implementasi

### Fase 1: Persiapan
- [ ] Clone repository
- [ ] Masuk ke direktori `4.1 Membangun Aplikasi di Docker (without swarm)/compose/`
- [ ] Verifikasi semua file dan folder tersedia

### Fase 2: Konfigurasi (Opsional)
- [ ] Konfigurasi nginx-frontend.conf (jika menggunakan domain)
- [ ] Konfigurasi nginx-backend.conf (jika menggunakan domain)
- [ ] Konfigurasi file .env di frontend (jika tidak menggunakan localhost)

### Fase 3: Build Images
- [ ] Jalankan `docker compose build`
- [ ] Verifikasi semua images ter-build dengan `docker images`
- [ ] Pastikan ada 5 images: database, backend, frontend, nginx-frontend, nginx-backend

### Fase 4: Menjalankan Aplikasi
- [ ] Jalankan `docker compose up -d`
- [ ] Tunggu beberapa saat hingga semua container siap
- [ ] Verifikasi status dengan `docker compose ps`
- [ ] Pastikan semua container berstatus "Up"

### Fase 5: Verifikasi
- [ ] Akses aplikasi melalui browser: `http://localhost:6000`
- [ ] Test fitur login/register
- [ ] Verifikasi koneksi ke database
- [ ] Cek logs jika ada error: `docker compose logs`

### Fase 6: Maintenance (Opsional)
- [ ] Monitor logs: `docker compose logs -f`
- [ ] Restart service jika diperlukan: `docker compose restart <service>`
- [ ] Update aplikasi: build ulang dan restart

### Fase 7: Cleanup (Jika Diperlukan)
- [ ] Stop aplikasi: `docker compose down`
- [ ] Hapus volume (hati-hati!): `docker compose down -v`

## Troubleshooting

### Masalah: Port sudah digunakan
**Solusi**: 
- Cek port yang digunakan: `lsof -i :6000` atau `lsof -i :5000`
- Stop service yang menggunakan port tersebut
- Atau ubah port di docker-compose.yaml

### Masalah: Container tidak bisa start
**Solusi**:
- Cek logs: `docker compose logs <service-name>`
- Verifikasi Dockerfile dan konfigurasi
- Pastikan semua dependencies tersedia

### Masalah: Database connection error
**Solusi**:
- Pastikan service database sudah running
- Verifikasi environment variables di backend
- Cek network connectivity: `docker network inspect <network-name>`

### Masalah: Frontend tidak bisa connect ke backend
**Solusi**:
- Verifikasi NEXT_PUBLIC_API_ENDPOINT di .env
- Pastikan nginx-backend sudah running
- Cek logs nginx-backend: `docker compose logs nginx-backend`

## Perbedaan dengan Versi Swarm

| Aspek | Docker Compose (Tanpa Swarm) | Docker Swarm |
|-------|------------------------------|--------------|
| **Node** | 1 mesin | Multiple mesin (manager + worker) |
| **Network** | Bridge network | Overlay network |
| **Volume** | Local volume | NFS volume |
| **Registry** | Tidak diperlukan | Docker Registry diperlukan |
| **Deployment** | `docker compose up` | `docker stack deploy` |
| **Scaling** | Manual (edit compose file) | Otomatis dengan replicas |
| **Load Balancing** | Tidak ada | Built-in load balancing |

## Keuntungan Menggunakan Docker Compose

1. **Sederhana**: Tidak perlu setup multiple node
2. **Cepat**: Tidak perlu setup Docker Registry dan NFS
3. **Cocok untuk Development**: Ideal untuk development dan testing
4. **Mudah di-debug**: Semua container pada satu mesin

## Keterbatasan Docker Compose

1. **Single Machine**: Hanya berjalan pada satu mesin
2. **No Auto-scaling**: Tidak ada auto-scaling seperti Swarm
3. **No High Availability**: Tidak ada failover otomatis
4. **Limited Load Balancing**: Tidak ada built-in load balancing

## Catatan Penting

- Data database disimpan di volume `db`. Jika volume dihapus, data akan hilang.
- Untuk production, pertimbangkan menggunakan Docker Swarm atau Kubernetes
- Backup volume database secara berkala jika data penting
- Monitor resource usage (CPU, Memory, Disk)

