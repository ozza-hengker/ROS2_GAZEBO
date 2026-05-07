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

Install:

```bash
sudo apt install gz-harmonic -y
```

Install bridge ROS-Gazebo:

```bash
sudo apt install ros-jazzy-ros-gz -y
```

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

Karena satu titik koma bisa menghancurkan harga diri engineer selama 3 jam.

---

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

Kalau hari pertama terasa membingungkan itu normal.

ROS2 memang bukan teknologi yang ramah untuk pemula.
Tapi begitu konsep node dan topic mulai nyambung, semua sistem robot nanti terasa jauh lebih masuk akal.

Dan di situlah perjalanan debugging tanpa akhir dimulai.
