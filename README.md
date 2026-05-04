# 🌊 Pertemuan 3: ROS 2 & Gazebo Simulation (Software)

**Divisi Programming Motion | OPREC Learning Week 2026**

Selamat datang di Pertemuan 3! Setelah memahami dasar-dasar Linux, pada tahap ini kita mulai masuk ke inti sistem robotika: **ROS 2** sebagai “otak” robot, dan **Gazebo** sebagai lingkungan simulasi berbasis fisika.

---

## 🎯 Tujuan Pembelajaran

* Memahami arsitektur komunikasi robot menggunakan ROS 2
* Mengenal Gazebo sebagai simulator 3D berbasis fisika
* Mengimplementasikan robot differential drive
* Membuat robot yang mampu mengejar target secara otomatis

---

## 🛠️ Instalasi ROS 2 Jazzy & Gazebo

> Lewati langkah ini jika sudah terpasang

```bash
sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

sudo apt install software-properties-common
sudo add-apt-repository universe
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

sudo apt update
sudo apt install ros-jazzy-desktop ros-jazzy-gazebo-ros-pkgs -y

echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

---

## 🚀 PROJECT: THE CHASER BOT

Membangun robot beroda (Differential Drive) yang dapat mengejar target secara otomatis di Gazebo.

---

## 📁 Langkah 1: Setup Workspace ROS 2

```bash
mkdir -p ~/chaser_ws/src
cd ~/chaser_ws/src

ros2 pkg create chaser_bot --build-type ament_python --dependencies rclpy geometry_msgs gazebo_msgs
```

---

## 🤖 Langkah 2: Desain Robot (URDF)

```bash
cd ~/chaser_ws/src/chaser_bot/chaser_bot
```

Buat file `chaser.urdf`:

```xml
<?xml version="1.0" ?>
<robot name="chaser_bot">
  <link name="base_link">
    <visual>
      <geometry><box size="0.4 0.2 0.1"/></geometry>
      <material name="blue"><color rgba="0 0.5 1 1"/></material>
    </visual>
    <collision><geometry><box size="0.4 0.2 0.1"/></geometry></collision>
  </link>

  <link name="left_wheel">
    <visual><geometry><cylinder radius="0.08" length="0.04"/></geometry></visual>
  </link>

  <joint name="left_wheel_joint" type="continuous">
    <parent link="base_link"/>
    <child link="left_wheel"/>
  </joint>

  <link name="right_wheel">
    <visual><geometry><cylinder radius="0.08" length="0.04"/></geometry></visual>
  </link>

  <joint name="right_wheel_joint" type="continuous">
    <parent link="base_link"/>
    <child link="right_wheel"/>
  </joint>

  <gazebo>
    <plugin name="diff_drive" filename="libgazebo_ros_diff_drive.so">
      <left_joint>left_wheel_joint</left_joint>
      <right_joint>right_wheel_joint</right_joint>
      <command_topic>cmd_vel</command_topic>
    </plugin>
  </gazebo>
</robot>
```

---

## 🧠 Langkah 3: Node Python (Chaser Logic)

Buat file `chaser_node.py`:

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from geometry_msgs.msg import Twist
from gazebo_msgs.msg import ModelStates
import math

class ChaserBot(Node):
    def __init__(self):
        super().__init__('chaser_bot_node')
        self.cmd_pub = self.create_publisher(Twist, '/cmd_vel', 10)
        self.state_sub = self.create_subscription(
            ModelStates,
            '/gazebo/model_states',
            self.state_callback,
            10
        )

    def state_callback(self, msg):
        try:
            bot_idx = msg.name.index('chaser_bot')
            target_idx = msg.name.index('unit_sphere')

            bot_pos = msg.pose[bot_idx].position
            target_pos = msg.pose[target_idx].position

            dx = target_pos.x - bot_pos.x
            dy = target_pos.y - bot_pos.y

            jarak = math.sqrt(dx**2 + dy**2)
            sudut_target = math.atan2(dy, dx)

            cmd = Twist()

            if jarak < 0.8:
                cmd.linear.x = 0.0
            else:
                cmd.linear.x = 0.5
                cmd.angular.z = 1.5 * sudut_target

            self.cmd_pub.publish(cmd)

        except ValueError:
            pass

def main():
    rclpy.init()
    node = ChaserBot()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

```bash
chmod +x chaser_node.py
```

---

## ⚙️ Langkah 4: Build Package

```bash
cd ~/chaser_ws/src/chaser_bot

sed -i "s/'console_scripts': \[/'console_scripts': [\n            'chaser_node = chaser_bot.chaser_node:main',/g" setup.py

cd ~/chaser_ws
colcon build
source install/setup.bash
```

---

## 🎮 Langkah 5: Simulasi

### Terminal 1 – Jalankan Gazebo & Spawn Robot

```bash
source ~/chaser_ws/install/setup.bash
ros2 launch gazebo_ros gazebo.launch.py &

sleep 5

ros2 run gazebo_ros spawn_entity.py \
-entity chaser_bot \
-file ~/chaser_ws/src/chaser_bot/chaser_bot/chaser.urdf \
-z 0.5
```

---

### Terminal 2 – Tambahkan Target

Di Gazebo:

* Klik **Insert → Sphere**
* Tempatkan bola (target)

---

### Terminal 3 – Jalankan Node

```bash
source ~/chaser_ws/install/setup.bash
ros2 run chaser_bot chaser_node
```

---

## 💡 Konsep Penting

* Robot menggunakan **Differential Drive**
* Kontrol gerak berbasis:

  * Linear velocity (`cmd.linear.x`)
  * Angular velocity (`cmd.angular.z`)
* Menggunakan **P-Controller sederhana**:

```python
cmd.angular.z = 1.5 * error_sudut
```

Semakin besar error sudut → robot belok lebih cepat
Semakin kecil → gerakan lebih halus

---

## 🏁 Hasil Akhir

Robot akan:

* Mendeteksi posisi target (bola)
* Menghitung arah dan jarak
* Bergerak menuju target
* Berhenti saat cukup dekat

---

## 🎉 Kesimpulan

Pada pertemuan ini, kita berhasil:

* Membangun robot di Gazebo
* Menghubungkan ROS 2 dengan simulasi
* Mengimplementasikan kontrol gerak dasar
* Membuat sistem autonomous sederhana

Ini adalah fondasi penting untuk pengembangan sistem robotika yang lebih kompleks, termasuk AUV di dunia nyata.

---
