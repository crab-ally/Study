# Mujoco 3.6.0

## 1. 설치 및 버전 확인

- 설치

`pip install mujoco==3.6.0`

- 버전 확인

`mujoco.__version__`

---

## 2. MJCF(MuJoCo XML Format) 구조

```
<mujoco>
  <option .../>          ← 시뮬레이터 전체 설정
  <worldbody>            ← 모든 물체의 루트(월드 좌표계)
    <light .../>            ← 조명
    <geom .../>             ← 월드에 고정된 형상
    <body ...>           ← 움직이는 물체 단위(물리 객체의 기준 좌표계)
      <joint .../>          ← 부모 body에 대해 어떻게 움직일 수 있는지 정의
      <geom .../>           ← body의 실제 모양
      <site .../>           ← 위치 표시용 점(marker)
      <camera .../>         ← 카메라
    </body>
  </worldbody>
  <actuator> ... </actuator>
  <sensor>   ... </sensor>
</mujoco>
```

> 하나의 body 안에 여러 geom을 넣을 수 있으며, joint는 부모 body와의 상대 운동을 정의합니다.

---

### 2-1. mesh (.stl, .obj, .dae)

로봇의 외형 파일

---

## 3. `<option>` — 전역 물리 설정

```xml
<option timestep="0.01" gravity="0 0 -9.81"/>
```

| 속성 | 설명 |
|------|------|
| `timestep` | `mj_step()` 한 번의 시간 간격 [s] |
| `gravity="x y z"` | 중력 벡터 [m/s²] (기본값: `"0 0 -9.81"`) |

---

## 4. `<geom>` — 형상 정의

body의 모양과 물리 속성을 정의합니다.

### 4-1. geom type

| `type` | 설명 |
|---|---|
| `plane` | 무한 평면 (바닥에 사용) |
| `box` | 직육면체, `size = 각 축 반길이 (x y z)` |
| `sphere` | 구, `size = 반지름` |
| `capsule` | 캡슐 (원통 + 양 끝 반구), `size = [반지름, 반높이]` |
| `cylinder` | 원통, `size = [반지름, 반높이]` |

> `fromto`는 `capsule`과 `cylinder`에서 사용한다.
> `fromto` 사용 시 `size`는 반지름만 사용한다.

### 4-2. geom 속성

| 속성 | 설명 |
|------|------|
| `pos="x y z"` | 초기 위치 [m] |
| `size` | 크기 [m] |
| `mass` | 질량 [kg] |
| `density` | 밀도 → 질량, 관성 자동 계산 |
| `euler="rx ry rz"` | 각 축 기준 회전각 [deg] (예: `"0 30 0"` → y축 30° 기울임) |
| `friction="sliding torsional rolling"` | 마찰 계수 — 미끄럼 / 회전 / 구름 |
| `solref="timeconst damping"` | 접촉 후 복원 특성 — 복원 시간상수 [s] / 감쇠 비율 |
| `solimp="dmin dmax width"` | 접촉 강성 — 최소·최대 강성(0~1) / 전환 폭 |
| `rgba="r g b a"` | 색상 및 투명도 |

> 좌표계 기준은 부모 기준 상대 좌표

---

## 5. `<joint>` — 관절 정의

body의 움직임을 정의합니다. joint가 없으면 body는 부모에 고정됩니다.

| 속성 | 설명 |
|------|------|
| `pos="x y z"` | 회전 중심 위치 |
| `type="hinge"` | 회전 관절 [1DOF] |
| `type="slide"` | 직선 이동 관절 [1DOF] |
| `type="ball"` | 모든 방향 회전 관절 [3DOF] |
| `axis="x y z"` | 관절 동작 기준 방향 벡터 (hinge: 회전축, slide: 이동축) |
| `limited="true"` | `range` 제한 활성화 |
| `range` | hinge: 각도 범위 [deg], slide: 이동 범위 [m] |
| `<freejoint/>` | 6자유도 — 공간에서 자유롭게 이동·회전 |

> DOF(Degree of Freedom): 움직일 수 있는 축의 개수  
> DOF 많을수록 움직임 자유도가 높으나 제어 난이도도 높다.  

> **qpos / qvel 구조**
> - `freejoint`: qpos 7개 `[x y z qw qx qy qz]`, qvel 6개 `[vx vy vz wx wy wz]`
> - `hinge` / `slide`: qpos 1개 (angle/position), qvel 1개

---

## 6. `<light>` — 조명

| 속성 | 설명 |
|------|------|
| `pos` | 조명 위치 (x y z) [m] |
| `dir` | 조명 방향 벡터 (기본값: `"0 0 -1"`, 아래 방향) |

---

## 7. `<camera>` — 카메라

```xml
<camera name="front_camera" pos="2 -3 2" xyaxes="1 0 0  0 0 1"/>
```

| 속성 | 설명 |
|------|------|
| `pos` | 카메라 위치 |
| `xyaxes="x1 x2 x3 y1 y2 y3"` | 카메라 방향을 정의하는 x·y 축 벡터 |

---

## 8. `<site>` — 기준점

공간상의 특정 위치·방향을 표시하는 마커입니다.
센서의 측정 기준점으로 활용하거나, 엔드 이펙터 위치 추적에 사용합니다.

```xml
<site name="end_site" pos="1 0 0" size="0.05" rgba="1 0 0 1"/>
```

| 속성 | 설명 |
|------|------|
| `pos` | body 기준 site의 위치 |
| `size` | 화면 표시 크기 |
| `rgba` | 색상 및 투명도 |

---

## 9. `<actuator>` — 구동기

### 9-1. motor — 힘/토크 직접 입력

```xml
<motor name="hinge_motor"
       joint="hinge_joint"
       gear="1"
       ctrllimited="true"
       ctrlrange="-5 5"/>
```

| 속성 | 설명 |
|---|---|
| `gear` | 입력을 힘/토크로 변환하는 비율 |

### 9-2. position — 목표 위치 추종 (PD 제어)

```xml
<position name="pos_servo"
           joint="joint1"
           kp="10"
           ctrllimited="true"
           ctrlrange="-3.14 3.14"/>
```

| 속성 | 설명 |
|---|---|
| `kp` | 위치 오차에 대한 제어 강도 (position 전용) |

### 9-3. velocity — 목표 속도 추종

```xml
<velocity name="wheel_velocity_servo"
           joint="wheel_joint"
           kv="5"
           ctrllimited="true"
           ctrlrange="-10 10"/>
```

| 속성 | 설명 |
|---|---|
| `kv` | 속도 오차에 대한 제어 강도 (velocity 전용) |

### 9-4. 공통 속성

| 속성 | 설명 |
|---|---|
| `joint` | 구동할 joint 이름 |
| `ctrllimited` | `ctrlrange` 제한 사용 여부 |
| `ctrlrange` | 입력 가능한 제어값 범위 |

> **제어 입력 할당**: `data.ctrl[actuator_id] = value`
> - motor: 힘/토크 [N 또는 Nm]
> - position: 목표 위치 [rad]
> - velocity: 목표 속도 [rad/s]

---

## 10. `<sensor>` — 센서

### 10-1. joint 상태 센서

```xml
<jointpos name="joint1_pos" joint="joint1"/>       <!-- 위치 -->
<jointvel name="joint1_vel" joint="joint1"/>       <!-- 속도 -->
<jointforce name="joint1_force" joint="joint1"/>   <!-- 힘/토크 -->
```

### 10-2. site 위치 센서

```xml
<!-- 특정 site의 3D 위치 측정 → [x, y, z] -->
<framepos name="end_site_position" objtype="site" objname="end_site"/>
```

### 10-3. IMU (가속도 + 각속도)

```xml
<accelerometer name="imu_acc"  site="imu_site"/>
<gyro          name="imu_gyro" site="imu_site"/>
```

### 10-4. 힘/토크 센서

```xml
<force  name="force_sensor"  site="force_site"/>    <!-- [fx, fy, fz] -->
<torque name="torque_sensor" site="torque_site"/>   <!-- [tx, ty, tz] -->
```

> force: 직선 방향 힘  
> torque: 회전 방향 힘

### 10-5. 공통 속성

| 속성 | 설명 |
|------|------|
| `joint` | 측정 대상 joint 이름 (joint 직접 측정용) |
| `objtype` | 측정 대상 종류 (`site`, `body` 등) |
| `objname` | 측정 대상 이름 |
| `site` | 가속도·각속도·힘·토크 측정 기준 site 이름 |

---

## 11. XML 전체 예시

```xml
<mujoco>
    <option timestep="0.01" gravity="0 0 -9.81"/>

    <worldbody>
        <light name="top_light" pos="0 0 3" dir="0 0 -1" castshadow="true"/>
        <camera name="front_camera" pos="2 -3 2" xyaxes="1 0 0  0 0 1"/>

        <!-- 바닥 -->
        <geom name="ground" type="plane" size="5 5 0.1"
              friction="0.3 0.01 0.001"
              solref="0.01 1" solimp="0.9 0.95 0.001"/>

        <!-- 경사면 -->
        <geom name="slope" type="box" size="3 1 0.05"
              pos="0 0 0" euler="0 30 0"
              friction="0.3 0.01 0.001"/>

        <!-- 자유 낙하 박스 -->
        <body name="box_body" pos="0 0 1">
            <freejoint/>
            <geom name="box_geom" type="box" size="0.2 0.2 0.2" mass="1"
                  rgba="0.2 0.4 1 1"
                  solref="0.01 1" solimp="0.9 0.95 0.001"/>
        </body>

        <!-- 튀는 공 -->
        <body name="ball_body" pos="0 0 2">
            <freejoint/>
            <geom name="ball_geom" type="sphere" size="0.2" mass="1"
                  solref="0.01 1" solimp="0.9 0.95 0.001"/>
        </body>

        <!-- 회전 관절 + 엔드 이펙터 site -->
        <body name="base" pos="0 0 0">
            <joint name="joint1" type="hinge" axis="0 0 1"
                   limited="true" range="-180 180"/>
            <geom name="link1" type="capsule"
                  fromto="0 0 0  1 0 0" size="0.05" mass="1"/>
            <site name="end_site" pos="1 0 0" size="0.05" rgba="1 0 0 1"/>
        </body>

        <!-- 직선 이동 관절 -->
        <body name="slider" pos="0 0 0.5">
            <joint name="slide_x" type="slide" axis="1 0 0"
                   limited="true" range="-1 1"/>
            <geom name="slider_geom" type="box" size="0.1 0.1 0.1" mass="1"/>
        </body>
    </worldbody>

    <actuator>
        <motor name="hinge_motor" joint="joint1" gear="1" ctrllimited="true" ctrlrange="-5 5"/>
        <position name="joint1_servo" joint="joint1" kp="20" ctrllimited="true" ctrlrange="-3.14 3.14"/>
        <velocity name="slide_velocity" joint="slide_x" kv="5" ctrllimited="true" ctrlrange="-10 10"/>
    </actuator>

    <sensor>
        <jointpos name="joint1_pos" joint="joint1"/>
        <jointvel name="joint1_vel" joint="joint1"/>
        <framepos name="end_site_position" objtype="site" objname="end_site"/>
    </sensor>
</mujoco>
```

---

## 12. Python API

### 12-1. 모델·데이터 생성

```python
import mujoco

# XML 파일 경로로 모델 로드
model = mujoco.MjModel.from_xml_path(xml_path)

# XML 문자열로 모델 로드
model = mujoco.MjModel.from_xml_string(xml_string)

# 시뮬레이션 가변 상태를 담는 객체
data = mujoco.MjData(model)

# 시뮬레이션 한 스텝 진행 (timestep 만큼)
mujoco.mj_step(model, data)
```

### 12-2. 이름 ↔ ID 변환

```python
# name → id (실패 시 -1 반환)
body_id     = mujoco.mj_name2id(model, mujoco.mjtObj.mjOBJ_BODY, "body_name")
actuator_id = mujoco.mj_name2id(model, mujoco.mjtObj.mjOBJ_ACTUATOR, "actuator_name")
sensor_id   = mujoco.mj_name2id(model, mujoco.mjtObj.mjOBJ_SENSOR, "sensor_name")

# id → name (실패 시 None 반환)
geom_name = mujoco.mj_id2name(model, mujoco.mjtObj.mjOBJ_GEOM, geom_id)
```

> `mjtObj` 주요 상수: `mjOBJ_BODY`, `mjOBJ_GEOM`, `mjOBJ_JOINT`,
> `mjOBJ_ACTUATOR`, `mjOBJ_SENSOR`, `mjOBJ_SITE`, `mjOBJ_CAMERA`

### 12-3. mujoco 속성

```python
# 이미지 생성
renderer = mujoco.Renderer(model, height=480, width=640)       # 이미지 렌더링을 위한 객체 생성 함수
renderer.enable_depth_rendering()           # render() 시 깊이(Depth) 데이터 반환
renderer.disable_depth_rendering()          # render() 시 RGB 데이터 반환
renderer.update_scene(data, camera="name")  # 시점 갱신
renderer.render()                           # 이미지를 numpy 배열로 받기 [RGB - 0~255]

mujoco.mj_resetData(model, data)    # data 초기화
```

### 12-4. model 속성

```python
model.nbody         # body 개수
model.ngeom         # geom 개수
model.njnt          # joint 개수
model.nactuator     # actuator 개수
model.nsensor       # sensor 개수
model.nsite         # site 개수
model.ncam          # camera 개수
model.nu            # 제어입력 개수
model.nq            # qpos 원소 수 (위치 상태 변수 개수)
model.nv            # qvel 원소 수 (속도 상태 변수 개수)
model.opt.timestep  # 설정된 timestep 값

# 센서 데이터 인덱스 정보 
model.sensor_adr[sensor_id]   # sensordata 내 시작 인덱스
model.sensor_dim[sensor_id]   # 해당 센서의 데이터 개수

# 위치 데이터 인덱스 정보
model.jnt_qposadr[joint_id]   # qpos 내 해당 joint의 시작 인덱스

# 속도 데이터 인섹스 정보
model.jnt_dofadr[joint_id]    # qvel 내 해당 joint의 시작 인덱스

# 관절 타입 정보
model.jnt_type[joint_id] # int enum으로 저장

# 관절 제한 범위
model.jnt_range[joint_id] # [min, max]
model.jnt_limited[joint_id] # 0,1 반환

# 엑츄에이터
model.actuator_ctrllimited[actuator_id]
model.actuator_ctrlrange[actuator_id]   # [min, max ]
model.actuator_trnid # 액추에이터가 어떤 객체(joint, tendon 등)에 연결되어 있는지 나타내는 ID 정보
```

> model.nu == model.actuator

### 12-5. data 속성

```python
data.time          # 현재 시뮬레이션 시간 [s]
data.ncon          # 현재 접촉 쌍의 수

# 접촉 정보
data.contact[index].geom1   # 접촉한 geom id (첫 번째)
data.contact[index].geom2   # 접촉한 geom id (두 번째)

data.xpos           # 모든 body의 월드 좌표 [x, y, z]
data.qpos           # 모든 joint 위치값 (상태)
data.qvel           # 모든 joint 속도값
data.qacc           # 모든 joint 가속도값
data.xquat          # 모든 body의 월드 쿼터니언 [qw, qx, qy, qz]

data.sensordata     # 모든 센서 데이터 (1D 배열, 연속 저장)

# 힘/토크
data.qfrc_actuator  # 액추에이터(모터)가 생성한 힘/토크
data.qfrc_bias      # 중력, 코리올리 힘, 원심력 등 동역학적 bias force
data.qfrc_passive   # 스프링, 댐핑 등 수동(passive) 요소가 생성한 힘/토크
data.qfrc_applied   # 사용자가 직접 적용한 외력/외부 토크
```

> data.qvel.shape == data.qacc.shape == data.qfrc_actuator.shape == data.qfrc_bias.shape == data.qfrc_passive.shape == data.qfrc_applied.shape : 관절당 인덱스도 같음

### 12-6. 값 할당

```python
# 액추에이터 제어 입력
data.ctrl[actuator_id] = value

# freejoint 초기 위치·자세 직접 설정
# qpos: [x, y, z, qw, qx, qy, qz]  (7개)
data.qpos[0:3] = [1.0, 0.0, 0.5]    # 위치
data.qpos[3:7] = [1.0, 0.0, 0.0, 0.0]  # 자세 (단위 쿼터니언)

# freejoint 속도 직접 설정
# qvel: [vx, vy, vz, wx, wy, wz]  (6개)
data.qvel[0] = 2.0   # x 방향 선속도
```

> **값 할당 후 순방향 키네마틱 갱신이 필요한 경우**:
> `mujoco.mj_forward(model, data)` 를 호출하면 `xpos` 등 파생 값이 즉시 갱신됩니다.
> `mj_step()` 은 내부적으로 `mj_forward()` 를 포함합니다.

### 12-7. 센서 데이터 읽기

```python
def read_sensor(model, data, name):
    sid = mujoco.mj_name2id(model, mujoco.mjtObj.mjOBJ_SENSOR, name)
    adr = model.sensor_adr[sid]
    dim = model.sensor_dim[sid]
    return data.sensordata[adr:adr + dim].copy()

pos = read_sensor(model, data, "end_site_position")  # [x, y, z]
```

---

## 13. Viewer (실시간 시각화)

```python
import time
import mujoco
import mujoco.viewer

with mujoco.viewer.launch_passive(model, data) as viewer:
    while viewer.is_running():
        mujoco.mj_step(model, data)
        viewer.sync()   # 시뮬레이션 상태를 viewer 화면에 반영
        time.sleep(model.opt.timestep)   # 너무 빠르게 실행되지 않도록 잠깐 대기
```

> `launch_passive`: 별도 창을 띄우되 메인 루프는 Python이 관리합니다.

---

## 14. CSV 데이터 로깅

```python
import csv
import math
import mujoco

log_file = "log.csv"

# 파일 열기
with open(log_file, "w", newline="", encoding="utf-8") as f:
    writer = csv.writer(f)  # csv 파일에 행을 기록하는 객체
    writer.writerow([       # 행 기록
        "time",
        "target_deg",
        "qpos_deg",
        "qvel_deg_s",
        "sensor_pos_deg",
        "sensor_vel_deg_s"
    ])

    for _ in range(500):
        target = math.radians(45) * math.sin(2 * math.pi * 0.5 * data.time)
        data.ctrl[0] = target

        mujoco.mj_step(model, data)

        sensor_pos = data.sensordata[0]
        sensor_vel = data.sensordata[1]

        writer.writerow([
            data.time,
            math.degrees(target),
            math.degrees(data.qpos[0]),
            math.degrees(data.qvel[0]),
            math.degrees(sensor_pos),
            math.degrees(sensor_vel)
        ])
```

---

## 15. PID 제어

### P 제어

현재 오차에 비례해서 힘을 줌
- 장점: 구현이 간단, 목표 방향으로 빠르게 움직임
- 단점: 목표 지점 근처에서 진동 발생, 오버슈트(목표를 지나침) 발생 가능

```python
# 2-link arm 예시
error0 = target_shoulder - data.qpos[0]
error1 = target_elbow - data.qpos[1]

ctrl0 = kp * error0
ctrl1 = kp * error1

data.ctrl[0] = ctrl0
data.ctrl[1] = ctrl1
```

### PD 제어

P 제어에 속도 기반 감쇠(D)를 추가
- 장점: 진동과 오버슈트 감소, 더 부드럽게 목표 도달
- 단점: kd가 너무 크면 움직임이 둔해짐

```python
# 2-link arm 예시
error0 = target_shoulder - data.qpos[0]
error1 = target_elbow - data.qpos[1]

ctrl0 = kp * error0 - kd * data.qvel[0]
ctrl1 = kp * error1 - kd * data.qvel[1]

data.ctrl[0] = ctrl0
data.ctrl[1] = ctrl1
```

---

## 16. Inverse Kinematics (역기구학)

```python
# 거리
d = math.sqrt(x*x + y*y)

# 안전 클램프
cos_theta2 = (x*x + y*y - L1*L1 - L2*L2) / (2*L1*L2)
cos_theta2 = max(-1.0, min(1.0, cos_theta2))

theta2 = math.acos(cos_theta2)

# theta1 계산
k1 = L1 + L2 * math.cos(theta2)
k2 = L2 * math.sin(theta2)

theta1 = math.atan2(y, x) - math.atan2(k2, k1)
```

> 도달 불가능 영역: sqrt(x² + y²) > L1 + L2  
> 두 개의 해 존재: elbow up / elbow down

---

### IK + PD 제어

```python
import time
import math
import mujoco
import mujoco.viewer

model = mujoco.MjModel.from_xml_path("two_link_arm.xml")
data = mujoco.MjData(model)

kp = 5.0
kd = 1.0

t = 0.0

def inverse_kinematics(x, y, L1=0.5, L2=0.4):
    cos_theta2 = (x*x + y*y - L1*L1 - L2*L2) / (2*L1*L2)
    cos_theta2 = max(-1.0, min(1.0, cos_theta2))
    theta2 = math.acos(cos_theta2)

    k1 = L1 + L2 * math.cos(theta2)
    k2 = L2 * math.sin(theta2)

    theta1 = math.atan2(y, x) - math.atan2(k2, k1)

    return theta1, theta2

with mujoco.viewer.launch_passive(model, data) as viewer:
    while viewer.is_running():

        # 🎯 목표 위치 (원형 trajectory)
        x = 0.6 * math.cos(t)
        y = 0.4 * math.sin(t)

        # IK 계산
        target = inverse_kinematics(x, y)

        # PD 제어
        for i in range(2):
            error = target[i] - data.qpos[i]
            ctrl = kp * error - kd * data.qvel[i]
            ctrl = max(-1.0, min(1.0, ctrl))
            data.ctrl[i] = ctrl

        mujoco.mj_step(model, data)
        viewer.sync()

        t += model.opt.timestep
        time.sleep(model.opt.timestep)
```