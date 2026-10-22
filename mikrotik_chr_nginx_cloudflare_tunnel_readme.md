<p align="center">
  <h1 align="center">🚀 MikroTik CHR Web Server & Cloudflare Tunnel</h1>
  <p align="center"><i>Panduan Praktis Deployment Nginx Container & Secure Tunnel di MikroTik RouterOS v7</i></p>
</p>

---

## 📌 Daftar Isi
- [Arsitektur Sistem](#-arsitektur-sistem)
- [Prasyarat & Konfigurasi Jaringan](#-prasyarat--konfigurasi-jaringan)
- [Bab 1: Deployment Nginx Container](#-bab-1-deployment--konfigurasi-nginx-reverse-proxy)
- [Bab 2: Deployment Cloudflare Tunnel](#-bab-2-deployment--konfigurasi-cloudflare-tunnel)
- [Verifikasi & Pengujian](#-verifikasi--pengujian)

---

## 🏗️ Arsitektur Sistem

```text
[Internet] ──> [Cloudflare Edge] ──(Secure Tunnel)──> [Cloudflared Container (veth-cf)]
                                                               │
                                                       [Nginx Container (veth1)]
```

---

## 📋 Prasyarat & Konfigurasi Jaringan

Sebelum menjalankan kontainer, pastikan fitur **Container** di RouterOS (v7.x) sudah aktif dan skema jaringan lokal telah diatur.

### 1. Aktifkan Fitur Container
```routeros
/system/device-mode/update container=yes
```
> ⚠️ **Catatan:** Lakukan *reboot* pada router setelah menjalankan perintah di atas.

### 2. Konfigurasi Bridge & VETH Interface
```routeros
/interface bridge add name=bridge-container
/interface veth add name=veth1 address=172.17.0.2/24 gateway=172.17.0.1
/interface veth add name=veth-cf address=172.17.0.3/24 gateway=172.17.0.1
/interface bridge port add bridge=bridge-container interface=veth1
/interface bridge port add bridge=bridge-container interface=veth-cf
/ip address add address=172.17.0.1/24 interface=bridge-container
```

### 3. Konfigurasi DNS & SRC NAT (Akses Internet)
```routeros
/ip dns set allow-remote-requests=yes servers=1.1.1.1,8.8.8.8
/ip firewall nat add chain=srcnat src-address=172.17.0.0/24 action=masquerade
```

---

## 🛠️ Bab 1: Deployment & Konfigurasi Nginx (Reverse Proxy)

### 1. Buat Kontainer Nginx
```routeros
/container add remote-image=nginx:alpine interface=veth1 root-dir=container/nginx
```

### 2. Jalankan Kontainer Nginx
```routeros
# Cek ID kontainer
/container print

# Nyalakan kontainer (sesuaikan nomor ID, misal: 0)
/container start 0
```

---

## 🌐 Bab 2: Deployment & Konfigurasi Cloudflare Tunnel

### 1. Buat Tunnel di Cloudflare Zero Trust
1. Buka dashboard **Cloudflare Zero Trust** → **Networks** → **Tunnels**.
2. Klik **Create a Tunnel**, beri nama (misal: `mikrotik-tunnel`), lalu salin **Token Tunnel**.
3. Tambahkan **Public Hostname**:
   - **Domain:** `web.apriliatmand.my.id`
   - **Service:** `HTTP` → `172.17.0.2:80`

### 2. Buat Kontainer Cloudflared di MikroTik
> *Ganti `<TOKEN_CLOUDFLARE_ANDA>` dengan token asli Anda.*
```routeros
/container add remote-image=cloudflare/cloudflared:latest interface=veth-cf root-dir=container/cloudflared cmd="tunnel --no-autoupdate run --token <TOKEN_CLOUDFLARE_ANDA>"
```

### 3. Jalankan Kontainer Cloudflared
```routeros
# Cek ID kontainer
/container print

# Nyalakan kontainer (sesuaikan nomor ID, misal: 1)
/container start 1
```

---

## ✅ Verifikasi & Pengujian

Buka browser Anda dan akses domain publik:
👉 **`https://web.apriliatmand.my.id`**

Jika berhasil, halaman sambutan **Welcome to nginx!** akan muncul dengan aman melalui Cloudflare Tunnel.