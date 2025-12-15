# Troubleshooting: No Such Image Error di Docker Swarm

## Error yang Terjadi

### Error 1: No Such Image
```
musicapp_db.1   Shutdown   Rejected   "No such image: 192.168.64.19:4000/database:latest"
```

### Error 2: HTTP Response to HTTPS Client
```
Error response from daemon: failed to resolve reference "192.168.64.19:4000/database:latest": 
failed to do request: Head "https://192.168.64.19:4000/v2/database/manifests/latest": 
http: server gave HTTP response to HTTPS client
```

**Penyebab Error 2:** Docker mencoba mengakses registry menggunakan HTTPS, padahal registry menggunakan HTTP. Ini berarti konfigurasi `insecure-registries` tidak diterapkan dengan benar atau Docker daemon belum di-restart setelah konfigurasi.

## Langkah-Langkah Perbaikan

### Langkah 1: Verifikasi Images di Registry (Manager Node)

**Di Manager Node (192.168.64.19):**

```bash
# Cek images yang ada di registry
curl http://192.168.64.19:4000/v2/_catalog | json_pp

# Atau tanpa json_pp
curl http://192.168.64.19:4000/v2/_catalog
```

**Pastikan output menunjukkan semua images:**
- database
- backend
- frontend
- nginx-frontend
- nginx-backend

**Jika images tidak ada, push images terlebih dahulu:**

```bash
# Di Manager Node
docker push 192.168.64.19:4000/database
docker push 192.168.64.19:4000/backend
docker push 192.168.64.19:4000/frontend
docker push 192.168.64.19:4000/nginx-frontend
docker push 192.168.64.19:4000/nginx-backend
```

### Langkah 2: Test Koneksi Registry dari Worker Node

**Di Worker Node (debian2 dan debian3):**

```bash
# Test koneksi HTTP ke registry
curl http://192.168.64.19:4000/v2/_catalog

# Test ping ke manager node
ping -c 3 192.168.64.19

# Test port 4000
telnet 192.168.64.19 4000
# atau
nc -zv 192.168.64.19 4000
```

**Jika curl gagal:**
- Periksa firewall rules di manager node
- Pastikan port 4000 terbuka
- Periksa network connectivity

### Langkah 3: Konfigurasi Insecure Registry di Worker Node

**Di SEMUA Worker Node (debian2 dan debian3):**

**Langkah 3.1: Cek file daemon.json**

```bash
cat /etc/docker/daemon.json
```

**Langkah 3.2: Edit atau buat file daemon.json**

```bash
sudo nano /etc/docker/daemon.json
```

**Pastikan isi file:**

```json
{
  "insecure-registries": ["192.168.64.19:4000"]
}
```

**Jika file tidak ada, buat dengan:**

```bash
sudo tee /etc/docker/daemon.json > /dev/null <<EOF
{
  "insecure-registries": ["192.168.64.19:4000"]
}
EOF
```

**Langkah 3.3: Validasi JSON**

```bash
python3 -m json.tool /etc/docker/daemon.json
```

**Jika ada error, perbaiki syntax JSON.**

**Langkah 3.4: Set permission**

```bash
sudo chmod 644 /etc/docker/daemon.json
```

**Langkah 3.5: Restart Docker daemon (PENTING!)**

```bash
# Restart Docker daemon
sudo systemctl restart docker

# Tunggu beberapa detik
sleep 5

# Verifikasi Docker berjalan
sudo systemctl status docker
```

**⚠️ PENTING:** Docker daemon HARUS di-restart setelah mengubah `daemon.json`. Tanpa restart, konfigurasi tidak akan diterapkan!

**Langkah 3.6: Verifikasi konfigurasi**

```bash
# Cek insecure registries
docker info | grep -i "insecure registries"
```

**Output harus menunjukkan:**
```
Insecure Registries:
 192.168.64.19:4000
```

**Jika tidak muncul, berarti:**
1. File daemon.json tidak benar
2. Docker daemon belum di-restart
3. Ada syntax error di daemon.json

**Cek detail:**
```bash
# Cek isi file
cat /etc/docker/daemon.json

# Validasi JSON
python3 -m json.tool /etc/docker/daemon.json

# Cek log Docker
sudo journalctl -u docker -n 20
```

### Langkah 4: Test Pull Image dari Worker Node

**Di Worker Node (debian2 dan debian3):**

```bash
# Test pull image dari registry
docker pull 192.168.64.19:4000/database

# Jika berhasil, hapus image lokal (opsional)
docker rmi 192.168.64.19:4000/database
```

**Jika pull gagal:**
- Periksa kembali konfigurasi insecure-registries
- Pastikan Docker daemon sudah di-restart
- Cek log Docker: `sudo journalctl -u docker -n 50`

### Langkah 5: Verifikasi Registry Container Berjalan

**Di Manager Node:**

```bash
# Cek registry container
docker ps | grep registry

# Jika tidak berjalan, start registry
docker run -d -p 4000:5000 --restart=always --name registry registry:2

# Verifikasi registry berjalan
curl http://localhost:4000/v2/_catalog
```

### Langkah 6: Deploy Ulang Stack

**Di Manager Node:**

```bash
# Remove stack yang gagal
docker stack rm musicapp

# Tunggu hingga semua services ter-remove
sleep 10

# Verifikasi stack sudah ter-remove
docker stack services musicapp

# Deploy ulang
docker stack deploy -c docker-compose.yaml musicapp

# Monitor deployment
watch docker stack services musicapp
```

**Atau monitor dengan:**

```bash
docker stack ps musicapp
```

### Langkah 7: Verifikasi Service Status

**Di Manager Node:**

```bash
# Cek status semua services
docker stack services musicapp

# Cek detail service yang sebelumnya gagal
docker service ps musicapp_db --no-trunc

# Cek logs (jika service sudah running)
docker service logs musicapp_db
```

## Checklist Perbaikan

- [ ] Images sudah di-push ke registry (Manager Node)
- [ ] Registry dapat diakses dari worker node (curl berhasil)
- [ ] File `/etc/docker/daemon.json` ada di semua worker node
- [ ] `insecure-registries` dikonfigurasi dengan benar
- [ ] Docker daemon sudah di-restart di semua worker node
- [ ] `docker info` menunjukkan insecure registries
- [ ] Test pull image berhasil dari worker node
- [ ] Registry container berjalan di manager node
- [ ] Stack sudah di-deploy ulang

## Troubleshooting Tambahan

### Jika Masih Gagal Setelah Semua Langkah

**1. Cek Firewall di Manager Node:**

```bash
# Untuk UFW
sudo ufw allow 4000/tcp
sudo ufw status

# Untuk firewalld
sudo firewall-cmd --add-port=4000/tcp --permanent
sudo firewall-cmd --reload
```

**2. Cek Network Connectivity:**

```bash
# Di Worker Node
ping 192.168.64.19
traceroute 192.168.64.19
```

**3. Cek Docker Logs:**

```bash
# Di Worker Node
sudo journalctl -u docker -n 100 | grep -i registry
sudo journalctl -u docker -n 100 | grep -i "no such image"
```

**4. Cek Service Events:**

```bash
# Di Manager Node
docker service ps musicapp_db --no-trunc
docker service inspect musicapp_db
```

## Quick Fix Script

Jalankan script berikut di **SEMUA Worker Node**:

```bash
#!/bin/bash

# Konfigurasi
REGISTRY_IP="192.168.64.19"
REGISTRY_PORT="4000"

echo "=== Fixing Docker Registry Configuration ==="

# Backup file lama (jika ada)
if [ -f /etc/docker/daemon.json ]; then
    echo "Backing up existing daemon.json..."
    sudo cp /etc/docker/daemon.json /etc/docker/daemon.json.backup.$(date +%Y%m%d_%H%M%S)
fi

# Buat/update daemon.json
echo "Creating/updating daemon.json..."
sudo tee /etc/docker/daemon.json > /dev/null <<EOF
{
  "insecure-registries": ["${REGISTRY_IP}:${REGISTRY_PORT}"]
}
EOF

# Set permission
sudo chmod 644 /etc/docker/daemon.json

# Validasi JSON
echo "Validating JSON..."
if python3 -m json.tool /etc/docker/daemon.json > /dev/null 2>&1; then
    echo "✓ JSON is valid"
else
    echo "✗ JSON validation failed!"
    exit 1
fi

# Restart Docker
echo "Restarting Docker daemon..."
sudo systemctl restart docker

# Tunggu Docker ready
echo "Waiting for Docker to be ready..."
sleep 5

# Verifikasi Docker berjalan
if sudo systemctl is-active --quiet docker; then
    echo "✓ Docker is running"
else
    echo "✗ Docker failed to start!"
    sudo systemctl status docker
    exit 1
fi

# Verifikasi konfigurasi
echo "Checking insecure registries..."
INSECURE_REG=$(docker info 2>/dev/null | grep -i "insecure registries" -A 1 | grep "${REGISTRY_IP}:${REGISTRY_PORT}")
if [ -n "$INSECURE_REG" ]; then
    echo "✓ Insecure registry configured: ${REGISTRY_IP}:${REGISTRY_PORT}"
else
    echo "✗ Insecure registry NOT configured!"
    echo "Current docker info:"
    docker info | grep -i "insecure registries" -A 2
    exit 1
fi

# Test pull
echo "Testing pull from registry..."
if docker pull ${REGISTRY_IP}:${REGISTRY_PORT}/database > /dev/null 2>&1; then
    echo "✓ Successfully pulled image from registry"
    docker rmi ${REGISTRY_IP}:${REGISTRY_PORT}/database > /dev/null 2>&1
else
    echo "✗ Failed to pull image from registry"
    echo "Trying again with verbose output..."
    docker pull ${REGISTRY_IP}:${REGISTRY_PORT}/database
    exit 1
fi

echo ""
echo "=== Configuration Complete ==="
echo "Registry: ${REGISTRY_IP}:${REGISTRY_PORT}"
echo "Status: Ready"
```

Simpan sebagai `fix-registry.sh`, lalu jalankan:

```bash
chmod +x fix-registry.sh
sudo ./fix-registry.sh
```

## Troubleshooting Error "HTTP Response to HTTPS Client"

Jika Anda mendapatkan error:
```
http: server gave HTTP response to HTTPS client
```

**Ini berarti Docker masih mencoba HTTPS ke registry. Solusinya:**

### 1. Pastikan daemon.json Benar

```bash
# Di Worker Node
cat /etc/docker/daemon.json
```

**Harus berisi:**
```json
{
  "insecure-registries": ["192.168.64.19:4000"]
}
```

**JANGAN ada:**
- Trailing comma
- Single quotes
- Format yang salah

### 2. Restart Docker Daemon

```bash
# Restart Docker
sudo systemctl restart docker

# Tunggu beberapa detik
sleep 5

# Verifikasi Docker berjalan
sudo systemctl status docker
```

### 3. Verifikasi Konfigurasi Aktif

```bash
# Cek insecure registries
docker info | grep -i "insecure registries" -A 2
```

**Jika tidak muncul, berarti:**
- Docker daemon belum di-restart
- File daemon.json tidak benar
- Ada error di log Docker

### 4. Cek Log Docker

```bash
# Cek log untuk error
sudo journalctl -u docker -n 50 | grep -i error
sudo journalctl -u docker -n 50 | grep -i registry
```

### 5. Test Pull Manual

```bash
# Test pull dengan verbose
docker pull 192.168.64.19:4000/database

# Jika masih error HTTPS, berarti konfigurasi belum aktif
```

### 6. Force Restart Docker

Jika masih tidak bekerja:

```bash
# Stop Docker
sudo systemctl stop docker

# Tunggu beberapa detik
sleep 3

# Start Docker
sudo systemctl start docker

# Verifikasi
sudo systemctl status docker
docker info | grep -i "insecure registries"
```

## Catatan Penting

1. **Insecure Registry harus dikonfigurasi di SEMUA worker node**
2. **Docker daemon HARUS di-restart setelah mengubah daemon.json**
3. **Pastikan registry container berjalan di manager node**
4. **Images harus di-push ke registry sebelum deploy**
5. **Test pull dari worker node untuk memastikan konfigurasi benar**

