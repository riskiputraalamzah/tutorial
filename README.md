# Tutorial Instalasi Docker di Windows Menggunakan WSL 2

Panduan ini akan memandu Anda melalui proses instalasi Docker di Windows dengan memanfaatkan Windows Subsystem for Linux (WSL) dan distro Ubuntu.

---

### **Bagian 1: Instalasi Windows Subsystem for Linux (WSL)**

WSL memungkinkan Anda menjalankan lingkungan Linux secara langsung di Windows tanpa perlu mesin virtual tradisional.

1.  **Buka Command Prompt atau PowerShell sebagai Administrator.**

    - Klik menu Start, ketik `cmd` atau `powershell`.
    - Klik kanan pada hasilnya, lalu pilih **"Run as administrator"**.

2.  **Jalankan Perintah Instalasi WSL.**
    Di jendela yang muncul, ketikkan perintah di bawah ini dan tekan Enter. Perintah ini akan secara otomatis mengunduh dan menginstal semua komponen yang diperlukan.

    ```bash
    wsl --install
    ```

3.  **Restart Komputer Anda.**
    Setelah proses instalasi selesai, Anda akan diminta untuk me-restart komputer Anda. Simpan pekerjaan Anda dan lakukan restart untuk menyelesaikan instalasi.

    > 📝 **Catatan:** Jika perintah di atas tidak berhasil (biasanya pada versi Windows yang lebih lama), Anda mungkin perlu mengaktifkan fitur "Virtual Machine Platform" dan "Windows Subsystem for Linux" secara manual melalui "Turn Windows features on or off". Namun, untuk Windows 10 (build 2004 ke atas) dan Windows 11, perintah `wsl --install` sudah mencakup semuanya.

---

### **Bagian 2: Instalasi Distro Ubuntu**

Setelah komputer restart, proses instalasi Ubuntu akan berlanjut.

1.  **Tunggu Instalasi Ubuntu Selesai.**
    Sebuah jendela terminal Ubuntu akan muncul dan menyelesaikan proses instalasi. Ini mungkin memakan waktu beberapa menit.

2.  **Buat Akun Pengguna Linux.**
    Anda akan diminta untuk membuat _username_ dan _password_ baru.

    - **Penting:** Password ini khusus untuk lingkungan Ubuntu Anda dan tidak harus sama dengan password Windows. Saat Anda mengetik password, tidak ada karakter yang akan muncul.

3.  **Verifikasi Instalasi (Opsional).**
    Untuk memastikan distro Ubuntu sudah terinstal, buka PowerShell atau Command Prompt dan ketik:
    ```bash
    wsl -l -v
    ```
    Perintah ini akan menampilkan daftar distro Linux yang terinstal dan memastikan versinya adalah `2`.

> Jika jendela Ubuntu tidak muncul otomatis, Anda bisa mencarinya di menu Start atau mengunduhnya langsung dari **Microsoft Store**.

---

### **Bagian 3: Instalasi Docker Engine di Ubuntu (WSL)**

Buka terminal Ubuntu Anda dari menu Start untuk memulai instalasi Docker.

1.  **Update Daftar Paket.**
    Jalankan perintah ini untuk memastikan semua paket perangkat lunak Anda adalah yang terbaru.

    ```bash
    sudo apt update && sudo apt upgrade -y
    ```

    Anda akan dimintai password Linux yang sudah Anda buat sebelumnya.

2.  **Instal Paket yang Diperlukan.**
    Instal beberapa paket prasyarat agar Docker bisa ditambahkan dengan benar.

    ```bash
    sudo apt update
    sudo apt install ca-certificates curl gnupg lsb-release -y

    ```

3.  **Tambahkan Kunci GPG Resmi Docker.**
    Ini menambahkan kunci keamanan agar sistem Anda mempercayai repositori Docker.

    ```bash
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

    ```

4.  **Tambahkan Repositori Docker.**
    Tambahkan repositori resmi Docker ke dalam daftar sumber paket sistem Anda.

    ```bash
    echo \
    "deb [arch=$(dpkg --print-architecture) \
    signed-by=/etc/apt/keyrings/docker.gpg] \
    https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

    ```

5.  **Instal Docker Engine.**
    Update lagi daftar paket Anda, lalu instal Docker Engine.

    ```bash
    sudo apt update
    sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

    ```


6.  **Mulai dan Verifikasi Docker.**
    Buka kembali terminal Ubuntu Anda.
    - Mulai layanan Docker:
      ```bash
      sudo service docker start
      ```
    - Verifikasi instalasi dengan menjalankan container "hello-world":
      ```bash
      docker run hello-world
      ```

Jika instalasi berhasil, Anda akan melihat pesan "Hello from Docker!". Ini menandakan Docker Engine telah terinstal dan berjalan dengan benar. Selamat! 🎉
