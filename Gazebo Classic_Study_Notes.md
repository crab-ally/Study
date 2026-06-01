# Gazebo Classic
## SDF XML (.world)

Gazebo가 실행될 전체 환경을 정의

---

### 1. SDF 기본 구조 태그

- `<model> </model>`: 하나의 모델 정의

| 태그 | 설명 |
| --- | --- |
| `<link> </link>` | 움직이는 몸체 정의 |
| `<joint> </joint>` | 관절 정의 |

- **`<link>`**

| 태그 | 설명 |
| --- | --- |
| `<inertial> </inertial>` | 물체의 질량과 관성 정보 정의 |
| `<collision> </collision>` | 충돌 감지 영역 |
| `<visual> </visual>` | 화면에 보이는 형상 |

- **`<joint>`**

| 태그/속성 | 설명 |
| --- | --- |
| `<parent> </parent>`, `<child> </child>` | 부모 link, 자식 link |
| `<pose>x y z roll pitch yaw</pose>` | 부모 link 기준 관절의 위치와 자세 |
| `type="fixed"` | 고정 관절 |
| `type="revolute"` | 회전 관절 |
| `type="prismatic"` | 직선 이동 관절 |
| `<axis> </axis>` | 회전축 정보 |

---

### 2. SDF 주요 속성 태그

- **`<model>`**

| 태그 | 설명 |
| --- | --- |
| `<static>true</static>` | 위치 고정 |
| `<pose>x y z roll pitch yaw</pose>` | 모델의 위치와 자세 |

---

- **`<inertial>`**

| 태그 | 설명 |
| --- | --- |
| `<mass> </mass>` | 질량 [kg] |
| `<inertia> </inertia>` | 회전 관성 행렬 정의 |

> 관성: 돌리려고 할 때 얼마나 잘 안 도는가

- `<inertia>`

| 태그 | 설명 |
| --- | --- |
| `<ixx> </ixx>`, `<iyy> </iyy>`, `<izz> </izz>` | x,y,z축 기준 회전 관성 |
| `<ixy> </ixy>`, `<ixz> </ixz>`, `<iyz> </iyz>` | 축 사이의 관성 결합 항 |

---

- **`<collision>`**

| 태그 | 설명 |
| --- | --- |
| `<surface> </surface>` | 충돌 면의 물리적 특성 정의 |
| `<geometry> </geometry>` | 물체의 모양 정의 |

- `<surface>`

| 태그 | 설명 |
| --- | --- |
| `<friction> </friction>` | 마찰 속성 설정 |
| `<ode> </ode>` | ODE 물리 엔진용 설정 |
| `<mu> </mu>`, `<mu2> </mu2>` | 접촉면의 n번째 접선 방향 마찰 |

| 태그 | 설명 |
| --- | --- |
| `<bounce> </bounce>` | 충돌 후 튀는 정도 설정 |
| `<restitution_coefficient> </restitution_coefficient>` | 반발 계수 (0~1) |
| `<threshold> </threshold>` | 반발 효과가 적용되는 최소 속도 기준 |

- `<geometry>`

| 태그 | 설명 |
| --- | --- |
| `<box> </box>` | 박스 형태 |
| `<size>x y z</size>` | 크기 [m] |

| 태그 | 설명 |
| --- | --- |
| `<sphere> </sphere>` | 구 형태 |
| `<radius> </radius>` | 반지름 [m] |

| 태그 | 설명 |
| --- | --- |
| `<cylinder> </cylinder>` | 원기둥 형태 |
| `<radius> </radius>` | 반지름 [m] |
| `<length> </length>` | 원기둥의 길이 [m] |

---

- **`<visual>`**

| 태그 | 설명 |
| --- | --- |
| `<geometry> </geometry>` | 물체의 모양 정의 |
| `<material> </material>` | 시각 재질 설정 |

- `<geometry>`

| 태그 | 설명 |
| --- | --- |
| `<box> </box>` | 박스 형태 |
| `<size>x y z</size>` | 크기 [m] |

| 태그 | 설명 |
| --- | --- |
| `<sphere> </sphere>` | 구 형태 |
| `<radius> </radius>` | 반지름 [m] |

| 태그 | 설명 |
| --- | --- |
| `<cylinder> </cylinder>` | 원기둥 형태 |
| `<radius> </radius>` | 반지름 [m] |
| `<length> </length>` | 원기둥의 길이 [m] |

- `<material>`

| 태그 | 설명 |
| --- | --- |
| `<ambient>R G B A</ambient>` | 주변광에서 보이는 색 |
| `<diffuse>R G B A</diffuse>` | 직접 조명을 받을 때 보이는 색 |

---

- **`<axis>`**

| 태그 | 설명 |
| --- | --- |
| `<xyz> </xyz>` | 회전축 방향 벡터 |

| 태그 | 설명 |
| --- | --- |
| `<limit> </limit>` | 관절의 움직임 범위 제한 |
| `<lower> </lower>`, `<upper> </upper>` | 최소값, 최대값 [회전관절 - rad / 직선관절 - m] |
| `<effort> </effort>` | 최대 힘/토크 제한 |
| `<velocity> </velocity>` | 최대 속도 제한 |

---

### 3. 전체 예제 코드

```xml
<?xml version="1.0" ?>
<sdf version="1.6">
  <world name="colored_box_world">

    <!-- 중력 설정 -->
    <gravity>0 0 -1.62</gravity>

    <!-- 외부에 이미 정의된 모델을 현재 월드에 포함 -->
    <include>
      <uri>model://ground_plane</uri>
    </include>

    <include>
      <uri>model://sun</uri>
    </include>

    <!-- 박스 -->
    <model name="red_box">
      <pose>0 0 0.5 0 0 0</pose>

      <link name="box_link">
        <inertial>
          <mass>0.2</mass>
          <inertia>
            <ixx>0.1667</ixx>
            <iyy>0.1667</iyy>
            <izz>0.1667</izz>
            <ixy>0</ixy>
            <ixz>0</ixz>
            <iyz>0</iyz>
          </inertia>
        </inertial>

        <collision name="box_collision">
          <surface>
            <friction>
              <ode>
                <mu>0.01</mu>
                <mu2>0.01</mu2>
              </ode>
            </friction>
            <bounce>
              <restitution_coefficient>0.9</restitution_coefficient>
              <threshold>0.01</threshold>
            </bounce>
          </surface>

          <geometry>
            <box>
              <size>1 1 1</size>
            </box>
          </geometry>
        </collision>

        <visual name="box_visual">
          <geometry>
            <box>
              <size>1 1 1</size>
            </box>
          </geometry>

          <material>
            <ambient>1 0 0 1</ambient>
            <diffuse>1 0 0 1</diffuse>
          </material>
        </visual>
      </link>
    </model>

    <!-- 구 -->
    <model name="simple_sphere">
      <pose>0 0 0.5 0 0 0</pose>

      <link name="sphere_link">
        <collision name="sphere_collision">
          <geometry>
            <sphere>
              <radius>0.5</radius>
            </sphere>
          </geometry>
        </collision>

        <visual name="sphere_visual">
          <geometry>
            <sphere>
              <radius>0.5</radius>
            </sphere>
          </geometry>

          <material>
            <ambient>0 0 1 1</ambient>
            <diffuse>0 0 1 1</diffuse>
          </material>
        </visual>
      </link>
    </model>

    <!-- 원기둥 -->
    <model name="simple_cylinder">
      <pose>0 0 0.5 0 0 0</pose>

      <link name="cylinder_link">
        <collision name="cylinder_collision">
          <geometry>
            <cylinder>
              <radius>0.3</radius>
              <length>1.0</length>
            </cylinder>
          </geometry>
        </collision>

        <visual name="cylinder_visual">
          <geometry>
            <cylinder>
              <radius>0.3</radius>
              <length>1.0</length>
            </cylinder>
          </geometry>

          <material>
            <ambient>0 1 0 1</ambient>
            <diffuse>0 1 0 1</diffuse>
          </material>
        </visual>
      </link>
    </model>

    <!-- 고정 벽 -->
    <model name="static_wall">
      <static>true</static>
      <pose>2 0 0.5 0 0 0</pose>

      <link name="wall_link">
        <collision name="wall_collision">
          <geometry>
            <box>
              <size>0.2 3.0 1.0</size>
            </box>
          </geometry>
        </collision>

        <visual name="wall_visual">
          <geometry>
            <box>
              <size>0.2 3.0 1.0</size>
            </box>
          </geometry>

          <material>
            <ambient>0.6 0.6 0.6 1</ambient>
            <diffuse>0.6 0.6 0.6 1</diffuse>
          </material>
        </visual>
      </link>
    </model>

    <!-- 고정 관절 -->
    <model name="fixed_joint_object">
      <pose>0 0 0.5 0 0 0</pose>

      <link name="base_link">
        <collision name="base_collision">
          <geometry>
            <box>
              <size>1 1 0.3</size>
            </box>
          </geometry>
        </collision>

        <visual name="base_visual">
          <geometry>
            <box>
              <size>1 1 0.3</size>
            </box>
          </geometry>
          <material>
            <ambient>0.2 0.2 0.8 1</ambient>
            <diffuse>0.2 0.2 0.8 1</diffuse>
          </material>
        </visual>
      </link>

      <link name="top_link">
        <pose>0 0 0.65 0 0 0</pose>

        <collision name="top_collision">
          <geometry>
            <box>
              <size>0.5 0.5 1.0</size>
            </box>
          </geometry>
        </collision>

        <visual name="top_visual">
          <geometry>
            <box>
              <size>0.5 0.5 1.0</size>
            </box>
          </geometry>
          <material>
            <ambient>0.8 0.2 0.2 1</ambient>
            <diffuse>0.8 0.2 0.2 1</diffuse>
          </material>
        </visual>
      </link>

      <joint name="base_to_top_joint" type="fixed">
        <parent>base_link</parent>
        <child>top_link</child>
      </joint>
    </model>

    <!-- 회전 관절 -->
    <model name="simple_revolute_arm">
      <pose>0 0 0.2 0 0 0</pose>

      <link name="base_link">
        <collision name="base_collision">
          <geometry>
            <box>
              <size>0.6 0.6 0.2</size>
            </box>
          </geometry>
        </collision>

        <visual name="base_visual">
          <geometry>
            <box>
              <size>0.6 0.6 0.2</size>
            </box>
          </geometry>
          <material>
            <ambient>0.2 0.2 0.2 1</ambient>
            <diffuse>0.2 0.2 0.2 1</diffuse>
          </material>
        </visual>
      </link>

      <link name="arm_link">
        <pose>0.6 0 0.2 0 0 0</pose>

        <inertial>
          <mass>1.0</mass>
        </inertial>

        <collision name="arm_collision">
          <geometry>
            <box>
              <size>1.0 0.15 0.15</size>
            </box>
          </geometry>
        </collision>

        <visual name="arm_visual">
          <geometry>
            <box>
              <size>1.0 0.15 0.15</size>
            </box>
          </geometry>
          <material>
            <ambient>0.9 0.5 0.1 1</ambient>
            <diffuse>0.9 0.5 0.1 1</diffuse>
          </material>
        </visual>
      </link>

      <joint name="base_to_arm_joint" type="revolute">
        <parent>base_link</parent>
        <child>arm_link</child>

        <!-- 부모 link 기준 관절(회전축)의 위치 -->
        <pose>0.3 0 0.2 0 0 0</pose>

        <axis>
          <xyz>0 0 1</xyz>
          <limit>
            <lower>-1.57</lower>
            <upper>1.57</upper>
            <effort>10</effort>
            <velocity>2</velocity>
          </limit>
        </axis>
      </joint>
    </model>

    <!-- 직선 이동 관절 -->
    <model name="simple_slider">
      <pose>0 0 0.2 0 0 0</pose>

      <link name="base_link">
        <collision name="base_collision">
          <geometry>
            <box>
              <size>1.0 0.3 0.2</size>
            </box>
          </geometry>
        </collision>

        <visual name="base_visual">
          <geometry>
            <box>
              <size>1.0 0.3 0.2</size>
            </box>
          </geometry>
          <material>
            <ambient>0.2 0.2 0.2 1</ambient>
            <diffuse>0.2 0.2 0.2 1</diffuse>
          </material>
        </visual>
      </link>

      <link name="slider_link">
        <pose>0 0 0.35 0 0 0</pose>

        <inertial>
          <mass>0.5</mass>
        </inertial>

        <collision name="slider_collision">
          <geometry>
            <box>
              <size>0.3 0.3 0.2</size>
            </box>
          </geometry>
        </collision>

        <visual name="slider_visual">
          <geometry>
            <box>
              <size>0.3 0.3 0.2</size>
            </box>
          </geometry>
          <material>
            <ambient>0.1 0.8 0.3 1</ambient>
            <diffuse>0.1 0.8 0.3 1</diffuse>
          </material>
        </visual>
      </link>

      <joint name="base_to_slider_joint" type="prismatic">
        <parent>base_link</parent>
        <child>slider_link</child>

        <axis>
          <xyz>1 0 0</xyz>
          <limit>
            <lower>-0.4</lower>
            <upper>0.4</upper>
            <effort>20</effort>
            <velocity>1</velocity>
          </limit>
        </axis>
      </joint>

    </model>

  </world>
</sdf>
```