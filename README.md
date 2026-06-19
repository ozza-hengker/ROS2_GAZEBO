# Ubuntu 22.04, ROS 2 Humble, dan Gazebo Installation Guide

Panduan instalasi Ubuntu 22.04 LTS menggunakan **Virtual Machine (VM)** maupun **Dual Boot**, dilengkapi dengan instalasi **ROS 2 Humble** dan **Gazebo** untuk kebutuhan praktikum robotika.

---

## Daftar Isi

- [Persyaratan Sistem](#persyaratan-sistem)
- [Download Ubuntu 22.04](#download-ubuntu-2204)
- [Metode Instalasi](#metode-instalasi)
  - [Virtual Machine](#opsi-a--virtual-machine)
  - [Dual Boot](#opsi-b--dual-boot)
- [Update Sistem](#update-sistem)
- [Instalasi ROS 2 Humble](#instalasi-ros-2-humble)
- [Instalasi Gazebo](#instalasi-gazebo)
- [Membuat Workspace ROS 2](#membuat-workspace-ros-2)
- [Verifikasi Instalasi](#verifikasi-instalasi)
- [Troubleshooting](#troubleshooting)
- [Referensi](#referensi)

---

# Persyaratan Sistem

| Komponen | Minimum | Disarankan |
|-----------|----------|------------|
| RAM | 4 GB | 8–16 GB |
| CPU | 2 Core | 4 Core atau lebih |
| Storage | 30 GB | 50 GB atau lebih |
| GPU | Tidak wajib | Mendukung OpenGL |

> **Catatan:** Gazebo membutuhkan resource yang cukup besar. Jika memungkinkan, gunakan Dual Boot untuk performa yang lebih baik.

---

# Download Ubuntu 22.04

Download Ubuntu 22.04 LTS (Jammy Jellyfish) melalui link berikut:

https://releases.ubuntu.com/jammy/

File yang disarankan:

```text
ubuntu-22.04.5-desktop-amd64.iso
```

---

# Metode Instalasi

Pilih salah satu metode berikut:

- Virtual Machine (VM)
- Dual Boot

---

# Opsi A — Virtual Machine

Metode ini menjalankan Ubuntu di dalam Windows tanpa mengubah partisi disk.

## Install VirtualBox

Download dan install VirtualBox dari:

https://www.virtualbox.org/

---

## Membuat Virtual Machine

1. Buka VirtualBox.
2. Klik **New**.
3. Isi konfigurasi:

```text
Name      : Ubuntu 22.04
Type      : Linux
Version   : Ubuntu (64-bit)
```

4. Alokasikan resource:

```text
RAM       : Minimal 4096 MB
Disarankan: 8192 MB

CPU       : Minimal 2 Core
Disarankan: 4 Core
```

5. Buat Virtual Hard Disk:

```text
Format      : VDI
Storage     : Dynamically Allocated
Ukuran      : Minimal 30 GB
Disarankan  : 50 GB
```

6. Buka menu **Settings → Storage**.
7. Masukkan file ISO Ubuntu yang telah diunduh.
8. Jalankan VM.

---

## Pengaturan Display VirtualBox

Sebelum menginstal Ubuntu:

Masuk ke:

```text
Settings → Display
```

Atur menjadi:

```text
Video Memory          : 128 MB
Graphics Controller   : VMSVGA
Enable 3D Acceleration: ✓
```

Pengaturan ini diperlukan agar Gazebo dapat berjalan dengan baik.

---

## Install Ubuntu di VM

1. Jalankan VM.
2. Pilih **Install Ubuntu**.
3. Pilih keyboard layout.
4. Pilih:

```text
Normal Installation
```

5. Centang:

```text
Install third-party software
```

6. Klik **Erase disk and install Ubuntu**.

> Opsi ini hanya menghapus disk virtual, bukan disk komputer utama.

7. Tunggu hingga instalasi selesai.
8. Restart VM.

---

# Opsi B — Dual Boot

Metode ini menginstall Ubuntu berdampingan dengan Windows.

---

## Persiapan

### Backup Data Penting

Pastikan seluruh data penting telah dibackup.

---

### Disable Fast Startup

Masuk ke:

```text
Control Panel
→ Power Options
→ Choose what the power buttons do
→ Turn off Fast Startup
```

---

### Membuat Ruang Kosong

1. Tekan:

```text
Windows + X
```

2. Pilih:

```text
Disk Management
```

3. Klik kanan drive yang ingin dikurangi.
4. Pilih:

```text
Shrink Volume
```

5. Kurangi minimal:

```text
50 GB (51200 MB)
```

6. Akan muncul:

```text
Unallocated Space
```

---

## Membuat Bootable USB

Gunakan salah satu software:

- Rufus
- Balena Etcher
- Ventoy

Langkah:

1. Masukkan flashdisk minimal 8 GB.
2. Pilih file ISO Ubuntu.
3. Klik **Start**.
4. Tunggu hingga selesai.

---

## Install Ubuntu Dual Boot

1. Boot komputer menggunakan flashdisk Ubuntu.
2. Pilih:

```text
Install Ubuntu
```

3. Pada Installation Type pilih:

```text
Install Ubuntu alongside Windows Boot Manager
```

atau

```text
Something Else
```

untuk pengaturan partisi manual.

---

### Skema Partisi yang Disarankan

| Mount Point | Ukuran |
|------------|---------|
| / | 40 GB |
| swap | 4–8 GB |
| /home | Sisa ruang |

4. Lanjutkan proses instalasi.
5. Restart komputer.
6. Setelah restart akan muncul menu GRUB untuk memilih:

```text
Ubuntu
Windows
```

---

# Update Sistem

Setelah berhasil masuk Ubuntu:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt autoremove -y
```

---

# Instalasi ROS 2 Humble

## Menambahkan Repository ROS 2

```bash
sudo apt update
sudo apt install software-properties-common curl -y
sudo add-apt-repository universe
```

```bash
export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F'"' '{print $4}')
```

```bash
curl -L -o /tmp/ros2-apt-source.deb \
"https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo ${UBUNTU_CODENAME:-${VERSION_CODENAME}})_all.deb"
```

```bash
sudo dpkg -i /tmp/ros2-apt-source.deb
```

---

## Install ROS 2 Desktop

```bash
sudo apt update
sudo apt upgrade -y
```

```bash
sudo apt install ros-humble-desktop -y
```

```bash
sudo apt install ros-dev-tools -y
```

---

## Setup Environment ROS 2

Tambahkan ke `.bashrc`:

```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
```

```bash
source ~/.bashrc
```

---

# Instalasi Gazebo Harmonic

## Tambahkan Repository Gazebo

```bash
sudo apt update
sudo apt install curl lsb-release gnupg -y
```

```bash
sudo curl https://packages.osrfoundation.org/gazebo.gpg \
--output /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg
```

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] https://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/gazebo-stable.list > /dev/null
```

```bash
sudo apt update
```

---

## Install Gazebo

```bash
sudo apt install gz-harmonic -y
```

---

# Membuat Workspace ROS 2

```bash
mkdir -p ~/ros2_ws/src
```

```bash
cd ~/ros2_ws
```

```bash
colcon build
```

Tambahkan ke `.bashrc`:

```bash
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc
```

```bash
source ~/.bashrc
```

---

# Verifikasi Instalasi

## Cek Versi Ubuntu

```bash
lsb_release -a
```

Output yang diharapkan:

```text
Ubuntu 22.04 LTS
```

---

## Cek ROS 2

Terminal 1:

```bash
ros2 run demo_nodes_cpp talker
```

Terminal 2:

```bash
ros2 run demo_nodes_py listener
```

Jika listener menerima data dari talker, ROS 2 berhasil terpasang.

---

## Cek Gazebo

```bash
gz sim
```

Jika jendela simulator Gazebo muncul, maka Gazebo berhasil terinstall.

---

## Cek Workspace ROS 2

```bash
cd ~/ros2_ws
```

```bash
colcon build
```

Jika tidak muncul error, workspace siap digunakan.

---

# Troubleshooting

## ROS 2 Tidak Terdeteksi

Periksa:

```bash
source /opt/ros/humble/setup.bash
```

atau:

```bash
cat ~/.bashrc
```

Pastikan terdapat:

```bash
source /opt/ros/humble/setup.bash
```

---

## Gazebo Tidak Muncul

Pastikan:

### Pada VirtualBox

```text
Video Memory          : 128 MB
Enable 3D Acceleration: ✓
Graphics Controller   : VMSVGA
```

### Update Driver Grafis

```bash
sudo ubuntu-drivers autoinstall
```

---

# Referensi

- Ubuntu 22.04 LTS  
  https://releases.ubuntu.com/jammy/

- ROS 2 Humble Documentation  
  https://docs.ros.org/en/humble/

- Gazebo Documentation  
  https://gazebosim.org/docs/

---

**Selamat! Ubuntu 22.04, ROS 2 Humble, dan Gazebo telah berhasil diinstal dan siap digunakan untuk praktikum robotika.**
