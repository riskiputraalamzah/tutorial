# Install Docker Engine on Windows (WSL + Ubuntu) — Step-by-step

Dokumentasi singkat, jelas, dan langsung praktik — untuk kondisi: **Windows belum punya WSL, belum ada distro Ubuntu, belum ada Docker**.

> **Ringkasan:** kita akan mengaktifkan WSL2, memasang Ubuntu (distro WSL), memastikan `systemd` tersedia, lalu meng-install **Docker Engine (Docker CE)** langsung di dalam Ubuntu WSL.

---

## Sebelum mulai — catatan penting

- Backup file penting dari Windows/WSL sebelum mencoba perubahan besar.
- Proses ini **tidak** menginstal Docker Desktop; ini menginstal _Docker Engine_ di dalam WSL/Ubuntu.
- Jika sebelumnya pernah menginstal **Docker Desktop**, sebaiknya uninstall Docker Desktop dulu untuk menghindari konflik (opsi dan langkah ada di bagian Troubleshooting).

---

## 1) Persiapan Windows & cek requirements

1. Buka PowerShell **sebagai Administrator**.
2. Pastikan Virtualization (VT-x/AMD-V) di-enable di BIOS/UEFI. (Biasanya muncul di _Performance/Virtualization_ pada BIOS.)
3. Update WSL ke versi terbaru store-backed (disarankan):

```powershell
wsl --update
```

4. (Optional) Cek versi WSL:

```powershell
wsl --version
```

Jika perintah tidak dikenali, lanjutkan ke langkah instalasi WSL di bawah.

---

## 2) Install WSL + Ubuntu (cara cepat)

**Rekomendasi (Windows 10/11 modern):** jalankan di PowerShell (Admin):

```powershell
wsl --install -d Ubuntu
```

Perintah di atas akan:

- Mengaktifkan fitur WSL dan Virtual Machine Platform jika belum aktif.
- Mengunduh dan menginstal WSL kernel dan distro Ubuntu terbaru.
- Mengatur WSL 2 sebagai default distro baru.

> Jika kamu ingin hanya mengaktifkan fitur WSL tanpa menginstal distro, gunakan:

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

Jika perintah `wsl` tidak dikenali (WSL belum terpasang) atau kamu menggunakan versi Windows yang belum menyediakan `wsl --install`, ikuti langkah singkat ini di PowerShell (jalankan sebagai Administrator):

```powershell
# 1) Aktifkan fitur Windows yang diperlukan
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# 2) Restart Windows untuk menerapkan perubahan (jika diminta)
# Setelah restart, buka PowerShell (Admin) lagi dan jalankan:

# 3) Update WSL dan set default ke WSL2
wsl --update
wsl --set-default-version 2

# 4) Install Ubuntu (bisa lewat Microsoft Store atau dengan perintah jika tersedia):
wsl --install -d Ubuntu
```

Catatan: jika `wsl --install` masih tidak tersedia setelah langkah di atas, buka Microsoft Store lalu cari dan install "Ubuntu" secara manual, lalu jalankan distro tersebut sekali untuk menyelesaikan konfigurasi awal (username & password).

Setelah `wsl --install -d Ubuntu` selesai, kamu bisa langsung masuk ke distro Ubuntu dengan menjalankan:

```powershell
wsl -d Ubuntu
```

Atau, jika lebih nyaman, kamu tetap bisa membuka aplikasi **Ubuntu** dari Start Menu atau menjalankan `wsl` di PowerShell untuk konfigurasi awal (buat username & password Linux).

### Skrip bantu (opsional)

Ada skrip PowerShell helper di `scripts/install-wsl-ubuntu.ps1` yang menggunakan `wsl.exe --install` dan menyediakan opsi untuk langsung memasuki Ubuntu setelah instalasi.

Contoh penggunaan (PowerShell sebagai Administrator):

```powershell
# Jalankan interaktif (akan menampilkan instruksi jika perlu restart)
.\scripts\install-wsl-ubuntu.ps1

# Coba otomatis masuk ke Ubuntu setelah install (jika memungkinkan)
.\scripts\install-wsl-ubuntu.ps1 -AutoEnter
```

Catatan: skrip ini memakai `wsl.exe --install` seperti contoh perintah di atas. Jika Windows-mu belum menyediakan `wsl.exe`, skrip akan mengaktifkan fitur yang diperlukan dan meminta restart.

---

## 3) Update Ubuntu & pastikan systemd (penting untuk `systemctl`)

1. Buka Ubuntu (WSL). Update paket:

```bash
sudo apt update && sudo apt upgrade -y
```

2. Cek apakah systemd aktif (systemctl tersedia):

```bash
systemctl --version
```

3. Jika tidak aktif, aktifkan systemd (panduan ringkas):

- Pastikan WSL versi store >= 0.67.6 (`wsl --version`).
- Edit file `/etc/wsl.conf` dan tambahkan ini:

```boot
systemd=true
```

- Simpan lalu dari PowerShell (Admin) jalankan:

```powershell
wsl --shutdown
```

- Buka kembali Ubuntu; `systemd` harusnya aktif sekarang.

> Catatan: versi Ubuntu terbaru yang diinstal via `wsl --install` biasanya sudah menggunakan systemd secara default.

---

## 4) Install Docker Engine di Ubuntu (perintah persis)

Jalankan perintah berikut **di dalam** terminal Ubuntu (WSL):

```bash
# 1) Uninstall paket Docker yang mungkin konflik (opsional tapi disarankan)
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove -y $pkg || true; done

# 2) Install dependensi dan siapkan keyrings
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# 3) Tambahkan repository Docker
echo \"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo \"${UBUNTU_CODENAME:-$VERSION_CODENAME}\") stable\" |
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 4) Install Docker Engine
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

> Setelah instalasi selesai, Docker service biasanya akan berjalan otomatis.

---

## 5) Verifikasi & jalankan test container

1. Cek status Docker:

```bash
sudo systemctl status docker
```

2. Jalankan test image:

```bash
sudo docker run hello-world
```

Jika pesan `Hello from Docker!` muncul, instalasi berhasil.

---

## 6) Menjalankan Docker tanpa `sudo` (opsional)

Agar user Linuxmu bisa menjalankan `docker` tanpa `sudo`:

```bash
sudo usermod -aG docker $USER
# Terapkan perubahan grup (logout/login distro WSL diperlukan). Kamu bisa tutup jendela Ubuntu lalu buka kembali, atau jalankan:
newgrp docker

# lalu tes:
docker run hello-world
```

> Jika `newgrp docker` tidak cukup, logout dari distro (tutup terminal) lalu buka lagi.

---

## 7) Tips: Auto-start dan behavior di WSL

- Dengan systemd aktif, kamu bisa enable Docker agar otomatis start ketika distro WSL start:

```bash
sudo systemctl enable docker --now
```

- Perlu diingat: WSL tidak akan tetap berjalan selamanya — Windows akan menghentikan WSL saat tidak digunakan. Systemd di WSL akan menjalankan services saat distro aktif.

---

## 8) Troubleshooting singkat

- **Cannot connect to the Docker daemon** → cek `sudo systemctl start docker` lalu `sudo systemctl status docker`.
- **Izin (permission)** → pastikan user berada di grup `docker` dan sudah restart session WSL.
- **Konflik dengan Docker Desktop** → jika sebelumnya punya Docker Desktop, sebaiknya uninstall Docker Desktop di Windows sebelum pakai Docker Engine murni. Hapus symlink yang mungkin tersisa di WSL (contoh):

```bash
sudo rm -f /usr/bin/docker-compose /usr/bin/docker-credential-desktop.exe /usr/bin/hub-tool
```

- **Service tidak berjalan lewat systemctl** → pastikan `systemd=true` ada di `/etc/wsl.conf` dan kamu sudah `wsl --shutdown` lalu buka ulang distro.

---

## 9) Uninstall Docker Engine (kalau perlu)

Jalankan di Ubuntu WSL:

```bash
sudo apt-get purge -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
sudo rm -rf /var/lib/docker /var/lib/containerd
```

---

## 10) Ringkasan singkat & rekomendasi

- Gunakan **WSL2 + Ubuntu** sebagai host untuk Docker Engine di Windows (tanpa Docker Desktop).
- Ikuti langkah pemasangan Docker Engine resmi (apt repository) agar mudah update via `apt`.
- Aktifkan `systemd` di WSL agar `systemctl` dan service management bekerja seperti di Linux asli.

---

Kalau mau, aku bisa:

- Buatin script **PowerShell** satu baris yang jalankan `wsl --install -d Ubuntu` dan setup awal, atau
- Buatin skrip bash untuk di-run di Ubuntu yang otomatis meng-install Docker Engine (sesuai langkah 4).

Katakan mau yang mana 😉
