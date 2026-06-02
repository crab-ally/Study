# Gazebo Classic

## 1. SDF XML (.world) 개요

Gazebo가 실행될 전체 환경을 정의하는 파일 형식입니다.

```
<sdf>
  <world>
    <gravity .../>           ← 전역 물리 설정
    <include .../>           ← 외부 모델 포함 (ground_plane, sun 등)
    <model>                  ← 하나의 물체/로봇 단위
      <static/>
      <pose/>
      <link>                 ← 움직이는 몸체
        <inertial/>          ← 질량·관성
        <collision/>         ← 충돌 영역
        <visual/>            ← 시각 형상
      </link>
      <joint>                ← 링크 간 관절
        <parent/> <child/>
        <axis/>
      </joint>
    </model>
  </world>
</sdf>
```

---

## 2. `<model>` 속성

| 태그 | 설명 |
|------|------|
| `<static>true</static>` | 위치 고정 (물리 연산 제외) |
| `<pose>x y z roll pitch yaw</pose>` | 월드 기준 모델의 위치와 자세 [m, rad] |

---

## 3. `<link>` — 몸체 정의

링크는 하나의 강체(Rigid Body)입니다.
`<inertial>`, `<collision>`, `<visual>` 세 가지 구성 요소로 이루어집니다.

### 3-1. `<inertial>` — 질량·관성

> 관성: 회전시키려 할 때 얼마나 저항하는가 (질량의 회전 버전)

| 태그 | 설명 |
|------|------|
| `<mass>` | 질량 [kg] |
| `<inertia>` | 회전 관성 행렬 |

`<inertia>` 내부 태그:

| 태그 | 설명 |
|------|------|
| `<ixx>`, `<iyy>`, `<izz>` | x·y·z축 기준 회전 관성 [kg·m²] |
| `<ixy>`, `<ixz>`, `<iyz>` | 축 사이의 관성 결합 항 (대칭 형상이면 보통 0) |

### 3-2. `<collision>` — 충돌 영역

물리 엔진이 충돌 계산에 사용하는 형상입니다. `<visual>` 과 별도로 정의할 수 있습니다.

| 태그 | 설명 |
|------|------|
| `<geometry>` | 충돌 형상 정의 |
| `<surface>` | 충돌 면의 물리적 특성 |

**`<surface>` 하위 구조:**

```xml
<surface>
  <friction>          <!-- 마찰 속성 설정 -->
    <ode>             <!-- ODE 물리 엔진 설정 -->
      <mu>0.5</mu>    <!-- 접촉면의 주 접선 방향 마찰 계수 -->
      <mu2>0.5</mu2>  <!-- 접촉면의 보조 접선 방향 마찰 계수 -->
    </ode>
  </friction>
  <bounce>            <!-- 충돌 후 튀는 정도 설정 -->
    <restitution_coefficient>0.9</restitution_coefficient>  <!-- 반발 계수 (0~1) -->
    <threshold>0.01</threshold>  <!-- 반발 적용 최소 속도 [m/s] -->
  </bounce>
</surface>
```

### 3-3. `<visual>` — 시각 형상

화면에 렌더링되는 형상입니다.

| 태그 | 설명 |
|------|------|
| `<geometry>` | 시각 형상 정의 |
| `<material>` | 색상·재질 설정 |

**`<material>` 태그:**

| 태그 | 설명 |
|------|------|
| `<ambient>R G B A</ambient>` | 주변광에서 보이는 색 |
| `<diffuse>R G B A</diffuse>` | 직접 조명을 받을 때 보이는 색 |

### 3-4. `<geometry>` — 형상 종류

`<collision>` 과 `<visual>` 양쪽에서 동일하게 사용합니다.

| 형상 | 태그 구조 | 주요 속성 |
|------|-----------|-----------|
| 박스 | `<box><size>x y z</size></box>` | 전체 크기 [m] |
| 구 | `<sphere><radius>r</radius></sphere>` | 반지름 [m] |
| 원기둥 | `<cylinder><radius>r</radius><length>l</length></cylinder>` | 반지름·길이 [m] |

---

## 4. `<joint>` — 관절 정의

두 링크를 연결하고 상대 운동을 정의합니다.

| 태그/속성 | 설명 |
|-----------|------|
| `type="fixed"` | 고정 관절 |
| `type="revolute"` | 회전 관절 |
| `type="prismatic"` | 직선 이동 관절 |
| `<parent>`, `<child>` | 부모·자식 링크 이름 |
| `<pose>x y z roll pitch yaw</pose>` | 부모 링크 기준 관절의 위치와 자세 |
| `<axis>` | 회전·이동축 정보 |

### 4-1. `<axis>` 하위 태그

| 태그 | 설명 |
|------|------|
| `<xyz>x y z</xyz>` | 회전·이동 축 방향 단위 벡터 |
| `<limit>` | 관절 움직임 범위 제한 |

**`<limit>` 태그:**

| 태그 | 설명 |
|------|------|
| `<lower>`, `<upper>` | 최솟값·최댓값 [revolute: rad / prismatic: m] |
| `<effort>` | 최대 힘·토크 제한 [N 또는 N·m] |
| `<velocity>` | 최대 속도 제한 [m/s 또는 rad/s] |

---

## 5. 외부 모델 포함

```xml
<include>
  <uri>model://ground_plane</uri>  <!-- 기본 제공 바닥 -->
</include>

<include>
  <uri>model://sun</uri>           <!-- 기본 제공 태양광 -->
</include>
```

> `model://` 접두사는 Gazebo 모델 경로(`~/.gazebo/models` 등)를 가리킵니다.

---

## 6. 전체 예제

```xml
<?xml version="1.0" ?>
<sdf version="1.6">
  <world name="example_world">

    <gravity>0 0 -9.81</gravity>

    <include><uri>model://ground_plane</uri></include>
    <include><uri>model://sun</uri></include>

    <!-- 박스 -->
    <model name="red_box">
      <pose>0 0 0.5 0 0 0</pose>
      <link name="box_link">
        <inertial>
          <mass>0.2</mass>
          <inertia>
            <ixx>0.1667</ixx><iyy>0.1667</iyy><izz>0.1667</izz>
            <ixy>0</ixy><ixz>0</ixz><iyz>0</iyz>
          </inertia>
        </inertial>
        <collision name="box_collision">
          <surface>
            <friction><ode><mu>0.01</mu><mu2>0.01</mu2></ode></friction>
            <bounce>
              <restitution_coefficient>0.9</restitution_coefficient>
              <threshold>0.01</threshold>
            </bounce>
          </surface>
          <geometry><box><size>1 1 1</size></box></geometry>
        </collision>
        <visual name="box_visual">
          <geometry><box><size>1 1 1</size></box></geometry>
          <material><ambient>1 0 0 1</ambient><diffuse>1 0 0 1</diffuse></material>
        </visual>
      </link>
    </model>

    <!-- 구 -->
    <model name="blue_sphere">
      <pose>3 0 0.5 0 0 0</pose>
      <link name="sphere_link">
        <collision name="sphere_collision">
          <geometry><sphere><radius>0.5</radius></sphere></geometry>
        </collision>
        <visual name="sphere_visual">
          <geometry><sphere><radius>0.5</radius></sphere></geometry>
          <material><ambient>0 0 1 1</ambient><diffuse>0 0 1 1</diffuse></material>
        </visual>
      </link>
    </model>

    <!-- 원기둥 -->
    <model name="green_cylinder">
      <pose>6 0 0.5 0 0 0</pose>
      <link name="cylinder_link">
        <collision name="cylinder_collision">
          <geometry><cylinder><radius>0.3</radius><length>1.0</length></cylinder></geometry>
        </collision>
        <visual name="cylinder_visual">
          <geometry><cylinder><radius>0.3</radius><length>1.0</length></cylinder></geometry>
          <material><ambient>0 1 0 1</ambient><diffuse>0 1 0 1</diffuse></material>
        </visual>
      </link>
    </model>

    <!-- 고정 벽 (static) -->
    <model name="static_wall">
      <static>true</static>
      <pose>2 0 0.5 0 0 0</pose>
      <link name="wall_link">
        <collision name="wall_collision">
          <geometry><box><size>0.2 3.0 1.0</size></box></geometry>
        </collision>
        <visual name="wall_visual">
          <geometry><box><size>0.2 3.0 1.0</size></box></geometry>
          <material><ambient>0.6 0.6 0.6 1</ambient><diffuse>0.6 0.6 0.6 1</diffuse></material>
        </visual>
      </link>
    </model>

    <!-- 고정 관절 -->
    <model name="fixed_joint_object">
      <pose>0 3 0.5 0 0 0</pose>
      <link name="base_link">
        <collision name="base_collision">
          <geometry><box><size>1 1 0.3</size></box></geometry>
        </collision>
        <visual name="base_visual">
          <geometry><box><size>1 1 0.3</size></box></geometry>
          <material><ambient>0.2 0.2 0.8 1</ambient><diffuse>0.2 0.2 0.8 1</diffuse></material>
        </visual>
      </link>
      <link name="top_link">
        <pose>0 0 0.65 0 0 0</pose>
        <collision name="top_collision">
          <geometry><box><size>0.5 0.5 1.0</size></box></geometry>
        </collision>
        <visual name="top_visual">
          <geometry><box><size>0.5 0.5 1.0</size></box></geometry>
          <material><ambient>0.8 0.2 0.2 1</ambient><diffuse>0.8 0.2 0.2 1</diffuse></material>
        </visual>
      </link>
      <joint name="base_to_top" type="fixed">
        <parent>base_link</parent>
        <child>top_link</child>
      </joint>
    </model>

    <!-- 회전 관절 -->
    <model name="revolute_arm">
      <pose>0 6 0.2 0 0 0</pose>
      <link name="base_link">
        <collision name="base_collision">
          <geometry><box><size>0.6 0.6 0.2</size></box></geometry>
        </collision>
        <visual name="base_visual">
          <geometry><box><size>0.6 0.6 0.2</size></box></geometry>
          <material><ambient>0.2 0.2 0.2 1</ambient><diffuse>0.2 0.2 0.2 1</diffuse></material>
        </visual>
      </link>
      <link name="arm_link">
        <pose>0.6 0 0.2 0 0 0</pose>
        <inertial><mass>1.0</mass></inertial>
        <collision name="arm_collision">
          <geometry><box><size>1.0 0.15 0.15</size></box></geometry>
        </collision>
        <visual name="arm_visual">
          <geometry><box><size>1.0 0.15 0.15</size></box></geometry>
          <material><ambient>0.9 0.5 0.1 1</ambient><diffuse>0.9 0.5 0.1 1</diffuse></material>
        </visual>
      </link>
      <joint name="base_to_arm" type="revolute">
        <parent>base_link</parent>
        <child>arm_link</child>
        <pose>0.3 0 0.2 0 0 0</pose>   <!-- 부모 기준 회전축 위치 -->
        <axis>
          <xyz>0 0 1</xyz>
          <limit><lower>-1.57</lower><upper>1.57</upper><effort>10</effort><velocity>2</velocity></limit>
        </axis>
      </joint>
    </model>

    <!-- 직선 이동 관절 -->
    <model name="prismatic_slider">
      <pose>0 9 0.2 0 0 0</pose>
      <link name="base_link">
        <collision name="base_collision">
          <geometry><box><size>1.0 0.3 0.2</size></box></geometry>
        </collision>
        <visual name="base_visual">
          <geometry><box><size>1.0 0.3 0.2</size></box></geometry>
          <material><ambient>0.2 0.2 0.2 1</ambient><diffuse>0.2 0.2 0.2 1</diffuse></material>
        </visual>
      </link>
      <link name="slider_link">
        <pose>0 0 0.35 0 0 0</pose>
        <inertial><mass>0.5</mass></inertial>
        <collision name="slider_collision">
          <geometry><box><size>0.3 0.3 0.2</size></box></geometry>
        </collision>
        <visual name="slider_visual">
          <geometry><box><size>0.3 0.3 0.2</size></box></geometry>
          <material><ambient>0.1 0.8 0.3 1</ambient><diffuse>0.1 0.8 0.3 1</diffuse></material>
        </visual>
      </link>
      <joint name="base_to_slider" type="prismatic">
        <parent>base_link</parent>
        <child>slider_link</child>
        <axis>
          <xyz>1 0 0</xyz>
          <limit><lower>-0.4</lower><upper>0.4</upper><effort>20</effort><velocity>1</velocity></limit>
        </axis>
      </joint>
    </model>

  </world>
</sdf>
```

---

## URDF (.urdf)

로봇을 정의하는 형식

```xml
<?xml version="1.0"?>
<robot name="ros_camera_robot">

  <link name="base_link">
    <visual>
      <origin xyz="0 0 0.15" rpy="0 0 0"/>
      <geometry>
        <box size="1.0 0.6 0.3"/>
      </geometry>
    </visual>

    <collision>
      <origin xyz="0 0 0.15" rpy="0 0 0"/>
      <geometry>
        <box size="1.0 0.6 0.3"/>
      </geometry>
    </collision>

    <inertial>
      <origin xyz="0 0 0.15" rpy="0 0 0"/>
      <mass value="3.0"/>
      <inertia ixx="0.2" ixy="0" ixz="0" iyy="0.2" iyz="0" izz="0.2"/>
    </inertial>
  </link>

  <link name="camera_link">
    <visual>
      <origin xyz="0 0 0.05" rpy="0 0 0"/>
      <geometry>
        <box size="0.15 0.08 0.08"/>
      </geometry>
    </visual>

    <collision>
      <origin xyz="0 0 0.05" rpy="0 0 0"/>
      <geometry>
        <box size="0.15 0.08 0.08"/>
      </geometry>
    </collision>

    <inertial>
      <origin xyz="0 0 0.05" rpy="0 0 0"/>
      <mass value="0.1"/>
      <inertia ixx="0.001" ixy="0" ixz="0" iyy="0.001" iyz="0" izz="0.001"/>
    </inertial>
  </link>

  <joint name="base_to_camera_joint" type="fixed">
    <parent link="base_link"/>
    <child link="camera_link"/>
    <origin xyz="0.45 0 0.35" rpy="0 0 0"/>
  </joint>

  <!-- URDF 표준에는 없는 Gazebo 전용 확장 태그 -->
  <gazebo reference="base_link">
    <material>Gazebo/Blue</material>
  </gazebo>

  <!-- 카메라 센서 - 색상 정보 제공 -->
  <gazebo reference="camera_link">
    <material>Gazebo/Black</material>

    <sensor name="front_camera" type="camera">
      <always_on>true</always_on>
      <update_rate>30</update_rate>
      <visualize>true</visualize>

      <camera>
        <horizontal_fov>1.047</horizontal_fov>
        <image>
          <width>640</width>
          <height>480</height>
          <format>R8G8B8</format>
        </image>
        <clip>
          <near>0.1</near>
          <far>100.0</far>
        </clip>
      </camera>

      <!-- ROS2 토픽 발행을 위한 플러그인 -->
      <plugin name="front_camera_controller" filename="libgazebo_ros_camera.so">
        <ros>
          <!-- ROS2 토픽 네임스페이스 -->
          <namespace>/camera</namespace>
        </ros>
        <camera_name>front_camera</camera_name>
        <frame_name>camera_link</frame_name>
      </plugin>
    </sensor>
  </gazebo>

  <gazebo reference="camera_link">
    <material>Gazebo/Black</material>

    <!-- Depth 카메라 센서 - 색상 + 거리 정보 제공 -->
    <sensor name="depth_camera" type="depth">
      <always_on>true</always_on>
      <update_rate>30</update_rate>
      <visualize>true</visualize>

      <camera>
        <horizontal_fov>1.047</horizontal_fov>
        <image>
          <width>640</width>
          <height>480</height>
          <format>R8G8B8</format>
        </image>
        <clip>
          <near>0.1</near>
          <far>10.0</far>
        </clip>
      </camera>

      <plugin name="depth_camera_controller" filename="libgazebo_ros_camera.so">
        <ros>
          <namespace>/depth_camera</namespace>
        </ros>
        <camera_name>depth</camera_name>
        <frame_name>camera_link</frame_name>
      </plugin>
    </sensor>
  </gazebo>

  <gazebo reference="laser_link">
    <material>Gazebo/Black</material>

    <!-- Ray 기반 2D LiDAR 센서 -->
    <sensor name="laser_sensor" type="ray">
      <always_on>true</always_on>
      <visualize>true</visualize>
      <update_rate>10</update_rate>

      <ray>
        <scan>
          <horizontal>
            <samples>360</samples>
            <resolution>1</resolution>
            <min_angle>-3.14159</min_angle>
            <max_angle>3.14159</max_angle>
          </horizontal>
        </scan>

        <range>
          <min>0.12</min>
          <max>8.0</max>
          <resolution>0.01</resolution>
        </range>

        <!-- 노이즈 추가 - 실제 센서와 유사한 환경 구현 -->
        <noise>
          <type>gaussian</type>
          <mean>0.0</mean>
          <stddev>0.01</stddev> <!-- 표준편차 -->
        </noise>
      </ray>

      <plugin name="laser_controller" filename="libgazebo_ros_ray_sensor.so">
        <ros>
          <namespace>/scan</namespace>
          <remapping>~/out:=scan</remapping>
        </ros>
        <output_type>sensor_msgs/LaserScan</output_type>
        <frame_name>laser_link</frame_name>
      </plugin>
    </sensor>
  </gazebo>

  <gazebo reference="base_link">
    <material>Gazebo/Blue</material>

    <!-- IMU 센서 - 로봇의 자세 + 가속도 + 각속도 정보 제공 -->
    <sensor name="imu_sensor" type="imu">
      <always_on>true</always_on>
      <update_rate>100</update_rate>
      <visualize>true</visualize>

      <imu>
        <angular_velocity>
          <x>
            <noise type="gaussian">
              <mean>0.0</mean>
              <stddev>0.0002</stddev>
            </noise>
          </x>
          <y>
            <noise type="gaussian">
              <mean>0.0</mean>
              <stddev>0.0002</stddev>
            </noise>
          </y>
          <z>
            <noise type="gaussian">
              <mean>0.0</mean>
              <stddev>0.0002</stddev>
            </noise>
          </z>
        </angular_velocity>

        <linear_acceleration>
          <x>
            <noise type="gaussian">
              <mean>0.0</mean>
              <stddev>0.01</stddev>
            </noise>
          </x>
          <y>
            <noise type="gaussian">
              <mean>0.0</mean>
              <stddev>0.01</stddev>
            </noise>
          </y>
          <z>
            <noise type="gaussian">
              <mean>0.0</mean>
              <stddev>0.01</stddev>
            </noise>
          </z>
        </linear_acceleration>
      </imu>

      <plugin name="imu_plugin" filename="libgazebo_ros_imu_sensor.so">
        <ros>
          <namespace>/imu</namespace>
          <!-- 토픽 이름 변경 -->
          <remapping>~/out:=data</remapping>
        </ros>
        <frame_name>base_link</frame_name>
      </plugin>
    </sensor>
  </gazebo>

  <gazebo reference="front_bumper_link">
    <material>Gazebo/Orange</material>

    <!-- Contact Sensor - 충돌이나 접촉을 감지 -->
    <sensor name="front_contact_sensor" type="contact">
      <always_on>true</always_on>
      <update_rate>50</update_rate>

      <contact>
        <collision>front_bumper_collision</collision>
      </contact>

      <plugin name="front_contact_plugin" filename="libgazebo_ros_bumper.so">
        <ros>
          <namespace>/bumper</namespace>
          <remapping>bumper_states:=states</remapping>
        </ros>
        <frame_name>front_bumper_link</frame_name>
      </plugin>
    </sensor>
  </gazebo>

</robot>
```

| 태그/속성 | 설명 |
|-----------|------|
| `<origin>` | 부모 좌표계 기준 해당 요소의 위치(xyz)와 방향(rpy) 정의 |
| `<color>` | 색상 설정 |
| `<sensor>` | 센서 설정 |
| `<always_on>` | 센서가 항상 켜져 있는지 여부 |
| `<update_rate>` | 센서 업데이트 주기 (초당 프레임 수) |
| `<visualize>` | 센서 시각화 여부 (Gazebo GUI에서 센서 영상 확인 여부) |
| `<camera>` | 카메라 설정 |
| `<horizontal_fov>` | 카메라의 수평 시야각 [rad] |
| `<image>` | 이미지 설정 |
| `<width>`, `<height>` | 이미지 해상도 |
| `<format>` | 이미지 포맷 (R8G8B8 - RGB 8비트 컬러 이미지) |
| `<clip>` | 카메라 클리핑 설정 |
| `<near>`, `<far>` | 카메라 최소·최대 거리 [m] |
| `<update_rate>` | 센서 업데이트 주기 (초당 프레임 수) |
| `<ray>` | 레이저 설정 |
| `<scan>` | 몇 방향으로 측정할지 설정 |
| `<horizontal>` | 수평 방향 설정 |
| `<samples>` | 레이저 발사 개수 |
| `<resolution>` | 샘플 당 출력값 개수 |
| `<min_angle>`, `<max_angle>` | 레이저 발사 각도 범위 [rad] |
| `<range>` | 레이저 측정 거리 범위 [m] |
| `<min>`, `<max>` | 레이저 최소·최대 거리 [m] |
| `<resolution>` | 거리 정밀도 [m] |

---

## Xacro (.xacro)

```xml
<?xml version="1.0"?>
<robot xmlns:xacro="http://www.ros.org/wiki/xacro" name="box_robot">

  <!-- 변수 선언 -->
  <xacro:property name="base_length" value="1.0"/>
  <xacro:property name="base_width" value="0.6"/>
  <xacro:property name="base_height" value="0.3"/>
  <xacro:property name="base_mass" value="2.0"/>

  <link name="base_link">
    <visual>
      <origin xyz="0 0 ${base_height / 2}" rpy="0 0 0"/>
      <geometry>
        <box size="${base_length} ${base_width} ${base_height}"/>
      </geometry>
    </visual>

    <collision>
      <origin xyz="0 0 ${base_height / 2}" rpy="0 0 0"/>
      <geometry>
        <box size="${base_length} ${base_width} ${base_height}"/>
      </geometry>
    </collision>

    <inertial>
      <origin xyz="0 0 ${base_height / 2}" rpy="0 0 0"/>
      <mass value="${base_mass}"/>
      <inertia ixx="0.15" ixy="0" ixz="0" iyy="0.20" iyz="0" izz="0.25"/>
    </inertial>
  </link>

  <gazebo reference="base_link">
    <material>Gazebo/Blue</material>
  </gazebo>

</robot>
```

```xml
<?xml version="1.0"?>
<robot xmlns:xacro="http://www.ros.org/wiki/xacro" name="macro_box_robot">

  <!-- 함수 선언 -->
  <xacro:macro name="box_link" params="link_name length width height mass color">
    <link name="${link_name}">
      <visual>
        <origin xyz="0 0 ${height / 2}" rpy="0 0 0"/>
        <geometry>
          <box size="${length} ${width} ${height}"/>
        </geometry>
      </visual>

      <collision>
        <origin xyz="0 0 ${height / 2}" rpy="0 0 0"/>
        <geometry>
          <box size="${length} ${width} ${height}"/>
        </geometry>
      </collision>

      <inertial>
        <origin xyz="0 0 ${height / 2}" rpy="0 0 0"/>
        <mass value="${mass}"/>
        <inertia ixx="0.1" ixy="0" ixz="0" iyy="0.1" iyz="0" izz="0.1"/>
      </inertial>
    </link>

    <gazebo reference="${link_name}">
      <material>${color}</material>
    </gazebo>
  </xacro:macro>

  <!-- 함수 호출 -->
  <xacro:box_link
    link_name="base_link"
    length="1.0"
    width="0.6"
    height="0.3"
    mass="2.0"
    color="Gazebo/Blue"/>

  <xacro:box_link
    link_name="sensor_link"
    length="0.3"
    width="0.2"
    height="0.2"
    mass="0.5"
    color="Gazebo/Green"/>

  <joint name="base_to_sensor_joint" type="fixed">
    <parent link="base_link"/>
    <child link="sensor_link"/>
    <origin xyz="0.3 0 0.3" rpy="0 0 0"/>
  </joint>

</robot>
```