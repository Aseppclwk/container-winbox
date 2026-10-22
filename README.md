Markdown
# 🚀 MikroTik CHR Web Server with Cloudflare Tunnel

Repositori ini berisi panduan dan konfigurasi lengkap untuk menjalankan web server berbasis **Nginx** di dalam kontainer **MikroTik CHR**, yang terhubung secara publik dan aman menggunakan **Cloudflare Tunnel**.

---

## 📋 Prasyarat: Mengaktifkan Fitur Container & Jaringan

Sebelum membuat kontainer, pastikan fitur **Container** di RouterOS (versi 7.x) sudah aktif dan konfigurasi dasarnya telah disiapkan.

### 1. Aktifkan Fitur Container via Terminal WinBox
```routeros
/system/device-mode/update container=yes
Catatan: Lakukan Reboot pada MikroTik jika diminta oleh sistem.

2. Konfigurasi Interface VETH & Bridge Khusus Kontainer
Cuplikan kode
/interface bridge add name=bridge-container
/interface veth add name=veth1 address=172.17.0.2/24 gateway=172.17.0.1
/interface veth add name=veth-cf address=172.17.0.3/24 gateway=172.17.0.1
/interface bridge port add bridge=bridge-container interface=veth1
/interface bridge port add bridge=bridge-container interface=veth-cf
/ip address add address=172.17.0.1/24 interface=bridge-container
3. Konfigurasi DNS & SRC NAT (Akses Internet Kontainer)
Agar kontainer dapat mengunduh image dari Docker Hub dan terhubung ke internet, tambahkan DNS Resolver dan aturan SRC NAT (Masquerading):

Cuplikan kode
# Set DNS Resolver MikroTik
/ip dns set allow-remote-requests=yes servers=1.1.1.1,8.8.8.8

# Tambahkan Aturan Masquerade NAT untuk Subnet Kontainer
/ip firewall nat add chain=srcnat src-address=172.17.0.0/24 action=masquerade
🛠️ BAB 1: Deployment & Konfigurasi Nginx (Reverse Proxy)
Pada bab ini, kita akan membuat dan menjalankan kontainer Nginx sebagai web server di MikroTik.

1. Buat Container Nginx
Jalankan perintah berikut di Terminal MikroTik:

Cuplikan kode
/container add remote-image=nginx:alpine interface=veth1 root-dir=container/nginx
2. Jalankan Container Nginx
Cek daftar kontainer untuk melihat nomor ID-nya:

Cuplikan kode
/container print
Jalankan kontainer Nginx (ganti 0 dengan ID kontainer Nginx Anda):

Cuplikan kode
/container start 0
Pastikan status kontainer berubah menjadi R (Running).

🌐 BAB 2: Deployment & Konfigurasi Cloudflare Tunnel
Pada bab ini, kita akan membuat jalur tunnel aman dari Cloudflare Zero Trust ke kontainer Nginx tanpa perlu membuka port IP Publik secara langsung.

1. Buat Tunnel di Dashboard Cloudflare
Buka Cloudflare Zero Trust → Networks → Tunnels.

Klik Create a Tunnel, beri nama tunnel (misal: mikrotik-tunnel), lalu simpan.

Salin Token Tunnel yang diberikan oleh Cloudflare.

Pada bagian Public Hostname:

Subdomain/Domain: Masukkan domain Anda (contoh: web.apriliatmand.my.id).

Service: Pilih HTTP dan masukkan IP VETH Nginx: 172.17.0.2:80.

2. Deploy Container Cloudflared di MikroTik
Jalankan perintah ini di Terminal MikroTik (ganti <TOKEN_CLOUDFLARE_ANDA> dengan token yang disalin):

Cuplikan kode
/container add remote-image=cloudflare/cloudflared:latest interface=veth-cf root-dir=container/cloudflared cmd="tunnel --no-autoupdate run --token <TOKEN_CLOUDFLARE_ANDA>"
3. Jalankan Container Cloudflared
Cek ID kontainer cloudflared:

Cuplikan kode
/container print
Jalankan kontainer Cloudflared (ganti 1 dengan ID kontainer Cloudflared Anda):

Cuplikan kode
/container start 1
Pastikan status kontainer R (Running) dan di dashboard Cloudflare status tunnel berubah menjadi HEALTHY.

✅ Verifikasi & Pengujian
Buka browser dan akses domain publik Anda:
👉 https://web.apriliatmand.my.id

Jika konfigurasi berhasil, halaman Welcome to nginx! akan tampil dengan koneksi HTTPS yang aman melalui Cloudflare Tunnel.
