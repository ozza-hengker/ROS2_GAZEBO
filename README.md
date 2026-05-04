🌊 Pertemuan 3: ROS 2 & Gazebo Simulation (Software)

Divisi Programming Motion | OPREC Learning Week 2026

Selamat datang di Pertemuan 3! Setelah kalian menguasai dasar-dasar Linux, sekarang kita akan masuk ke "Otak" dari Autonomous Underwater Vehicle (AUV) kita: Robot Operating System (ROS 2), dan mengujinya di dunia virtual menggunakan Gazebo Simulator.

🎯 Tujuan Pembelajaran

Memahami arsitektur komunikasi robot menggunakan ROS 2.

Mengenal Gazebo sebagai lingkungan simulasi 3D berdasar hukum fisika nyata.

Ultimate Project: Membuat robot mobil beroda dari nol hingga bisa mengejar target secara otomatis.

🛠️ Langkah 0: Instalasi ROS 2 Jazzy & Gazebo

(Jika kalian sudah menginstal ROS 2, lewati langkah ini)

Buka terminal Ubuntu 24.04 kalian, lalu copy-paste kumpulan perintah ini untuk menginstal ROS 2 Jazzy dan Gazebo secara otomatis:

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

sudo apt install software-properties-common
sudo add-apt-repository universe
sudo apt update && sudo apt install curl -y
sudo curl -sSL [https://raw.githubusercontent.com/ros/rosdistro/master/ros.key](https://raw.githubusercontent.com/ros/rosdistro/master/ros.key) -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] [http://packages.ros.org/ros2/ubuntu](http://packages.ros.org/ros2/ubuntu) $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

sudo apt update
sudo apt install ros-jazzy-desktop ros-jazzy-gazebo-ros-pkgs -y
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
source ~/.bashrc


🚀 PROJECT: THE CHASER BOT

Hari ini kita tidak hanya belajar teori. Kita akan membangun robot beroda (Differential Drive - 2 Roda Belakang, 1 Roda Penyeimbang Depan) dan memprogramnya agar bisa mengejar bola merah di Gazebo.

Langkah 1: Setup Workspace ROS 2

Kita harus membuat "ruang kerja" (workspace) untuk menyimpan kode kita. Buka terminal dan jalankan:

# Membuat folder workspace
mkdir -p ~/chaser_ws/src
cd ~/chaser_ws/src

# Membuat ROS 2 Package menggunakan Python
ros2 pkg create chaser_bot --build-type ament_python --dependencies rclpy geometry_msgs gazebo_msgs


Langkah 2: Membuat Desain Fisik Robot (URDF)

Robot di Gazebo dibentuk menggunakan file XML khusus bernama URDF (Unified Robot Description Format). Kita akan membuat desain sasis kotak dengan 2 roda besar di belakang dan 1 roda kecil (caster) di depan.

Jalankan perintah ini di terminal untuk langsung membuat file desain robotnya:

cd ~/chaser_ws/src/chaser_bot/chaser_bot
cat << 'EOF' > chaser.urdf
<?xml version="1.0" ?>
<robot name="chaser_bot">
  <!-- Sasis Utama (Kotak) -->
  <link name="base_link">
    <visual>
      <geometry><box size="0.4 0.2 0.1"/></geometry>
      <material name="blue"><color rgba="0 0.5 1 1"/></material>
    </visual>
    <collision><geometry><box size="0.4 0.2 0.1"/></geometry></collision>
    <inertial><mass value="2.0"/><inertia ixx="0.01" ixy="0" ixz="0" iyy="0.01" iyz="0" izz="0.01"/></inertial>
  </link>

  <!-- Roda Kiri Belakang -->
  <link name="left_wheel">
    <visual><geometry><cylinder radius="0.08" length="0.04"/></geometry><material name="black"><color rgba="0 0 0 1"/></material></visual>
    <collision><geometry><cylinder radius="0.08" length="0.04"/></geometry></collision>
    <inertial><mass value="0.5"/><inertia ixx="0.001" ixy="0" ixz="0" iyy="0.001" iyz="0" izz="0.001"/></inertial>
  </link>
  <joint name="left_wheel_joint" type="continuous">
    <parent link="base_link"/><child link="left_wheel"/>
    <origin xyz="-0.1 0.12 0" rpy="-1.5708 0 0"/><axis xyz="0 0 1"/>
  </joint>

  <!-- Roda Kanan Belakang -->
  <link name="right_wheel">
    <visual><geometry><cylinder radius="0.08" length="0.04"/></geometry><material name="black"><color rgba="0 0 0 1"/></material></visual>
    <collision><geometry><cylinder radius="0.08" length="0.04"/></geometry></collision>
    <inertial><mass value="0.5"/><inertia ixx="0.001" ixy="0" ixz="0" iyy="0.001" iyz="0" izz="0.001"/></inertial>
  </link>
  <joint name="right_wheel_joint" type="continuous">
    <parent link="base_link"/><child link="right_wheel"/>
    <origin xyz="-0.1 -0.12 0" rpy="-1.5708 0 0"/><axis xyz="0 0 1"/>
  </joint>

  <!-- Roda Depan (Caster Kecil Penyeimbang) -->
  <link name="caster_wheel">
    <visual><geometry><sphere radius="0.04"/></geometry><material name="grey"><color rgba="0.5 0.5 0.5 1"/></material></visual>
    <collision><geometry><sphere radius="0.04"/></geometry></collision>
    <inertial><mass value="0.2"/><inertia ixx="0.0001" ixy="0" ixz="0" iyy="0.0001" iyz="0" izz="0.0001"/></inertial>
  </link>
  <joint name="caster_joint" type="fixed">
    <parent link="base_link"/><child link="caster_wheel"/>
    <origin xyz="0.15 0 -0.04" rpy="0 0 0"/>
  </joint>

  <!-- Plugin Penggerak (Otak Roda) -->
  <gazebo>
    <plugin name="diff_drive" filename="libgazebo_ros_diff_drive.so">
      <left_joint>left_wheel_joint</left_joint>
      <right_joint>right_wheel_joint</right_joint>
      <wheel_separation>0.24</wheel_separation>
      <wheel_diameter>0.16</wheel_diameter>
      <max_wheel_torque>20</max_wheel_torque>
      <max_wheel_acceleration>1.0</max_wheel_acceleration>
      <command_topic>cmd_vel</command_topic>
      <publish_odom>true</publish_odom>
      <publish_odom_tf>true</publish_odom_tf>
      <publish_wheel_tf>true</publish_wheel_tf>
      <odometry_frame>odom</odometry_frame>
      <robot_base_frame>base_link</robot_base_frame>
    </plugin>
  </gazebo>
</robot>
EOF


Langkah 3: Menulis Logika Otak (Python Node)

Sekarang kita buat program cerdasnya. Node Python ini akan bertindak sebagai Subscriber yang mengintip posisi semua objek di Gazebo (/gazebo/model_states), mencari kordinat target, lalu menghitung sudut belok. Setelah itu, Node ini akan menjadi Publisher yang mengirim kecepatan ke roda (/cmd_vel).

Jalankan perintah ini untuk membuat file Python-nya:

cat << 'EOF' > chaser_node.py
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from geometry_msgs.msg import Twist
from gazebo_msgs.msg import ModelStates
import math

class ChaserBot(Node):
    def __init__(self):
        super().__init__('chaser_bot_node')
        
        # Publisher untuk menggerakkan roda
        self.cmd_pub = self.create_publisher(Twist, '/cmd_vel', 10)
        
        # Subscriber untuk mengintip koordinat dari Gazebo
        self.state_sub = self.create_subscription(ModelStates, '/gazebo/model_states', self.state_callback, 10)
        
        self.get_logger().info('🤖 Chaser Bot Aktif! Menunggu target (unit_sphere)...')

    def state_callback(self, msg):
        try:
            # 1. Cari index robot dan index bola target
            bot_idx = msg.name.index('chaser_bot')
            target_idx = msg.name.index('unit_sphere') # Ini adalah nama default bola di Gazebo
            
            # 2. Ambil posisi X dan Y
            bot_pos = msg.pose[bot_idx].position
            target_pos = msg.pose[target_idx].position
            
            # 3. Hitung sudut dari Quaternion (Mencari arah hadap robot)
            q = msg.pose[bot_idx].orientation
            bot_yaw = math.atan2(2.0 * (q.w * q.z + q.x * q.y), 1.0 - 2.0 * (q.y * q.y + q.z * q.z))
            
            # 4. MATEMATIKA MOTION: Hitung sudut ke target
            dx = target_pos.x - bot_pos.x
            dy = target_pos.y - bot_pos.y
            jarak = math.sqrt(dx**2 + dy**2)
            sudut_target = math.atan2(dy, dx)
            
            # Hitung error/selisih sudut (Seberapa jauh kita harus belok?)
            error_sudut = sudut_target - bot_yaw
            
            # Normalisasi sudut agar robot tidak muter balik terlalu jauh
            while error_sudut > math.pi: error_sudut -= 2.0 * math.pi
            while error_sudut < -math.pi: error_sudut += 2.0 * math.pi

            # 5. LOGIKA KENDALI (Kirim ke Motor)
            cmd = Twist()
            
            if jarak < 0.8:
                self.get_logger().info('✅ Target Tertangkap! Berhenti.')
                cmd.linear.x = 0.0
                cmd.angular.z = 0.0
            else:
                self.get_logger().info(f'Mengejar... Jarak: {jarak:.2f}m | Belok: {error_sudut:.2f} rad')
                # Maju perlahan
                cmd.linear.x = 0.5 
                # Belok sebanding dengan error sudut (Konsep dasar Proportional / P-Controller)
                cmd.angular.z = 1.5 * error_sudut 

            self.cmd_pub.publish(cmd)
            
        except ValueError:
            # Jika objek 'unit_sphere' belum dimasukkan ke Gazebo
            pass

def main(args=None):
    rclpy.init(args=args)
    chaser = ChaserBot()
    rclpy.spin(chaser)
    chaser.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
EOF

# Beri izin eksekusi
chmod +x chaser_node.py


Langkah 4: Build Package

Karena kita membuat paket ROS 2 baru, kita harus melakukan build agar file Python kita terdaftar di sistem.
(Buka file setup.py di folder ~/chaser_ws/src/chaser_bot/, dan pastikan chaser_node terdaftar di entry_points. Jalankan perintah di bawah ini untuk memperbaikinya otomatis):

cd ~/chaser_ws/src/chaser_bot
sed -i "s/'console_scripts': \[/'console_scripts': [\n            'chaser_node = chaser_bot.chaser_node:main',/g" setup.py
cd ~/chaser_ws
colcon build
source install/setup.bash


🎮 Langkah 5: SHOWTIME! (Simulasi Pengejaran)

Sekarang saatnya kita gabungkan semuanya. Kita butuh 3 Jendela Terminal.

Terminal 1: Buka Gazebo & Munculkan Robot

source ~/chaser_ws/install/setup.bash
ros2 launch gazebo_ros gazebo.launch.py &
sleep 5
ros2 run gazebo_ros spawn_entity.py -entity chaser_bot -file ~/chaser_ws/src/chaser_bot/chaser_bot/chaser.urdf -z 0.5


(Lihat ke Gazebo, robot kalian yang punya 2 ban belakang dan 1 caster depan sudah muncul!)

Terminal 2: Masukkan Target (Bola)
Di aplikasi Gazebo:

Perhatikan menu bagian atas.

Klik logo Bola / Sphere (Insert Sphere).

Klik di lokasi yang agak jauh dari robot kalian. Ini akan memunculkan target dengan nama unit_sphere.

Terminal 3: Nyalakan Otak Robot (Chaser Node)
Buka terminal baru, lalu ketik:

source ~/chaser_ws/install/setup.bash
ros2 run chaser_bot chaser_node


BOOM! 💥
Lihat kembali ke jendela Gazebo. Robot kalian sekarang akan merespon, berputar arah secara mandiri, lalu melaju lurus mendekati bola tersebut sampai berjarak dekat dan otomatis mengerem!

Jika kamu memindahkan bola itu menggunakan fitur Move (Tanda panah 4 arah) di Gazebo, robot itu akan kembali mengejarnya!

Catatan Mentor: Perhatikan bagian kode cmd.angular.z = 1.5 * error_sudut. Ini adalah bentuk paling fundamental dari Kontrol Proportional (P-Controller). Jika error sudutnya besar, roda akan berputar kencang. Jika sudah lurus (error kecil), putaran roda akan melambat dan halus. Inilah yang divisi Motion kerjakan setiap hari!

Selamat! Kalian telah berhasil menyelesaikan simulasi Misi AUV skala darat pertama kalian! 🥳
