# Mujoco 3.6.0

## 설치 및 버전 확인

- 설치

`pip install mujoco==3.6.0`

- 버전 확인

`mujoco.__version__`

## MJCF XML

> **body**: 물리 객체의 기준 좌표계

---

### option 속성

| 속성 | 설명 |
| --- | --- |
| `timestep` | 시뮬레이션(mj_step()) 시간 간격 [s] |
| `gravity="x y z"` | 중력 벡터 [m/s²] (기본값: "0 0 -9.81") |

---

### geom

body의 모양 정의

### geom 속성

| 속성 | 설명 |
| --- | --- |
| `pos` | 초기 위치 (x y z) [m] |
| `size` | 크기 [m] (박스·캡슐은 반지름 개념, 구는 반지름 단일값) |
| `mass` | 질량 [kg] |
| `friction="sliding torsional rolling"` | 마찰 계수 — sliding: 미끄럼, torsional: 비틀림, rolling: 구름 |
| `euler="rx ry rz"` | 각 축 기준 회전각 [deg] (예: "0 30 0" → y축으로 30° 기울임) |
| `solref="timeconst damping"` | 접촉 후 복원 특성 — timeconst: 복원 시간상수 [s], damping: 감쇠 비율 |
| `solimp="dmin dmax width"` | 접촉 강성 — dmin/dmax: 최소·최대 강성(0~1), width: 전환 폭 |

### geom type

| `type` | 설명 |
| --- | --- |
| `plane` | 무한 평면 (바닥에 사용) |
| `box` | 직육면체, size = 각 축 반길이 (x y z) |
| `sphere` | 구, size = 반지름 |
| `capsule` | 캡슐 (원통 + 양 끝 반구), `fromto="x1 y1 z1 x2 y2 z2"`로 양 끝점 지정, size = 반지름 |

---

### joint 속성

> **joint**: body의 움직임 정의

| 속성 | 설명 |
| --- | --- |
| `type="hinge"` | 회전 관절 (경첩) |
| `type="slide"` | 직선 이동 관절 |
| `axis="x y z"` | 관절 동작 기준 방향 벡터 (hinge: 회전축, slide: 이동축) |
| `limited="true"` | range 제한 활성화 |
| `range` | hinge: 각도 범위 [deg], slide: 이동 범위 [m] |
| `<freejoint/>` | 6자유도, 물체가 공간에서 자유롭게 이동·회전 |

---

### light 속성

| 속성 | 설명 |
| --- | --- |
| `pos` | 조명 위치 (x y z) [m] |
| `dir` | 조명 방향 벡터 (기본값: "0 0 -1", 아래 방향) |
| `diffuse` | 난반사 색상 (r g b, 기본값: "0.7 0.7 0.7") |
| `specular` | 정반사 색상 (r g b, 기본값: "0.3 0.3 0.3") |
| `directional` | true: 태양광처럼 방향만 있는 조명, false: 점광원 (기본값: false) |
| `castshadow` | 그림자 생성 여부 (기본값: true) |

---

### 기본 구조 예시

```xml
<mujoco>
    <option timestep="0.01" gravity="0 0 -9.81"/>
    <worldbody>
        <!-- 조명: pos에서 dir 방향으로 빛을 쏨 -->
        <light name="top_light" pos="0 0 3" dir="0 0 -1" castshadow="true"/>

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
                  solref="0.01 1" solimp="0.9 0.95 0.001"/>
        </body>

        <!-- 튀는 공 -->
        <body name="ball_body" pos="0 0 2">
            <freejoint/>
            <geom name="ball_geom" type="sphere" size="0.2" mass="1"
                  solref="0.01 1" solimp="0.9 0.95 0.001"/>
        </body>

        <!-- 회전 관절 -->
        <body name="link1" pos="0 0 0">
            <joint name="joint1" type="hinge" axis="0 0 1" limited="true" range="-90 90"/>
            <geom name="link1_geom" type="capsule" fromto="0 0 0 1 0 0" size="0.05" mass="1"/>
        </body>

        <!-- 직선 이동 관절 -->
        <body name="slider" pos="0 0 0.5">
            <joint name="slide_x" type="slide" axis="1 0 0" limited="true" range="-1 1"/>
            <geom name="slider_geom" type="box" size="0.1 0.1 0.1" mass="1"/>
        </body>
    </worldbody>
</mujoco>
```

```xml
<actuator>
    <!-- 관절에 직접 힘 또는 토크를 넣는 방식 -->
    <motor name="hinge_motor"
           joint="hinge_joint"
           gear="1"
           ctrllimited="true"
           ctrlrange="-5 5"/>

    <!-- 목표 위치를 따라가도록 제어 -->
    <position name="pos_servo"
            joint="joint1"
            kp="10"
            ctrllimited="true"
            ctrlrange="-3.14 3.14"/>
</actuator>
```

| 속성 | 설명 |
| --- | --- |
| `joint` | 어느 joint를 구동할 지 지정 |
| `gear` | 입력을 힘/토크로 변환하는 비율 |
| `ctrllimited` | ctrlrange 제한 사용 여부 |
| `ctrlrange` | 입력 가능한 제어값 범위 |
| `kp` | 위치 오차에 대한 제어 강도 |

---

## 모델 및 데이터

```python
# xml 문자열을 mujoco 모델 객체로 변환
model = mujoco.MjModel.from_xml_string(path)

# 시뮬레이션 중 가변 값을 담는 객체
data = mujoco.MjData(model)

# 시뮬레이션 한 스텝 진행
mujoco.mj_step(model, data)

# name으로 id 찾기
id = mujoco.mj_name2id(model, mujoco.mjtObj.mjOBJ_BODY, "name")

# id으로 name 찾기
name = mujoco.mj_id2name(model, mujoco.mjtObj.mjOBJ_GEOM, "id")
```

---

## 데이터 상세

1. model

```python
# 위치 상태 변수 개수 (len(data.qpos) == model.nq)
model.nq

# 속도 상태 변수 개수
model.nv
```

2. data

```python
# 시간
data.time

# 접촉 개수 (닿음 & 충돌)
data.ncon

# 접촉 정보 & 접촉한 두 geom id
data.contact[index]
data.contact[index].geom1
data.contact[index].geom2

# body 위치 - [x y z]
data.xpos[id]

# joint 위치값(상태값)
data.qpos

# joint 속도
data.qvel
```

---

## 값 할당

```python
# freejoint qpos - 3개의 위치 + 4개의 회전(쿼터니언) = 7개의 상태 [x y z qw qx qy qz]
# freejoint qvel - 3개의 선속도 + 3개의 각속도 = 6개의 속도 [vx vy vz wx wy wz]
# x방향 속도 할당
data.qvel[0] = 2.0

# actuator에 제어 입력 할당
# motor - 힘/토크, position - 목표 위치 [rad]
data.ctrl[index] = value
```

---

## viewer

```python
import time
import mujoco
import mujoco.viewer

# MuJoCo viewer 창을 실행
with mujoco.viewer.launch_passive(model, data) as viewer:
    while viewer.is_running():
        # 시뮬레이션을 한 스텝 진행
        mujoco.mj_step(model, data)
        # 시뮬레이션 상태를 viewer 화면에 반영
        viewer.sync()
        # 너무 빠르게 실행되지 않도록 잠깐 대기
        time.sleep(model.opt.timestep)
```