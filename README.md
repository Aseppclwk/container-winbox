# 🚀 Panduan Lengkap Praktikum: Mengulik Fitur Container di MikroTik CHR (RouterOS v7)

Repositori ini berisi panduan komprehensif untuk persiapan Virtual Machine (VM), penanganan kendala sistem (*troubleshooting*), hingga praktik langsung menjalankan Docker Container (Nginx Web Server) di dalam MikroTik Cloud Hosted Router (CHR) v7 via WinBox.

---

## 1. Prasyarat & Persiapan VM
Sebelum mulai mengulik container, pastikan VM MikroTik CHR Anda memiliki spesifikasi yang memadai:
* **Virtualizer:** VMware Workstation / VirtualBox / Proxmox (Pastikan fitur virtualisasi BIOS/SVM/VT-x sudah *Enabled* di komputer host).
* **RAM:** Minimal **1 GB (1024 MB)**.
* **vCPU:** Cukup **1 atau 2 vCPU**.
* **Hard Disk:** Minimal **2 GB – 4 GB** (Gunakan tipe SCSI/SATA, **jangan** menggunakan disk kosong berukuran 128 MB atau tipe IDE agar tidak cepat penuh dan lemot). Pastikan file image VMDK CHR resmi sudah terpasang dengan benar sebagai sistem utama.

---

## 2. Konfigurasi Global Container di WinBox
1. Masuk ke menu **Container** -> Pilih tab **Config**.
2. Sesuaikan parameter berikut:
   * **Registry URL:** `https://registry-1.docker.io`
   * **Layer Dir:** Pilih disk/direktori penyimpanan router yang lega.
   * **Memory High:** `256 MiB` (sesuaikan dengan kapasitas RAM CHR).
3. Klik **Apply** lalu **OK**.

---

## 3. Membuat Jaringan (VETH & IP Address)
VETH digunakan sebagai jembatan komunikasi antara RouterOS dan container.

### A. Membuat Interface VETH
1. Buka menu **Interfaces** -> Pilih tab **VETH** -> Klik tombol **`+` (Add)**.
2. Atur parameter:
   * **Name:** `veth1`
   * **Address:** `172.17.0.2/24`
   * **Gateway:** `172.17.0.1`
3. Klik **Apply** lalu **OK**.

### B. Menambahkan IP Address Gateway ke Router
1. Masuk ke menu **IP** -> **Addresses** -> Klik tombol **`+` (Add)**.
2. Masukkan **Address:** `172.17.0.1/24`.
3. Pilih **Interface:** `veth1`.
4. Klik **Apply** lalu **OK**.

---

## 4. Menjalankan Container Nginx
1. Masuk ke menu **Container** -> Pilih tab **Containers** -> Klik tombol **`+` (Add)**.
2. Masukkan konfigurasi berikut:
   * **Remote Image:** `nginx:alpine`
   * **Interface:** `veth1`
   * **Root Dir:** Pilih folder disk penyimpanan container.
3. Klik **Apply**. Router akan otomatis melakukan *pulling* (mengunduh) image dari Docker Hub.
4. Klik tombol **Start** hingga statusnya berubah menjadi **running**.

---

## 5. Pengujian (Testing)
Buka browser di laptop/PC Anda, kemudian akses alamat IP container:
```text
[http://172.17.0.2](http://172.17.0.2)
