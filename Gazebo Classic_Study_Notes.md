# Gazebo Classic

---

## 0. 전체 워크플로우

```
ROS2 패키지 생성
  → 로봇 모델 작성 (URDF / Xacro)
  → 월드 파일 작성 (SDF .world)  ← 선택
  → launch 파일 작성
  → setup.py 수정               ← URDF·launch 파일을 패키지에 포함
  → colcon build
  → source install/setup.bash
  → ros2 launch [패키지명] [launch파일명]
```

setup.py: Python 패키지의 빌드·설치 설정 파일.

---

## 1. 파일 형식 비교

SDF (.world)  : Gazebo 전체 환경(월드) 정의
URDF (.urdf)  : 로봇 구조 정의 (ROS 표준)
Xacro (.xacro): URDF의 매크로 확장판 (변수·함수 지원)

---

## 2. SDF (.world) — 월드 정의

### 2-1. 전체 구조

```xml
<sdf version="1.6">
  <world name="...">

    <!-- 전역 물리 설정 -->
    <gravity>0 0 -9.81</gravity>

    <!-- 외부 모델 포함 -->
    <include><uri>model://ground_plane</uri></include>
    <include><uri>model://sun</uri></include>

    <model name="...">
      <static/>               <!-- 위치 고정 여부 -->
      <pose/>                 <!-- 월드 기준 위치·자세 -->
      <link name="...">
        <inertial/>           <!-- 질량·관성 -->
        <collision/>          <!-- 충돌 영역 -->
        <visual/>             <!-- 시각 형상 -->
      </link>
      <joint name="..." type="..."> <!-- 링크 간 관절 -->
        <parent/> <child/>
        <axis/>
      </joint>
    </model>

  </world>
</sdf>
```

---

### 2-2. <model> 속성

```xml
<static>true</static>              : 위치 고정 — 물리 연산 제외
<pose>x y z roll pitch yaw</pose>  : 월드 기준 위치·자세 [m, rad]
```

---

### 2-3. <link> — 강체(Rigid Body) 정의

링크 하나 = 강체 하나. inertial, collision, visual 세 요소로 구성.

#### inertial — 질량·관성

관성(Inertia): 물체를 회전시키려 할 때의 저항 크기 (질량의 회전 버전)

<mass>             : 질량 [kg]
<ixx> <iyy> <izz>  : x·y·z축 기준 회전 관성 [kg·m²]
<ixy> <ixz> <iyz>  : 축 간 관성 결합항 — 대칭 형상이면 보통 0

#### collision — 충돌 영역

물리 엔진이 충돌 계산에 사용. visual과 독립적으로 정의 가능.

<collision name="...">
  <geometry> ... </geometry>
  <surface>
    <friction>
      <ode>
        <mu>0.5</mu>    <!-- 주 접선 방향 마찰 계수 -->
        <mu2>0.5</mu2>  <!-- 보조 접선 방향 마찰 계수 -->
      </ode>
    </friction>
    <bounce>
      <restitution_coefficient>0.9</restitution_coefficient>  <!-- 반발 계수 (0~1) -->
      <threshold>0.01</threshold>  <!-- 반발 적용 최소 속도 [m/s] -->
    </bounce>
  </surface>
</collision>

#### visual — 시각 형상

화면에 렌더링되는 형상.

<visual name="...">
  <geometry> ... </geometry>
  <material>
    <ambient>R G B A</ambient>   <!-- 주변광에서 보이는 색 -->
    <diffuse>R G B A</diffuse>   <!-- 직접 조명을 받을 때 보이는 색 -->
  </material>
</visual>

#### geometry — 형상 종류 (collision · visual 공통)

박스   : <box><size>x y z</size></box>                             전체 크기 [m]
구     : <sphere><radius>r</radius></sphere>                       반지름 [m]
원기둥 : <cylinder><radius>r</radius><length>l</length></cylinder> 반지름·길이 [m]

---

### 2-4. <joint> — 관절 정의

두 링크를 연결하고 상대 운동을 정의.

fixed     : 고정 관절
revolute  : 회전 관절
prismatic : 직선 이동 관절

<joint name="..." type="revolute">
  <parent>링크명</parent>
  <child>링크명</child>
  <pose>x y z roll pitch yaw</pose>  <!-- 부모 기준 관절 위치·자세 -->
  <axis>
    <xyz>0 0 1</xyz>                 <!-- 회전·이동 축 방향 단위벡터 -->
    <limit>
      <lower>-1.57</lower>           <!-- 최솟값 [revolute: rad / prismatic: m] -->
      <upper>1.57</upper>
      <effort>10</effort>            <!-- 최대 힘·토크 [N 또는 N·m] -->
      <velocity>2</velocity>         <!-- 최대 속도 [m/s 또는 rad/s] -->
    </limit>
  </axis>
</joint>

---

### 2-5. 전체 예제

<?xml version="1.0" ?>
<sdf version="1.6">
  <world name="example_world">

    <gravity>0 0 -9.81</gravity>
    <include><uri>model://ground_plane</uri></include>
    <include><uri>model://sun</uri></include>

    <!-- ① 박스 (낮은 마찰 + 높은 반발) -->
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

    <!-- ② 구 -->
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

    <!-- ③ 원기둥 -->
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

    <!-- ④ 정적 벽 (static) -->
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

    <!-- ⑤ 고정 관절 (fixed) -->
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

    <!-- ⑥ 회전 관절 (revolute) — z축 ±90° -->
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
        <pose>0.3 0 0.2 0 0 0</pose>
        <axis>
          <xyz>0 0 1</xyz>
          <limit><lower>-1.57</lower><upper>1.57</upper><effort>10</effort><velocity>2</velocity></limit>
        </axis>
      </joint>
    </model>

    <!-- ⑦ 직선 이동 관절 (prismatic) — x축 ±0.4m -->
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

---

## 3. URDF (.urdf) — 로봇 정의

### 3-1. 기본 구조

<?xml version="1.0"?>
<robot name="ros_camera_robot">

  <link name="base_link"> ... </link>
  <link name="camera_link"> ... </link>

  <joint name="base_to_camera_joint" type="fixed">
    <parent link="base_link"/>
    <child link="camera_link"/>
    <origin xyz="0.45 0 0.35" rpy="0 0 0"/>
  </joint>

  <!-- Gazebo 전용 확장 태그 (URDF 표준에 없음) -->
  <gazebo reference="링크명"> ... </gazebo>

</robot>

링크 구성요소 태그:
<origin xyz="..." rpy="..."/>  : 부모 좌표계 기준 위치(xyz)·방향(rpy)
<visual>                       : 시각 형상
<collision>                    : 충돌 형상
<inertial>                     : 질량·관성

---

### 3-2. Gazebo 센서 확장 태그 (<gazebo reference="...">)

URDF 표준에는 없는 Gazebo 전용 태그. 특정 링크에 센서·재질을 붙일 때 사용.

공통 센서 속성:
<always_on>   : 센서 항상 활성화 여부
<update_rate> : 업데이트 주기 [Hz]
<visualize>   : Gazebo GUI에서 센서 시각화 여부

---

#### ① 일반 카메라 (type="camera") — RGB 이미지

<gazebo reference="camera_link">
  <material>Gazebo/Black</material>
  <sensor name="front_camera" type="camera">
    <always_on>true</always_on>
    <update_rate>30</update_rate>
    <visualize>true</visualize>
    <camera>
      <horizontal_fov>1.047</horizontal_fov>   <!-- 수평 시야각 [rad] -->
      <image>
        <width>640</width>
        <height>480</height>
        <format>R8G8B8</format>                <!-- RGB 8비트 -->
      </image>
      <clip>
        <near>0.1</near>                       <!-- 최소 렌더 거리 [m] -->
        <far>100.0</far>                       <!-- 최대 렌더 거리 [m] -->
      </clip>
    </camera>
    <plugin name="front_camera_controller" filename="libgazebo_ros_camera.so">
      <ros><namespace>/camera</namespace></ros>
      <camera_name>front_camera</camera_name>
      <frame_name>camera_link</frame_name>
    </plugin>
  </sensor>
</gazebo>

---

#### ② 깊이 카메라 (type="depth") — RGB + 거리

일반 카메라와 구조 동일. type="depth"로 변경하고 <far>를 짧게 설정.

<sensor name="depth_camera" type="depth">
  ...
  <clip><near>0.1</near><far>10.0</far></clip>   <!-- depth는 보통 근거리 -->
  <plugin name="depth_camera_controller" filename="libgazebo_ros_camera.so">
    <ros><namespace>/depth_camera</namespace></ros>
    <camera_name>depth</camera_name>
    <frame_name>camera_link</frame_name>
  </plugin>
</sensor>

---

#### ③ 2D LiDAR (type="ray") — 레이저 거리 측정

<sensor name="laser_sensor" type="ray">
  <always_on>true</always_on>
  <visualize>true</visualize>
  <update_rate>10</update_rate>
  <ray>
    <scan>
      <horizontal>
        <samples>360</samples>          <!-- 레이저 발사 개수 -->
        <resolution>1</resolution>      <!-- 샘플당 출력값 수 -->
        <min_angle>-3.14159</min_angle> <!-- 스캔 시작 각도 [rad] -->
        <max_angle>3.14159</max_angle>  <!-- 스캔 끝 각도 [rad] -->
      </horizontal>
    </scan>
    <range>
      <min>0.12</min>                   <!-- 최소 측정 거리 [m] -->
      <max>8.0</max>                    <!-- 최대 측정 거리 [m] -->
      <resolution>0.01</resolution>     <!-- 거리 정밀도 [m] -->
    </range>
    <noise>
      <type>gaussian</type>
      <mean>0.0</mean>
      <stddev>0.01</stddev>             <!-- 실제 센서 노이즈 모사 -->
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

---

#### ④ IMU (type="imu") — 자세·가속도·각속도

<sensor name="imu_sensor" type="imu">
  <always_on>true</always_on>
  <update_rate>100</update_rate>
  <imu>
    <angular_velocity>
      <x><noise type="gaussian"><mean>0.0</mean><stddev>0.0002</stddev></noise></x>
      <y><noise type="gaussian"><mean>0.0</mean><stddev>0.0002</stddev></noise></y>
      <z><noise type="gaussian"><mean>0.0</mean><stddev>0.0002</stddev></noise></z>
    </angular_velocity>
    <linear_acceleration>
      <x><noise type="gaussian"><mean>0.0</mean><stddev>0.01</stddev></noise></x>
      <y><noise type="gaussian"><mean>0.0</mean><stddev>0.01</stddev></noise></y>
      <z><noise type="gaussian"><mean>0.0</mean><stddev>0.01</stddev></noise></z>
    </linear_acceleration>
  </imu>
  <plugin name="imu_plugin" filename="libgazebo_ros_imu_sensor.so">
    <ros>
      <namespace>/imu</namespace>
      <remapping>~/out:=data</remapping>   <!-- 토픽 이름 변경 -->
    </ros>
    <frame_name>base_link</frame_name>
  </plugin>
</sensor>

---

#### ⑤ 접촉 센서 (type="contact") — 충돌·접촉 감지

<sensor name="front_contact_sensor" type="contact">
  <always_on>true</always_on>
  <update_rate>50</update_rate>
  <contact>
    <collision>front_bumper_collision</collision>  <!-- 감지할 collision 이름 -->
  </contact>
  <plugin name="front_contact_plugin" filename="libgazebo_ros_bumper.so">
    <ros>
      <namespace>/bumper</namespace>
      <remapping>bumper_states:=states</remapping>
    </ros>
    <frame_name>front_bumper_link</frame_name>
  </plugin>
</sensor>

---

#### 센서별 플러그인 파일 정리

일반 카메라 / 깊이 카메라 : libgazebo_ros_camera.so
2D LiDAR                  : libgazebo_ros_ray_sensor.so
IMU                       : libgazebo_ros_imu_sensor.so
접촉 센서                  : libgazebo_ros_bumper.so

---

## 4. Xacro (.xacro) — URDF 매크로 확장

URDF를 그대로 쓰되 변수와 매크로(함수)를 추가해 반복을 줄인다.
빌드 시 xacro 처리기가 일반 URDF로 변환한다.

### 4-1. 변수 선언 및 사용

<?xml version="1.0"?>
<robot xmlns:xacro="http://www.ros.org/wiki/xacro" name="box_robot">

  <!-- 변수 선언 -->
  <xacro:property name="base_length" value="1.0"/>
  <xacro:property name="base_width"  value="0.6"/>
  <xacro:property name="base_height" value="0.3"/>
  <xacro:property name="base_mass"   value="2.0"/>

  <link name="base_link">
    <visual>
      <!-- ${} 로 변수 참조, 수식도 사용 가능 -->
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

---

### 4-2. 매크로(함수) 선언 및 호출

<?xml version="1.0"?>
<robot xmlns:xacro="http://www.ros.org/wiki/xacro" name="macro_box_robot">

  <!-- 매크로 선언 — params에 인수 목록 지정 -->
  <xacro:macro name="box_link" params="link_name length width height mass color">
    <link name="${link_name}">
      <visual>
        <origin xyz="0 0 ${height / 2}" rpy="0 0 0"/>
        <geometry><box size="${length} ${width} ${height}"/></geometry>
      </visual>
      <collision>
        <origin xyz="0 0 ${height / 2}" rpy="0 0 0"/>
        <geometry><box size="${length} ${width} ${height}"/></geometry>
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

  <!-- 매크로 호출 -->
  <xacro:box_link
    link_name="base_link"
    length="1.0" width="0.6" height="0.3"
    mass="2.0" color="Gazebo/Blue"/>

  <xacro:box_link
    link_name="sensor_link"
    length="0.3" width="0.2" height="0.2"
    mass="0.5" color="Gazebo/Green"/>

  <joint name="base_to_sensor_joint" type="fixed">
    <parent link="base_link"/>
    <child link="sensor_link"/>
    <origin xyz="0.3 0 0.3" rpy="0 0 0"/>
  </joint>

</robot>

---

## 5. ROS2 관련 개념

### 5-1. robot_state_publisher

URDF를 읽어 로봇의 좌표계 정보(TF)를 ROS2 토픽으로 발행하는 노드.
관절 상태(/joint_states)를 구독해 동적 TF를 계산한다.

URDF → robot_state_publisher → TF Tree

### 5-2. 의존성(Dependency)

내 패키지가 동작하기 위해 필요한 외부 패키지·라이브러리.
package.xml의 <depend> 태그에 명시하고, colcon build 시 자동 처리.

### 5-3. 주요 토픽

/joint_states : 로봇 관절의 현재 상태 (위치·속도·힘)
/tf           : 실시간으로 변하는 좌표계 관계
/tf_static    : 변하지 않는 고정 좌표계 관계