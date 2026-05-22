# Dokumentasi Panduan Update & Konfigurasi Antigravity IDE di Linux (Ubuntu)

Dokumentasi ini dibuat untuk standarisasi proses pembaruan, migrasi, dan konfigurasi lingkungan kerja **Antigravity IDE** pada arsitektur berbasis Linux. Panduan ini memastikan aplikasi berjalan secara steril dengan mematuhi kebijakan keamanan sistem operasi (SUID Sandbox) dan menghindari malafungsi akibat berkas usang (*orphaned files*).

---

## 📌 Kebijakan Penting & Aturan Arsitektur

Sebelum mengeksekusi perintah, pastikan Anda memahami batasan dan aturan berikut untuk mencegah kerusakan sistem atau kegagalan peluncuran aplikasi:

1. **Jangan Melakukan *Overwrite* Langsung (Menumpuk Folder)**
   Menimpa direktori lama dengan berkas baru dari versi teranyar tanpa pembersihan menyeluruh akan menyisakan dependensi usang. Hal ini memicu ketidakstabilan (*memory leak*) atau kegagalan *runtime*. Selalu lakukan *clean install*.
2. **Hindari Spasi pada Nama Direktori**
   Penggunaan spasi seperti `Antigravity IDE (1)` adalah kebiasaan buruk di lingkungan Unix. Spasi memecah argumen pada terminal dan merusak interpretasi *path* otomatis eksekusi skrip.
3. **Standarisasi Lokasi Aplikasi**
   Menyimpan aplikasi *standalone* yang kompleks di dalam direktori `/home/` pengguna akan membatasi fungsionalitas SUID dan menurunkan standar keamanan operasional. Lokasi standar industri untuk aplikasi pihak ketiga adalah `/opt/`.

---

## 🛠️ Alur Kerja Pembaruan Bersih & Instalasi (Clean Update)

Ikuti urutan prioritas eksekusi di bawah ini setiap kali ada pembaruan versi baru (misalnya dari versi 2.0 ke 2.0.2):

### Langkah 1: Pembersihan Direktori Sistem Lama
Hapus total kerangka aplikasi versi lama yang ada di direktori `/opt/` untuk memastikan tidak ada sisa file usang.
bash
sudo rm -rf /opt/antigravity-ide


### Langkah 2: Pemindahan & Standardisasi Berkas Baru
Pindahkan folder hasil unduhan baru yang berada di direktori lokal pengguna ke direktori sistem `/opt/`. Sesuaikan *source path* jika nama folder unduhan Anda berantakan.
bash
sudo mv "/home/$USER/Antigravity IDE (1)/Antigravity IDE" /opt/antigravity-ide


### Langkah 3: Standardisasi Hak Kepemilikan (Ownership)
Karena berkas baru dipindahkan dari folder lokal, kepemilikan berkas masih terikat pada pengguna biasa. Ubah kepemilikannya secara rekursif menjadi milik `root:root`.
bash
sudo chown -R root:root /opt/antigravity-ide


### Langkah 4: Injeksi SUID Bit pada Chrome Sandbox
Ini adalah langkah kritis yang wajib diulang setiap kali aplikasi diperbarui. Mesin berbasis Chromium/Electron membutuhkan hak istimewa tinggi pada modul isolasi keamanannya. Kegagalan mengeksekusi ini akan memicu *fatal error Trace/breakpoint trap (core dumped)*.
bash
sudo chmod 4755 /opt/antigravity-ide/chrome-sandbox


---

## 🖥️ Konfigurasi Shortcut Desktop & System Launcher

Agar aplikasi dapat diakses langsung melalui menu pencarian sistem (*Super/Windows Key*) atau dari layar depan desktop tanpa membuka terminal, buat *Desktop Entry* dengan standar XDG.

### 1. Buat Berkas Konfigurasi Launcher
Jalankan perintah ini untuk membuat berkas konfigurasi di bawah direktori aplikasi lokal pengguna:
bash
nano ~/.local/share/applications/antigravity-ide.desktop


### 2. Injeksi Kode Konfigurasi
Salin dan tempel konfigurasi di bawah ini ke dalam editor tersebut:
ini
[Desktop Entry]
Name=Antigravity IDE
Comment=Advanced Development Environment
Exec=/opt/antigravity-ide/antigravity-ide %F
Icon=code
Terminal=false
Type=Application
Categories=Development;IDE;

*> Catatan: Parameter `%F` berfungsi agar IDE dapat menerima input berkas eksternal langsung dari file manager.*

Simpan dengan menekan `Ctrl+O`, lalu keluar dengan `Ctrl+X`.

### 3. Berikan Hak Akses Eksekusi Launcher
Sistem Linux menolak memproses peluncur aplikasi yang tidak memiliki izin eksekusi eksplisit:
bash
chmod +x ~/.local/share/applications/antigravity-ide.desktop


### 4. Replikasi ke Layar Fisik Desktop (Opsional)
Jika Anda secara estetika membutuhkan ikon berada di layar depan, salin berkas tersebut ke direktori Desktop Anda:
bash
cp ~/.local/share/applications/antigravity-ide.desktop ~/Desktop/
chmod +x ~/Desktop/antigravity-ide.desktop

**🚨 Tindakan Wajib pada GUI:** Kembali ke tampilan Desktop Anda, klik kanan pada file `antigravity-ide.desktop` yang baru muncul, lalu pilih **"Allow Launching"** atau **"Mark as Trusted"** untuk menghilangkan tanda peringatan gembok/keamanan.

---

## 🔍 Pemecahan Masalah (Troubleshooting)

### Error: `The SUID sandbox helper binary was found, but is not configured correctly.`
* **Penyebab:** Berkas `chrome-sandbox` tidak dimiliki oleh `root` atau tidak memiliki hak akses `4755`. Hal ini biasanya terjadi karena Anda melewatkan Langkah 3 dan Langkah 4 setelah melakukan pembaruan berkas aplikasi.
* **Solusi Darurat Bypass (Tidak Direkomendasikan untuk Jangka Panjang):** Jika partisi `/home` Anda terkunci dengan flag `nosuid` dan metode `/opt/` mengalami kendala teknis dari kebijakan kernel, Anda dapat memaksa aplikasi berjalan dengan mematikan fitur sandbox. Ubah baris `Exec` pada file `.desktop` menjadi:
  ini
  Exec=/opt/antigravity-ide/antigravity-ide --no-sandbox %F
  
  *> Peringatan: Menggunakan `--no-sandbox` menghilangkan lapisan isolasi proses aplikasi dan mengekspos lingkungan kerja lokal Anda terhadap potensi eksploitasi jika IDE mengeksekusi modul eksternal berbahaya.*
