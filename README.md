# ROS2 & Gazebo Simulation (Ubuntu 24.04)

## Learning Week - Programming Motion

Panduan ini dibuat untuk anak-anak Learning Week supaya bisa:

* Install ROS2 di Ubuntu 24.04
* Install Gazebo Harmonic
* Menjalankan simulasi sederhana
* Memahami konsep dasar ROS2
* Menghubungkan ROS2 dengan Gazebo

Karena hidup manusia ternyata belum cukup rumit tanpa dependency error.

---

# 1. Persiapan

## Minimum Requirement

| Komponen       | Minimum            |
| -------------- | ------------------ |
| RAM            | 8 GB               |
| Storage kosong | 25 GB              |
| OS             | Ubuntu 24.04       |
| Processor      | Intel i5 / Ryzen 5 |

Disarankan:

* menggunakan SSD
* menggunakan Ubuntu native (dual boot)
* bukan VirtualBox kentang yang tiap buka terminal kipas laptop langsung teriak.

---

# 2. Update Ubuntu

Buka terminal lalu jalankan:

```bash
sudo apt update && sudo apt upgrade -y
```

Install package dasar:

```bash
sudo apt install curl gnupg lsb-release software-properties-common -y
```

---

# 3. Install ROS2 Jazzy

Ubuntu 24.04 cocok menggunakan:

* ROS2 Jazzy Jalisco

## Tambahkan ROS2 Repository

```bash
sudo apt update
sudo apt install software-properties-common -y
sudo add-apt-repository universe
```

Tambahkan ROS key:

```bash
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
-o /usr/share/keyrings/ros-archive-keyring.gpg
```

Tambahkan repository:

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

Update package:

```bash
sudo apt update
```

---

# 4. Install ROS2 Desktop

```bash
sudo apt install ros-jazzy-desktop -y
```

Install tools tambahan:

```bash
sudo apt install ros-dev-tools -y
```

---

# 5. Setup Environment ROS2

Tambahkan source ROS2 ke bash:

```bash
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
```

Reload terminal:

```bash
source ~/.bashrc
```

Cek apakah ROS2 berhasil:

```bash
ros2
```

Kalau muncul command list panjang berarti berhasil.
Kalau error, selamat datang di dunia robotics.

---

# 6. Test ROS2

## Terminal 1

Jalankan:

```bash
ros2 run demo_nodes_cpp talker
```

## Terminal 2

Jalankan:

```bash
ros2 run demo_nodes_py listener
```

Kalau berhasil:

* talker mengirim pesan
* listener menerima pesan

Ini konsep:

* Publisher = pengirim data
* Subscriber = penerima data
* Topic = jalur komunikasi

---

# 7. Penjelasan Konsep ROS2

## Node

Node adalah program kecil dalam sistem robot.

Contoh:

* camera node
* imu node
* movement node
* sensor node

Robot modern bukan satu program besar.
Tapi kumpulan node yang saling komunikasi.

---

## Topic

Topic adalah jalur komunikasi antar node.

Contoh:

```text
Camera Node --> /camera/image --> Vision Node
```

Node publish data ke topic.
Node lain subscribe topic.

---

## Publisher

Publisher mengirim data.

Contoh:

* sensor publish data IMU
* kamera publish gambar

---

## Subscriber

Subscriber menerima data.

Contoh:

* vision menerima gambar kamera
* controller menerima data sensor

---

## Service

Service adalah komunikasi request-response.

Contoh:

```text
Client meminta data
Server mengirim jawaban
```

Mirip seperti:

* “tolong reset sensor”
* “berapa status baterai?”

---

# 8. Install Gazebo Harmonic

Gazebo Harmonic adalah simulator resmi untuk ROS2 Jazzy.

Pada Ubuntu 24.04 biasanya package `gz-harmonic` tidak langsung ditemukan karena repository Gazebo belum ditambahkan.

Jalankan dependency awal:

```bash
sudo apt update
sudo apt install curl lsb-release gnupg -y
```

Tambahkan key Gazebo:

```bash
sudo curl -fsSL https://packages.osrfoundation.org/gazebo.gpg \
-o /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg
```

Tambahkan repository Gazebo:

```bash
echo "deb [signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] \
http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/gazebo-stable.list > /dev/null
```

Update package:

```bash
sudo apt update
```

Install Gazebo Harmonic:

```bash
sudo apt install gz-harmonic -y
```

Install bridge ROS2 dan Gazebo:

```bash
sudo apt install ros-jazzy-ros-gz -y
```

Kalau muncul error:

```text
E: Unable to locate package gz-harmonic
```

berarti repository Gazebo belum berhasil ditambahkan.

Karena dependency robotics memang punya hobi membuat mahasiswa mempertanyakan pilihan hidupnya.

---

# 9. Menjalankan Gazebo

Jalankan:

```bash
gz sim
```

Kalau berhasil akan muncul:

* world kosong
* simulator Gazebo

Kadang loading lama.
Karena simulator robot memang suka menguji kesabaran manusia.

---

# 10. Menambahkan Object di Gazebo

Di Gazebo:

1. Klik ikon “+”
2. Tambahkan:

   * box
   * sphere
   * cylinder

Object akan muncul di world.

---

# 11. Menjalankan World Bawaan

Contoh:

```bash
gz sim shapes.sdf
```

atau:

```bash
gz sim empty.sdf
```

---

# 12. Membuat Workspace ROS2

Masuk home:

```bash
cd ~
```

Buat workspace:

```bash
mkdir -p ~/ros2_ws/src
```

Masuk workspace:

```bash
cd ~/ros2_ws
```

Build workspace:

```bash
colcon build
```

Source workspace:

```bash
source install/setup.bash
```

Tambahkan otomatis ke bashrc:

```bash
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc
```

---

# 13. Install Colcon

Kalau command colcon tidak ditemukan:

```bash
sudo apt install python3-colcon-common-extensions -y
```

Coba lagi:

```bash
colcon build
```

---

# 14. Membuat Package ROS2 Python

Masuk src:

```bash
cd ~/ros2_ws/src
```

Buat package:

```bash
ros2 pkg create --build-type ament_python my_robot_pkg
```

Masuk package:

```bash
cd my_robot_pkg
```

---

# 15. Membuat Publisher Sederhana

Masuk folder:

```bash
cd ~/ros2_ws/src/my_robot_pkg/my_robot_pkg
```

Buat file:

```bash
nano publisher.py
```

Isi:

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class SimplePublisher(Node):

    def __init__(self):
        super().__init__('simple_publisher')
        self.publisher_ = self.create_publisher(String, 'chatter', 10)
        self.timer = self.create_timer(1.0, self.publish_message)

    def publish_message(self):
        msg = String()
        msg.data = 'Hello ROS2'
        self.publisher_.publish(msg)
        self.get_logger().info(msg.data)


def main(args=None):
    rclpy.init(args=args)
    node = SimplePublisher()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

---

# 16. Build Package

Kembali ke workspace:

```bash
cd ~/ros2_ws
```

Build:

```bash
colcon build
```

Source:

```bash
source install/setup.bash
```

---

# 17. Menjalankan Publisher

```bash
ros2 run my_robot_pkg publisher
```

Kalau belum bisa:

* cek setup.py
* cek permission file
* cek typo

Biasanya eror di setup.py, tambahin di bagian bawah nya tuh jadi gini

```bash
entry_points={
        'console_scripts': [
            'publisher = my_robot_pkg.publisher:main',

```

# 18. Integrasi ROS2 dengan Gazebo

Jalankan Gazebo:

```bash
gz sim shapes.sdf
```

Lalu cek topic ROS2:

```bash
ros2 topic list
```

Cek data topic:

```bash
ros2 topic echo /clock
```


Kalau muncul data waktu berarti:

* Gazebo berhasil komunikasi dengan ROS2

Kalau ga muncul mungkin coba paste ini buat komunikasi mereka

```bash
ros2 run ros_gz_bridge parameter_bridge /clock@rosgraph_msgs/msg/Clock[gz.msgs.Clock
```
---

# 19. Visualisasi Graph ROS2

Install rqt:

```bash
sudo apt install ros-jazzy-rqt-graph -y
```

Jalankan:

```bash
rqt_graph
```

Akan muncul diagram:

* node
* topic
* komunikasi antar node

Ini membantu memahami arsitektur robot.

---

# 20. Struktur Belajar yang Harus Dipahami

Urutan belajar ROS2 yang benar:

1. Linux
2. Terminal
3. ROS2 node
4. Topic
5. Publisher/subscriber
6. Service
7. Workspace
8. Package
9. Simulation
10. Hardware

Kalau langsung loncat ke AI vision tanpa ngerti topic ROS2 biasanya berakhir dengan:

> “kak ini error kenapa ya”

padahal terminal sudah jelas menulis error merah sepanjang dosa umat manusia.

---

# 21. Troubleshooting Umum

## colcon not found

Install:

```bash
sudo apt install python3-colcon-common-extensions -y
```

---

## ros2 command not found

Jalankan:

```bash
source /opt/ros/jazzy/setup.bash
```

---

## Gazebo tidak muncul

Coba:

```bash
gz sim -v 4
```

---

## Build gagal

Hapus build lama:

```bash
rm -rf build install log
```

Lalu build ulang:

```bash
colcon build
```

---

# 22. Hasil Akhir yang Diharapkan

Setelah pertemuan ini peserta diharapkan:

* Mengerti konsep ROS2
* Mengerti node dan topic
* Bisa install ROS2
* Bisa menjalankan Gazebo
* Bisa membuat workspace
* Bisa menjalankan publisher sederhana
* Mengerti komunikasi robot modern

---

# 23. Referensi

## Dokumentasi ROS2

[https://docs.ros.org/](https://docs.ros.org/)

## Dokumentasi Gazebo

[https://gazebosim.org/docs](https://gazebosim.org/docs)

## Dokumentasi ROS-GZ

[https://github.com/gazebosim/ros_gz](https://github.com/gazebosim/ros_gz)

---

# Penutup



# EAA MASIH ADA LANJUTANNYA
# SAUVC 2026 Simulation with Amarine GUI

Simulasi Singapore AUV Challenge 2026 menggunakan:

* ROS2
* Gazebo Harmonic
* ArduSub
* MAVROS
* BlueROV2
* Amarine GUI Launcher

Repository ini digunakan untuk menjalankan simulasi underwater robot secara lebih mudah menggunakan GUI launcher.

Karena ternyata hidup mahasiswa robotics belum cukup sulit tanpa harus menghafal 17 command terminal berbeda.

---

# 1. Features

## Simulation

* SAUVC Qualification World
* SAUVC Final World
* Underwater physics
* BlueROV2 simulation
* Camera simulation
* ArduSub integration

## GUI Launcher

GUI digunakan untuk:

* launch Gazebo
* launch ArduSub
* launch MAVROS
* launch ROS2 bridge
* menjalankan command penting dengan tombol GUI

---

# 2. Requirements

Pastikan sudah terinstall:

| Software | Version  |
| -------- | -------- |
| Ubuntu   | 24.04    |
| ROS2     | Jazzy    |
| Gazebo   | Harmonic |
| Python   | 3.10+    |

---

# 3. Workspace Setup

Buat workspace:

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
```

---

# 4. Clone Repositories

## Clone SAUVC World

```bash
git clone <SAUVC_WORLD_REPOSITORY>
```

---

## Clone Amarine GUI

```bash
git clone <AMARINE_GUI_REPOSITORY>
```

---

## Clone BlueROV2 Models

```bash
git clone https://github.com/clydemcqueen/bluerov2_gz.git
```

---

# 5. Build Workspace

Masuk workspace:

```bash
cd ~/ros2_ws
```

Build:

```bash
colcon build
```

Source workspace:

```bash
source install/setup.bash
```

Tambahkan ke `.bashrc`:

```bash
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc
```

Reload:

```bash
source ~/.bashrc
```

---

# 6. Setup Gazebo Resource Path

Tambahkan ke `.bashrc`:

```bash
export GZ_SIM_RESOURCE_PATH=$HOME/ros2_ws/src/sauvc26-world/models:$HOME/ros2_ws/src/sauvc26-world/worlds:$HOME/ros2_ws/src/bluerov2_gz/models
```

---

## Setup Plugin Path

Tambahkan:

```bash
export GZ_SIM_SYSTEM_PLUGIN_PATH=$HOME/ardupilot_gazebo/build
```

---

Reload bashrc:

```bash
source ~/.bashrc
```

---

# 7. Install GUI Dependencies

Masuk folder GUI:

```bash
cd ~/ros2_ws/src/amarine-gui
```

Jalankan setup:

```bash
bash setup_gui.sh
```

Kalau setup script belum tersedia:

```bash
pip install customtkinter
```

atau:

```bash
pip install -r requirements.txt
```

---

# 8. Running GUI

Masuk folder GUI:

```bash
cd ~/ros2_ws/src/amarine-gui
```

Jalankan GUI:

```bash
python3 command_gui.py
```

---

# 9. Setup Alias (Optional)

Tambahkan ke `.bashrc`:

```bash
alias gui='cd ~/ros2_ws/src/amarine-gui && python3 command_gui.py'
```

Reload:

```bash
source ~/.bashrc
```

Sekarang GUI dapat dijalankan hanya dengan:

```bash
gui
```

Karena manusia diciptakan untuk menekan tombol, bukan mengetik command 40 karakter berulang kali.

---

# 10. GUI Commands

GUI menyediakan beberapa command penting.

## Gazebo

### Qualification World

```bash
gz sim -v 3 -r sauvc_qualification.world
```

---

### Final World

```bash
gz sim -v 3 -r sauvc_final.world
```

---

# 11. Running ArduSub

Jalankan:

```bash
cd ~/ardupilot
Tools/autotest/sim_vehicle.py -L RATBeach -v ArduSub -f vectored --model=JSON --out=udp:0.0.0.0:14550 --console
```

Kalau berhasil:

* MAVProxy terbuka
* ArduSub connect ke Gazebo
* robot aktif di simulation

---

# 12. MAVProxy Commands

## Arm Robot

```bash
arm throttle
```

---

## Alt Hold Mode

```bash
mode alt_hold
```

---

## Thruster Test

```bash
rc 5 1550
```

---

## Disarm Robot

```bash
disarm
```

---

# 13. Camera Topic

Melihat stream camera:

```bash
gz topic -e -t /front_camera
```

Untuk melihat camera langsung di Gazebo:

* buka top bar Gazebo
* cari:

```text
Image Display
```

---

# 14. ROS2 Camera Bridge

Jalankan bridge:

```bash
ros2 run ros_gz_bridge parameter_bridge '/front_camera@sensor_msgs/msg/Image@ignition.msgs.Image'
```

---

# 15. View Camera in ROS2

## rqt_image_view

```bash
ros2 run rqt_image_view rqt_image_view /front_camera
```

---

## RViz2

```bash
rviz2
```

---

# 16. MAVROS

Launch MAVROS:

```bash
ros2 launch mavros apm.launch fcu_url:=udp://:14550@localhost:14555
```

---

# 17. Troubleshooting

## Gazebo Tidak Menemukan World

Error:

```text
Unable to find or download file
```

Pastikan:

```bash
echo $GZ_SIM_RESOURCE_PATH
```

mengarah ke:

* models
* worlds
* bluerov2_gz/models

---

## Robot Tidak Muncul

Error:

```text
Unable to find uri[model://bluerov2]
```

Pastikan repository `bluerov2_gz` sudah di-clone.

---

## Topic /clock Tidak Muncul

Install bridge:

```bash
sudo apt install ros-jazzy-ros-gz -y
```

Jalankan:

```bash
ros2 run ros_gz_bridge parameter_bridge /clock@rosgraph_msgs/msg/Clock[gz.msgs.Clock
```

---

## GUI Tidak Bisa Dibuka

Install dependency:

```bash
pip install customtkinter
```

---

## Gazebo Flickering

Kalau Gazebo berkedip di VMware:

```bash
LIBGL_ALWAYS_SOFTWARE=1 gz sim
```

Atau disable:

```text
Accelerate 3D Graphics
```

pada VMware settings.

---

# 18. Project Structure

```text
ros2_ws/
├── src/
│   ├── amarine-gui/
│   ├── sauvc26-world/
│   └── bluerov2_gz/
```

---

# 19. Credits

## BlueROV2 Gazebo

```text
https://github.com/clydemcqueen/bluerov2_gz
```

---

## ArduPilot Gazebo

```text
https://github.com/ArduPilot/ardupilot_gazebo
```

---

# Penutup

Dengan setup ini peserta dapat:

* menjalankan simulasi SAUVC
* testing movement underwater
* menjalankan ROS2 communication
* testing camera dan vision
* simulasi autonomous mission

Dan yang paling penting:

menghabiskan lebih sedikit waktu mencari command terminal yang hilang di chat Discord minggu lalu.
