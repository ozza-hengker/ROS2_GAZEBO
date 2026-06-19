# Ubuntu 22.04 + ROS 2 Humble + Gazebo Installation Guide

Panduan instalasi Ubuntu 22.04 LTS (Jammy Jellyfish), ROS 2 Humble, dan Gazebo untuk kebutuhan pengembangan robotika.

## Requirements

- Minimal RAM 8 GB (disarankan 16 GB)
- Storage minimal 50 GB
- Prosesor 64-bit (Intel/AMD)
- Koneksi internet

---

# 1. Download Ubuntu 22.04

Unduh Ubuntu 22.04 LTS (Jammy Jellyfish) melalui link berikut:

https://releases.ubuntu.com/jammy/

Pilih:

- **ubuntu-22.04.5-desktop-amd64.iso** (Recommended)

Ubuntu 22.04 merupakan sistem operasi yang didukung secara resmi oleh ROS 2 Humble. :contentReference[oaicite:0]{index=0}

---

# 2. Membuat Bootable USB

Gunakan salah satu software berikut:

- Rufus (Windows)
- Balena Etcher
- Ventoy

Langkah umum:

1. Colok flashdisk minimal 8 GB.
2. Buka Rufus / Etcher.
3. Pilih file ISO Ubuntu 22.04.
4. Klik **Start**.
5. Tunggu hingga proses selesai.

---

# 3. Install Ubuntu 22.04

1. Boot komputer menggunakan flashdisk.
2. Pilih **Install Ubuntu**.
3. Pilih keyboard layout.
4. Pilih **Normal Installation**.
5. Pilih:
   - Install third-party software
6. Atur partisi sesuai kebutuhan.
7. Buat username dan password.
8. Klik **Install Now**.
9. Tunggu hingga instalasi selesai.
10. Restart komputer.

---

# 4. Update Sistem

Setelah masuk Ubuntu:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt autoremove -y
```

---

# 5. Install ROS 2 Humble

ROS 2 Humble merupakan distribusi ROS 2 yang secara resmi mendukung Ubuntu 22.04 Jammy. :contentReference[oaicite:1]{index=1}

## Tambahkan Repository ROS 2

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

## Verifikasi Instalasi ROS 2

Terminal 1:

```bash
ros2 run demo_nodes_cpp talker
```

Terminal 2:

```bash
ros2 run demo_nodes_py listener
```

Jika listener menerima data dari talker, maka ROS 2 berhasil terpasang. :contentReference[oaicite:2]{index=2}

---

# 6. Install Gazebo Harmonic

Gazebo Harmonic mendukung Ubuntu 22.04 secara resmi. :contentReference[oaicite:3]{index=3}

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

## Verifikasi Instalasi Gazebo

Jalankan:

```bash
gz sim
```

Jika jendela simulator Gazebo muncul, maka instalasi berhasil. :contentReference[oaicite:4]{index=4}

---

# 7. Membuat Workspace ROS 2

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

# Struktur Workspace

```text
ros2_ws/
├── src/
├── build/
├── install/
└── log/
```

---

# Troubleshooting

## Cek Versi ROS

```bash
printenv | grep ROS
```

## Cek ROS 2

```bash
ros2 --help
```

## Cek Gazebo

```bash
gz sim --versions
```

---

# Referensi

- Ubuntu 22.04 LTS: https://releases.ubuntu.com/jammy/
- ROS 2 Humble Documentation: https://docs.ros.org/en/humble/
- Gazebo Harmonic Documentation: https://gazebosim.org/docs/harmonic/
