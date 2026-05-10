# Import SolidWorks into Gazebo Classic (PX4 SITL & HITL)

> Hướng dẫn đưa model SolidWorks vào Gazebo Classic để chạy mô phỏng SITL và HITL với PX4.  
> **A** = tên model bạn đặt khi export (ví dụ: `octorotor_coaxial`)  
> **B** = tên folder model trong Gazebo (ví dụ: `octorotor_coaxial`)

---

## Phần 1 — Chuẩn bị model (SolidWorks → SDF)

### Bước 1 — Vẽ lại model bằng solid block

Vẽ lại model bằng các khối đặc (**solid blocks**). Không dùng hình rỗng (hollow shapes) vì có thể gây lỗi collision trong Gazebo.

### Bước 2 — Cài URDF Exporter cho SolidWorks

Truy cập link sau để tải plugin phù hợp với phiên bản SolidWorks của bạn.  
Version **1.16** dùng cho SolidWorks 2021 trở lên.

```
https://github.com/ros/solidworks_urdf_exporter?tab=readme-ov-file
```

### Bước 3 — Export URDF từ SolidWorks

1. Đặt **điểm** và **trục** tại tâm khối lượng của **base** và từng **cánh quạt**
2. Thêm coordinate frames vào các điểm đó
3. Vào **Tools → Export as URDF**, export ra folder **A**

Tham khảo video hướng dẫn:
```
https://www.youtube.com/watch?v=H6YPkXmkdPg&t=1884s
```

### Bước 4 — Kiểm tra collision geometry

Upload folder **A** lên trang sau, bật **Show Collision** để kiểm tra collision geometry có đúng không:

```
https://gkjohnson.github.io/urdf-loaders/javascript/example/bundle/index.html
```

### Bước 5 — Convert URDF sang SDF

Copy folder **A** vào Gazebo workspace, vào folder **A/urdf/**, chạy lệnh:

```bash
gz sdf -p A.urdf > A.sdf
```

> **Lưu ý:** Tên file `.sdf` phải trùng với tên folder export. Ví dụ: folder tên `octorotor_coaxial` thì file phải là `octorotor_coaxial.sdf`.

---

## Phần 2 — Thêm model vào Gazebo

### Bước 6 — Tạo folder model trong Gazebo

Vào folder:
```
PX4-Autopilot/Tools/simulation/gazebo-classic/sitl_gazebo-classic/models/
```

Tạo folder mới tên **B** (ví dụ: `octorotor_coaxial`).  
Từ folder **A**, copy **mesh folder** và file **A.sdf** vào folder **B**.

### Bước 7 — Tạo file `model.config`

Trong folder **B**, tạo file `model.config` với nội dung sau:

```xml
<?xml version="1.0"?>
<model>
  <name>octorotor_coaxial</name>
  <version>1.0</version>
  <sdf version="1.7">octorotor_coaxial.sdf</sdf>
  <author>
    <name>Your Name</name>
    <email>you@example.com</email>
  </author>
  <description>
    Coaxial octorotor UAV converted from URDF to SDF
  </description>
</model>
```

| Trường | Mô tả |
|--------|-------|
| `<name>` | Tên model |
| `<sdf>` | Tên file A.sdf sẽ được gọi khi chạy |
| `<author>` | Thông tin người tạo |
| `<description>` | Mô tả model |

---

## Phần 3 — Chỉnh sửa file SDF

### Bước 8 — Thêm các plugin vào file A.sdf

Mở file **A.sdf** trong folder **B** và thêm các đoạn sau.

#### 8.1 — IMU link & joint (thêm ngay sau `</link>` của base_link)

```xml
<link name='/imu_link'>
  <pose>0 0 0.085 0 0 0</pose>
  <inertial>
    <pose>0 0 0 0 0 0</pose>
    <mass>0.015</mass>
    <inertia>
      <ixx>1e-05</ixx>
      <ixy>0</ixy>
      <ixz>0</ixz>
      <iyy>1e-05</iyy>
      <iyz>0</iyz>
      <izz>1e-05</izz>
    </inertia>
  </inertial>
</link>
<joint name='/imu_joint' type='revolute'>
  <child>/imu_link</child>
  <parent>base_link</parent>
  <axis>
    <xyz>0 0 1</xyz>
    <limit>
      <lower>0</lower>
      <upper>0</upper>
      <effort>0</effort>
      <velocity>0</velocity>
    </limit>
    <dynamics>
      <spring_reference>0</spring_reference>
      <spring_stiffness>0</spring_stiffness>
    </dynamics>
    <use_parent_model_frame>1</use_parent_model_frame>
  </axis>
</joint>
```

#### 8.2 — Rotational joint (kiểm tra từng joint cánh quạt)

Mỗi joint quay **bắt buộc** phải có đủ các phần sau, nếu thiếu cánh quạt sẽ không quay được. Lặp lại cho **tất cả** các joint cánh quạt:

```xml
<link name='rotor_0'>
  <pose>0.1905 -0.1905 0.23 0 0 0</pose>
  <inertial>
    <pose>0 0 0 0 0 0</pose>
    <mass>0.005</mass>
    <inertia>
      <ixx>9.75e-07</ixx>
      <ixy>0</ixy>
      <ixz>0</ixz>
      <iyy>0.000273104</iyy>
      <iyz>0</iyz>
      <izz>0.000274004</izz>
    </inertia>
  </inertial>
  <collision name='rotor_0_collision'>
    <pose>0 0 0.0045 0 0 0</pose>
    <geometry>
      <cylinder>
        <length>0.005</length>
        <radius>0.128</radius>
      </cylinder>
    </geometry>
    <surface>
      <contact>
        <ode/>
      </contact>
      <friction>
        <ode/>
      </friction>
    </surface>
  </collision>
  <visual name='rotor_0_visual'>
    <pose>0 0 0 0 0 0</pose>
    <geometry>
      <mesh>
        <scale>1 1 1</scale>
        <uri>model://iris/meshes/iris_prop_ccw.dae</uri>
      </mesh>
    </geometry>
    <material>
      <script>
        <name>Gazebo/Blue</name>
        <uri>file://media/materials/scripts/gazebo.material</uri>
      </script>
    </material>
  </visual>
  <gravity>1</gravity>
  <velocity_decay/>
</link>
<joint name='rotor_0_joint' type='revolute'>
  <child>rotor_0</child>
  <parent>base_link</parent>
  <axis>
    <xyz>0 0 1</xyz>
    <limit>
      <lower>-1e+16</lower>
      <upper>1e+16</upper>
    </limit>
    <dynamics>
      <spring_reference>0</spring_reference>
      <spring_stiffness>0</spring_stiffness>
    </dynamics>
    <use_parent_model_frame>1</use_parent_model_frame>
  </axis>
</joint>
```

#### 8.3 — Plugin base (copy nguyên, không sửa)

```xml
<plugin name='rosbag' filename='libgazebo_multirotor_base_plugin.so'>
  <robotNamespace/>
  <linkName>base_link</linkName>
  <rotorVelocitySlowdownSim>10</rotorVelocitySlowdownSim>
</plugin>
```

#### 8.4 — Plugin motor (lặp lại cho từng motor)

```xml
<plugin name='front_right_motor_model' filename='libgazebo_motor_model.so'>
  <robotNamespace/>
  <jointName>rotor_0_joint</jointName>          <!-- tên joint và link tương ứng -->
  <linkName>rotor_0</linkName>
  <turningDirection>ccw</turningDirection>  <!-- ccw hoặc cw nhìn từ trên -->
  <timeConstantUp>0.0125</timeConstantUp>
  <timeConstantDown>0.025</timeConstantDown>
  <maxRotVelocity>1199</maxRotVelocity>
  <motorConstant>7.2706e-06</motorConstant>   <!-- thông số motor -->
  <momentConstant>0.06</momentConstant>
  <commandSubTopic>/gazebo/command/motor_speed</commandSubTopic>
  <motorNumber>0</motorNumber>              <!-- index bắt đầu từ 0 -->
  <rotorDragCoefficient>0.000175</rotorDragCoefficient>
  <rollingMomentCoefficient>1e-06</rollingMomentCoefficient>
  <motorSpeedPubTopic>/motor_speed/0</motorSpeedPubTopic>
  <rotorVelocitySlowdownSim>10</rotorVelocitySlowdownSim>
</plugin>
```

#### 8.5 — GPS, sensors, và MAVLink plugins (copy nguyên, không sửa)

```xml
<include>
  <uri>model://gps</uri>
  <pose>0 0 0.144 0 0 0</pose>
  <name>gps0</name>
</include>
<joint name='gps0_joint' type='fixed'>
  <child>gps0::link</child>
  <parent>base_link</parent>
</joint>
<plugin name='groundtruth_plugin' filename='libgazebo_groundtruth_plugin.so'>
  <robotNamespace/>
</plugin>
<plugin name='magnetometer_plugin' filename='libgazebo_magnetometer_plugin.so'>
  <robotNamespace/>
  <pubRate>100</pubRate>
  <noiseDensity>0.0004</noiseDensity>
  <randomWalk>6.4e-06</randomWalk>
  <biasCorrelationTime>600</biasCorrelationTime>
  <magTopic>/mag</magTopic>
</plugin>
<plugin name='barometer_plugin' filename='libgazebo_barometer_plugin.so'>
  <robotNamespace/>
  <pubRate>50</pubRate>
  <baroTopic>/baro</baroTopic>
  <baroDriftPaPerSec>0</baroDriftPaPerSec>
</plugin>
```

#### 8.6 — MAVLink interface plugin

Plugin sau phần (motor index và joint name) chỉnh theo số motor của bạn, lặp lại cho tất cả motor:

```xml
<plugin name='mavlink_interface' filename='libgazebo_mavlink_interface.so'>
    <robotNamespace/>
    <imuSubTopic>/imu</imuSubTopic>
    <magSubTopic>/mag</magSubTopic>
    <baroSubTopic>/baro</baroSubTopic>
    <mavlink_addr>INADDR_ANY</mavlink_addr>
    <mavlink_tcp_port>4560</mavlink_tcp_port>
    <mavlink_udp_port>14560</mavlink_udp_port>
    <serialEnabled>0</serialEnabled>
    <serialDevice>/dev/ttyACM0</serialDevice>
    <baudRate>921600</baudRate>
    <qgc_addr>INADDR_ANY</qgc_addr>
    <qgc_udp_port>14550</qgc_udp_port>
    <sdk_addr>INADDR_ANY</sdk_addr>
    <sdk_udp_port>14540</sdk_udp_port>
    <hil_mode>0</hil_mode>
    <hil_state_level>0</hil_state_level>
    <send_vision_estimation>0</send_vision_estimation>
    <send_odometry>1</send_odometry>
    <enable_lockstep>1</enable_lockstep>
    <use_tcp>1</use_tcp>
    <motorSpeedCommandPubTopic>/gazebo/command/motor_speed</motorSpeedCommandPubTopic>
    <control_channels>
      <channel name='rotor1'>
        <input_index>0</input_index>
        <input_offset>0</input_offset>
        <input_scaling>1000</input_scaling>
        <zero_position_disarmed>0</zero_position_disarmed>
        <zero_position_armed>100</zero_position_armed>
        <joint_control_type>velocity</joint_control_type>
      </channel>
    <channel name='rotor2'>
      <!-- lặp lại tương tự cho các motor còn lại -->
    </channel>
  </control_channels>
</plugin>
```

#### 8.7 — IMU plugin (copy nguyên, không sửa)

```xml
<static>0</static>
<l<plugin name='rotors_gazebo_imu_plugin' filename='libgazebo_imu_plugin.so'>
  <robotNamespace/>
  inkName>/imu_link</linkName>
  <imuTopic>/imu</imuTopic>
  <gyroscopeNoiseDensity>0.00018665</gyroscopeNoiseDensity>
  <gyroscopeRandomWalk>3.8785e-05</gyroscopeRandomWalk>
  <gyroscopeBiasCorrelationTime>1000.0</gyroscopeBiasCorrelationTime>
  <gyroscopeTurnOnBiasSigma>0.0087</gyroscopeTurnOnBiasSigma>
  <accelerometerNoiseDensity>0.00186</accelerometerNoiseDensity>
  <accelerometerRandomWalk>0.006</accelerometerRandomWalk>
  <accelerometerBiasCorrelationTime>300.0</accelerometerBiasCorrelationTime>
  <accelerometerTurnOnBiasSigma>0.196</accelerometerTurnOnBiasSigma>
</plugin>
```

> **Tham khảo SDF đầy đủ:** https://github.com/thanhtnngoc/Example-octorotor_coaxial.sdf

---

## Phần 4 — Cấu hình PX4 SITL

### Bước 9 — Tạo airframe file cho SITL

Vào folder:
```
PX4-Autopilot/ROMFS/px4fmu_common/init.d-posix/airframes/
```

Tạo file mới theo pattern: `ID_gazebo-classic_modelname`  
Dùng ID trong khoảng **22000 – 22999**.

Ví dụ: `22000_gazebo-classic_octorotor_coaxial`


### Bước 10 — Nội dung airframe file SITL

Điền nội dung vào file tạo ở Bước 9:

```sh
#!/bin/sh
# @name Octorotor Coaxial Custom
# @type Octorotor coaxial
# @class Copter
# @maintainer Your Name <your@email.com>

. ${R}etc/init.d/rc.mc_defaults

param set-default CA_AIRFRAME 0
param set-default CA_ROTOR_COUNT 8

# Rotor 1 — vị trí và chiều quay
param set-default CA_ROTOR0_PX  0.192121   # khoảng cách theo X từ tâm (m)
param set-default CA_ROTOR0_PY  0.193271   # khoảng cách theo Y từ tâm (m)
param set-default CA_ROTOR0_KM  0.05       # KM > 0: CCW | KM < 0: CW

# Rotor 2
param set-default CA_ROTOR1_PX  0.192452
param set-default CA_ROTOR1_PY -0.192939
param set-default CA_ROTOR1_KM -0.05

# ... lặp lại cho tất cả motor

# PWM output mapping (số lượng tương ứng số motor)
param set-default PWM_MAIN_FUNC1 101
param set-default PWM_MAIN_FUNC2 102
# ... tiếp tục đến hết số motor
```

**KM convention:**
| Giá trị KM | Chiều quay |
|-----------|-----------|
| `KM > 0` | Ngược chiều kim đồng hồ (CCW) |
| `KM < 0` | Thuận chiều kim đồng hồ (CW) |


### Bước 11 — Đăng ký airframe trong CMakeLists.txt

Trong cùng folder trên, mở `CMakeLists.txt`, thêm tên file vào cuối danh sách:

```cmake
# [22000, 22999] Reserve for custom models
22000_gazebo-classic_octorotor_coaxial
```

### Bước 12 — Thêm model vào danh sách SITL targets

Vào file:
```
PX4-Autopilot/src/modules/simulation/simulator_mavlink/sitl_targets_gazebo-classic
```

Tìm section `set(models ...)` và thêm tên model vào:

```cmake
set(models
    advanced_plane
    believer
    iris
    iris_dual_gps
    octorotor_coaxial   # thêm vào đây
)
```

### Bước 13 — Chạy SITL

```bash
make px4_sitl gazebo-classic_octorotor_coaxial
```

---

## Phần 5 — Cấu hình PX4 HITL

> Từ bước 14 trở đi là cấu hình cho **HITL** (Hardware-In-The-Loop).

### Bước 14 — Tạo folder model HITL

1. Vào folder `PX4-Autopilot/Tools/simulation/gazebo-classic/sitl_gazebo-classic/models/`
2. Copy folder **B** (tạo ở Bước 6), đổi tên thành **B_hitl** (ví dụ: `octorotor_coaxial_hitl`)
3. Trong folder **B_hitl**, giữ lại: file `model.config` và file `A.sdf`
4. Đổi tên `A.sdf` thành `A_hitl.sdf`
5. Trong `A_hitl.sdf`, sửa các tham số sau trong plugin `mavlink_interface`:

```xml
<serialEnabled>1</serialEnabled>      <!-- đổi từ 0 thành 1 -->
<hil_mode>1</hil_mode>                <!-- đổi từ 0 thành 1 -->
<enable_lockstep>0</enable_lockstep>  <!-- đổi từ 1 thành 0 -->
<use_tcp>0</use_tcp>                  <!-- đổi từ 1 thành 0 -->
```

### Bước 14b — Cập nhật `model.config` cho HITL

```xml
<?xml version="1.0"?>
<model>
  <name>octorotor_coaxial_hitl</name>
  <version>1.0</version>
  <sdf version="1.7">octorotor_coaxial_hitl.sdf</sdf>
  <author>
    <name>Your Name</name>
    <email>you@example.com</email>
  </author>
  <description>
    Coaxial octorotor UAV converted from URDF to SDF
  </description>
</model>
```

### Bước 15 — Tạo airframe file HITL

Vào folder:
```
PX4-Autopilot/ROMFS/px4fmu_common/init.d/airframes/
```

Tạo file mới theo pattern: `ID_modelname.hil`  
Dùng ID trong khoảng **1000 – 1999**.

Ví dụ: `1103_octorotor.hil`

```sh
#!/bin/sh
# @name HIL Octocopter X
# @type Simulation
# @class Copter
# @maintainer Lorenz Meier <lorenz@px4.io>
# @board px4_fmu-v2 exclude

. ${R}etc/init.d/rc.mc_defaults

param set SYS_HITL 1
param set UAVCAN_ENABLE 0
param set-default MAV_TYPE 14      # MAVLink airframe type (14 = Octorotor)

# Disable checks (không cần battery/safety switch thật)
param set-default CBRK_SUPPLY_CHK 894281
param set-default CBRK_IO_SAFETY 22027

param set-default CA_ROTOR_COUNT 8

# Rotor positions và KM (giống SITL)
param set-default CA_ROTOR0_PX  0.192121
param set-default CA_ROTOR0_PY  0.193271
param set-default CA_ROTOR0_KM  0.05
# ... lặp lại cho tất cả motor

param set-default PWM_MAIN_FUNC1 101
param set-default PWM_MAIN_FUNC2 102
# ...
```

**MAVLink airframe type (`MAV_TYPE`):**

| Số | Loại |
|----|------|
| 0 | Generic micro air vehicle |
| 1 | Fixed wing |
| 2 | Quadrotor |
| 3 | Coaxial helicopter |
| 13 | Hexarotor |
| 14 | Octorotor |
| 15 | Tricopter |
| 19 | VTOL Two-rotor Tailsitter |
| 22 | VTOL Standard |

### Bước 16 — Đăng ký file HITL trong CMakeLists.txt

Mở file `CMakeLists.txt` trong cùng folder, tìm section `if(CONFIG_MODULES_SIMULATION_PWM_OUT_SIM)` và thêm tên file `.hil`:

```cmake
if(CONFIG_MODULES_SIMULATION_PWM_OUT_SIM)
  px4_add_romfs_files(
    # [1000, 1999] Simulation setups
    1001_rc_quad_x.hil
    1002_standard_vtol.hil
    1103_octorotor.hil     # thêm vào đây
  )
endif()
```

### Bước 17 — Tạo world file cho HITL

Vào folder:
```
PX4-Autopilot/Tools/simulation/gazebo-classic/sitl_gazebo-classic/worlds/
```

Copy file `hitl_iris.world`, đổi tên thành `hitl_modelname.world`.  
Trong file mới, tìm dòng:

```xml
<uri>model://iris_hitl</uri>
```

Đổi `iris_hitl` thành tên folder **B_hitl** đã tạo ở Bước 14.

### Bước 18 — Build firmware cho Pixhawk

Tra cứu lệnh build cho đúng phiên bản Pixhawk tại:
```
https://docs.px4.io/main/en/dev_setup/building_px4.html
```

Ví dụ với Pixhawk 6C:
```bash
make px4_fmu-v6c_default upload
```

> Thêm `upload` vào cuối lệnh để tự động upload lên Pixhawk sau khi build.

### Bước 19 — Kiểm tra file HITL sau build

Sau khi build, kiểm tra file `.hil` đã tồn tại chưa:
```
PX4-Autopilot/build/px4_fmu-v6c_default/etc/init.d/airframes/
```

Nếu thấy file `ID_modelname.hil` → tiếp tục Bước 20.  
Nếu không thấy → quay lại kiểm tra từ Bước 13.

### Bước 20 — Xác nhận HITL setup hoàn tất

Vào folder:
```
PX4-Autopilot/build/px4_fmu-v6c_default/
```

Mở file `airframes.xml`, kiểm tra có entry tương tự sau không:

```xml
<airframe name="HIL Octocopter X" id="1103" maintainer="...">
  <class>Copter</class>
  <type>Simulation</type>
</airframe>
```

Nếu có → setup HITL thành công.

### Bước 21 — Chạy HITL với Gazebo Classic

```bash
# Terminal 1 — khởi động Gazebo
cd PX4-Autopilot
source Tools/simulation/gazebo-classic/setup_gazebo.bash $(pwd) $(pwd)/build/px4_sitl_default
gazebo Tools/simulation/gazebo-classic/sitl_gazebo-classic/worlds/hitl_modelname.world
```

Kết nối Pixhawk với laptop qua USB trước khi chạy.

### Bước 22 — Cấu hình QGroundControl

1. Kết nối QGroundControl với PX4
2. Vào **Parameters**, tìm **SYS_AUTOSTART**
3. Nhập **ID** của file tạo ở Bước 15 (ví dụ: `1103`)
4. Kiểm tra **MAV_TYPE** đã cập nhật đúng loại vehicle chưa

### Bước 23 — Cấu hình Actuator trong QGroundControl

1. Vào section **Actuator**
2. Nhập khoảng cách từ tâm model đến tâm motor theo **X (PX)** và **Y (PY)**
3. Trong tab **HIL**, gán từng channel tương ứng với motor

Ví dụ với octorotor coaxial (8 motor):

| Motor | Position X | Position Y | CCW |
|-------|-----------|-----------|-----|
| Motor 1 | 0.19 | 0.19 | ✓ |
| Motor 2 | 0.19 | -0.19 | |
| Motor 3 | -0.19 | -0.19 | ✓ |
| Motor 4 | -0.19 | 0.19 | |
| Motor 5 | 0.19 | -0.19 | ✓ |
| Motor 6 | 0.19 | 0.19 | |
| Motor 7 | -0.19 | 0.19 | ✓ |
| Motor 8 | -0.19 | -0.19 | |

### Bước 24 — Chạy HITL

Sau khi hoàn tất các bước trên, chạy HITL như bình thường.

---

## Tóm tắt file structure

```
PX4-Autopilot/
├── ROMFS/px4fmu_common/
│   ├── init.d-posix/airframes/
│   │   ├── CMakeLists.txt                          ← thêm ID_gazebo-classic_modelname
│   │   └── 22000_gazebo-classic_octorotor_coaxial  ← tạo mới (SITL)
│   └── init.d/airframes/
│       ├── CMakeLists.txt                          ← thêm ID_modelname.hil
│       └── 1103_octorotor.hil                      ← tạo mới (HITL)
├── src/modules/simulation/simulator_mavlink/
│   └── sitl_targets_gazebo-classic                 ← thêm modelname
└── Tools/simulation/gazebo-classic/sitl_gazebo-classic/
    ├── models/
    │   ├── octorotor_coaxial/                      ← folder B (SITL)
    │   │   ├── model.config
    │   │   ├── octorotor_coaxial.sdf
    │   │   └── meshes/
    │   └── octorotor_coaxial_hitl/                 ← folder B_hitl (HITL)
    │       ├── model.config
    │       └── octorotor_coaxial_hitl.sdf
    └── worlds/
        └── hitl_octorotor_coaxial.world            ← tạo mới (HITL)
```
