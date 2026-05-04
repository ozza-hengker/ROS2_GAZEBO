# 🌊 Pertemuan 3: ROS 2 & Gazebo Harmonic Simulation

**Divisi Programming Motion | OPREC Learning Week 2026**

Pada pertemuan ini, kita mulai masuk ke inti sistem robotika modern: **ROS 2 Jazzy** sebagai middleware komunikasi robot, dan **Gazebo Harmonic (Gz Sim)** sebagai simulator berbasis fisika untuk Ubuntu 24.04.

---

## 🎯 Tujuan Pembelajaran

* Memahami arsitektur komunikasi robot menggunakan ROS 2 Jazzy
* Mengenal Gazebo Harmonic (Gz Sim) sebagai simulator 3D
* Mengimplementasikan robot differential drive dengan plugin terbaru
* Membuat robot yang mampu mengejar target secara otomatis

---

## 🛠️ Instalasi ROS 2 Jazzy & Gazebo Harmonic

Ikuti langkah berikut untuk melakukan instalasi:

```bash
# 1. Setup Locale
sudo apt update && sudo apt install locales -y
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

# 2. Tambahkan Repository ROS 2
sudo apt install software-properties-common -y
sudo add-apt-repository universe -y
sudo apt update && sudo apt install curl -y

sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
-o /usr/share/keyrings/ros-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
http://packages.ros.org/ros2/ubuntu \
$(. /etc/os-release && echo $UBUNTU_CODENAME) main" | \
sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

# 3. Install ROS 2 Jazzy + Gazebo Harmonic
sudo apt update
sudo apt install ros-jazzy-desktop ros-jazzy-ros-gz -y

# 4. Setup Environment
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

---

## 🚀 PROJECT: THE CHASER BOT

Membangun robot beroda (Differential Drive) yang mampu mengejar target secara otomatis di Gazebo Harmonic.

---

## 📁 Langkah 1: Setup Workspace

```bash
mkdir -p ~/chaser_ws/src
cd ~/chaser_ws/src

ros2 pkg create chaser_bot \
--build-type ament_python \
--dependencies rclpy geometry_msgs ros_gz_interfaces
```

---

## 🤖 Langkah 2: Desain Robot (URDF)

Masuk ke folder package:

```bash
cd ~/chaser_ws/src/chaser_bot/chaser_bot
```

Buat file `chaser.urdf`:

```xml
<?xml version="1.0" ?>
<robot name="chaser_bot">

  <!-- Body -->
  <link name="base_link">
    <visual>
      <geometry><box size="0.4 0.2 0.1"/></geometry>
      <material name="blue">
        <color rgba="0 0.5 1 1"/>
      </material>
    </visual>
    <collision>
      <geometry><box size="0.4 0.2 0.1"/></geometry>
    </collision>
  </link>

  <!-- Wheels (simplified) -->
  <!-- Tambahkan left_wheel & right_wheel + joint sesuai kebutuhan -->

  <!-- Plugin Gazebo Harmonic -->
  <gazebo>
    <plugin filename="gz-sim-diff-drive-system"
            name="gz::sim::systems::DiffDrive">
      <left_joint>left_wheel_joint</left_joint>
      <right_joint>right_wheel_joint</right_joint>
      <wheel_separation>0.2</wheel_separation>
      <wheel_radius>0.08</wheel_radius>
      <topic>cmd_vel</topic>
    </plugin>
  </gazebo>

</robot>
```

---

## 🧠 Langkah 3: Node Python (Chaser Logic)

Pada ROS 2 Jazzy, komunikasi dengan Gazebo dilakukan melalui **bridge**.

Node Python akan:

* Subscribe posisi objek dari Gazebo
* Menghitung arah target
* Publish ke `/cmd_vel`

*(Implementasi node dapat disesuaikan dari versi sebelumnya / Gazebo Classic)*

---

## ⚙️ Langkah 4: Build Project

```bash
cd ~/chaser_ws
colcon build
source install/setup.bash
```

---

## 🎮 Langkah 5: Simulasi (ROS 2 Jazzy + Gz Sim)

### 1. Jalankan Gazebo Harmonic

```bash
gz sim empty.sdf
```

---

### 2. Spawn Robot

```bash
ros2 run ros_gz_sim create \
-file ~/chaser_ws/src/chaser_bot/chaser_bot/chaser.urdf \
-name chaser_bot \
-z 0.5
```

---

### 3. Aktifkan Bridge

```bash
ros2 run ros_gz_bridge parameter_bridge \
/cmd_vel@geometry_msgs/msg/Twist@gz.msgs.Twist
```

---

## 💡 Perubahan Penting di ROS 2 Jazzy

| Komponen     | Versi Lama     | Versi Baru               |
| ------------ | -------------- | ------------------------ |
| Paket Gazebo | gazebo_ros     | ros_gz                   |
| Simulator    | Gazebo Classic | Gazebo Harmonic (gz sim) |
| Komunikasi   | Langsung       | Via `ros_gz_bridge`      |

---

## 🏁 Hasil Akhir

Robot akan:

* Menerima perintah kecepatan dari ROS 2
* Bergerak di Gazebo Harmonic
* Siap dikembangkan menjadi sistem autonomous

---

## 🎉 Kesimpulan

Pada pertemuan ini, kita berhasil:

* Menginstall ROS 2 Jazzy
* Menggunakan Gazebo Harmonic (Gz Sim)
* Memahami integrasi ROS ↔ Gazebo melalui bridge
* Membangun dasar sistem robot otonom

Ini menjadi fondasi penting untuk pengembangan robotika tingkat lanjut, termasuk simulasi dan implementasi di dunia nyata.

---
