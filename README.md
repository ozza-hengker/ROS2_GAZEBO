# Ubuntu 22.04, ROS 2 Humble, dan Gazebo Installation Guide

Panduan instalasi Ubuntu 22.04 LTS menggunakan **VMware Workstation** maupun **Dual Boot**, dilengkapi dengan instalasi **ROS 2 Humble** dan **Gazebo** untuk kebutuhan praktikum robotika.

---

# Daftar Isi

* Persyaratan Sistem
* Download Ubuntu 22.04
* Metode Instalasi

  * Opsi A — VMware Workstation
  * Opsi B — Dual Boot
* Update Sistem
* Instalasi ROS 2 Humble
* Instalasi Gazebo
* Membuat Workspace ROS 2
* Verifikasi Instalasi
* Troubleshooting
* Referensi

---

# Persyaratan Sistem

| Komponen | Minimum     | Disarankan        |
| -------- | ----------- | ----------------- |
| RAM      | 4 GB        | 8–16 GB           |
| CPU      | 2 Core      | 4 Core atau lebih |
| Storage  | 30 GB       | 50 GB atau lebih  |
| GPU      | Tidak wajib | Mendukung OpenGL  |

> Untuk pengguna VMware, minimal gunakan RAM 8 GB dan 4 CPU Core agar Gazebo serta RViz dapat berjalan dengan lancar.

---

# Download Ubuntu 22.04

Download Ubuntu 22.04 LTS (Jammy Jellyfish):

https://releases.ubuntu.com/jammy/

File yang disarankan:

```text
ubuntu-22.04.5-desktop-amd64.iso
```

---

# Metode Instalasi

Pilih salah satu metode berikut:

* VMware Workstation
* Dual Boot

---

# Opsi A — VMware Workstation

Metode ini menjalankan Ubuntu di dalam Windows tanpa mengubah partisi disk utama.

## Install VMware Workstation

Download dan install VMware Workstation.

---

## Membuat Virtual Machine

1. Buka VMware Workstation.
2. Klik **Create a New Virtual Machine**.
3. Pilih:

```text
Typical (recommended)
```

4. Pilih:

```text
Installer disc image file (iso)
```

5. Masukkan file:

```text
ubuntu-22.04.5-desktop-amd64.iso
```

6. Klik Next.

---

## Konfigurasi Akun Ubuntu

Isi:

```text
Full Name
Username
Password
```

Klik Next.

---

## Konfigurasi Virtual Machine

Nama VM:

```text
Ubuntu 22.04
```

Pilih lokasi penyimpanan sesuai kebutuhan.

---

## Konfigurasi Storage

Atur:

```text
Disk Size : 50 GB
```

Pilih:

```text
Store virtual disk as a single file
```

---

## Konfigurasi Hardware

Klik:

```text
Customize Hardware
```

Atur:

```text
Memory : 8192 MB (8 GB)
Processors : 4 Core
```

Jika spesifikasi laptop terbatas:

```text
Memory : 4096 MB
Processors : 2 Core
```

---

## Pengaturan Display VMware

Masuk ke:

```text
VM Settings → Display
```

Centang:

```text
Accelerate 3D Graphics
```

Disarankan:

```text
Graphics Memory : 2 GB atau Auto
```

Pengaturan ini diperlukan agar Gazebo dapat berjalan dengan baik.

---

## Menjalankan Instalasi Ubuntu

1. Klik **Power On This Virtual Machine**.
2. Pilih:

```text
Install Ubuntu
```

3. Pilih keyboard layout.
4. Pilih:

```text
Normal Installation
```

5. Centang:

```text
Install third-party software
```

6. Pilih:

```text
Erase disk and install Ubuntu
```

> Aman digunakan karena hanya menghapus virtual disk VMware.

7. Tunggu hingga instalasi selesai.
8. Restart VM.

---

## Install VMware Tools

Setelah Ubuntu berhasil masuk desktop:

```bash
sudo apt update
sudo apt install open-vm-tools open-vm-tools-desktop -y
sudo reboot
```

Fitur yang aktif:

* Full Screen
* Auto Resize Display
* Copy Paste
* Drag and Drop
* Shared Folder

---

# Opsi B — Dual Boot

Metode ini menginstall Ubuntu berdampingan dengan Windows.

---

## Persiapan

### Backup Data

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

Gunakan:

* Rufus
* Balena Etcher
* Ventoy

Langkah:

1. Masukkan flashdisk minimal 8 GB.
2. Pilih file ISO Ubuntu.
3. Klik Start.
4. Tunggu hingga selesai.

---

## Instalasi Ubuntu Dual Boot

1. Boot menggunakan flashdisk Ubuntu.
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

untuk partisi manual.

---

### Skema Partisi yang Disarankan

| Mount Point | Ukuran     |
| ----------- | ---------- |
| /           | 40 GB      |
| swap        | 4–8 GB     |
| /home       | Sisa ruang |

4. Lanjutkan instalasi.
5. Restart komputer.
6. Pilih sistem operasi melalui menu GRUB.

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
sudo apt install ros-humble-desktop -y
sudo apt install ros-dev-tools -y
```

---

## Setup Environment ROS 2

```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

---

# Instalasi Gazebo Harmonic

## Menambahkan Repository Gazebo

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

Tambahkan ke bashrc:

```bash
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

---

# Verifikasi Instalasi

## Cek Ubuntu

```bash
lsb_release -a
```

Output:

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

Jika jendela simulator muncul, Gazebo berhasil terpasang.

---

## Cek Workspace

```bash
cd ~/ros2_ws
colcon build
```

Jika tidak ada error, workspace siap digunakan.

---

# Troubleshooting

## ROS 2 Tidak Terdeteksi

Jalankan:

```bash
source /opt/ros/humble/setup.bash
```

Pastikan pada file `.bashrc` terdapat:

```bash
source /opt/ros/humble/setup.bash
```

---

## Gazebo Tidak Muncul pada VMware

Periksa:

```text
VM Settings → Display
```

Pastikan:

```text
Accelerate 3D Graphics ✓
```

dan RAM VM minimal:

```text
8 GB
```

---

# Referensi

Ubuntu 22.04 LTS
https://releases.ubuntu.com/jammy/

ROS 2 Humble Documentation
https://docs.ros.org/en/humble/

Gazebo Documentation
https://gazebosim.org/docs/

---

**Selamat! Ubuntu 22.04, ROS 2 Humble, dan Gazebo telah berhasil diinstal dan siap digunakan untuk praktikum robotika.**
