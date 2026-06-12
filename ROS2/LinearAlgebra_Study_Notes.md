# Linear Algebra — 로보틱스 관점 정리

로봇은 세상을 좌표와 방향으로 인식한다. 이를 수학적으로 표현하고 계산하는 핵심 도구가 선형대수이다.

---

## 1. 기본 개념

### 1-1. 점 vs 스칼라 vs 벡터

| 개념 | 설명 | 로보틱스 예시 |
|---|---|---|
| 점(Point) | 위치 | 로봇의 현재 위치 `(x, y)` |
| 스칼라(Scalar) | 크기 | 목표까지의 거리 `3.5m` |
| 벡터(Vector) | 이동량, 변화량 (크기 + 방향) | 속도 벡터 `(vx, vy)` |

> n차원 벡터 = 원소 n개로 이루어진 벡터

### 1-2. 기본 용어

| 용어 | 설명 |
|---|---|
| frame | 기준 좌표계 |
| transform | 한 좌표계의 점을 다른 좌표계로 변환 |
| unit vector | 크기가 1인 벡터 (방향만 유지) |

---

## 2. 벡터 연산

### 2-1. 기본 연산

```
덧셈:    (1, 2) + (3, 1) = (4, 3)   → 로봇 이동
뺄셈:    (4, 3) - (1, 2) = (3, 1)   → 목표 방향 벡터 계산
크기:    sqrt(x² + y²)              → 목표까지 거리 계산
스칼라배: 2 * (1, 2) = (2, 4)        → velocity scaling (로봇의 이동량 = 방향 단위벡터 × 속도 스칼라)
```

### 2-2. 내적 (Dot Product)

두 벡터가 얼마나 같은 방향을 향하고 있는지 측정한다.
→ **로봇이 목표를 향해 바라보고 있는지 판단**

```
A·B = |A||B|cosθ
```

| 결과 | 의미 | 로보틱스 해석 |
|---|---|---|
| 양수 | 같은 방향 | 로봇이 목표를 향하고 있음 |
| 0 | 수직 | 로봇이 목표와 직각 방향을 보고 있음 |
| 음수 | 반대 방향 | 로봇이 목표 반대 방향을 보고 있음 |

**예시 — 로봇이 목표를 바라보는지 확인:**

```python
import numpy as np

robot_dir = np.array([1, 0])      # 로봇이 향하는 방향 (동쪽)
to_goal   = np.array([0.8, 0.6])  # 목표 방향 벡터

dot = np.dot(robot_dir, to_goal)
# dot > 0 이면 목표가 로봇 전방에 있음
```

### 2-3. 외적 (Cross Product)

회전 방향을 판단한다.
→ **목표가 로봇 기준 왼쪽/오른쪽 어느 쪽에 있는지 판단**

```
[2D] cross_z = ax × by - ay × bx
```

| 결과 | 의미 | 로보틱스 해석 |
|---|---|---|
| 양수 | 반시계 방향 | 목표가 로봇 왼쪽에 있음 → 왼쪽으로 회전 |
| 0 | 같은/반대 방향 | 정렬된 상태 |
| 음수 | 시계 방향 | 목표가 로봇 오른쪽에 있음 → 오른쪽으로 회전 |

**예시 — 로봇이 어느 방향으로 회전할지 결정:**

```python
robot_dir = np.array([1, 0])
to_goal   = np.array([0.8, 0.6])

cross_z = robot_dir[0] * to_goal[1] - robot_dir[1] * to_goal[0]
# cross_z > 0 → 왼쪽으로 회전
# cross_z < 0 → 오른쪽으로 회전
```

### 2-4. 투영 (Projection)

벡터를 특정 축에 투영하면 그 축 방향의 성분을 추출할 수 있다. 계산은 내적으로 수행한다.

```python
v = (3, 4)
x축 방향 = (1, 0)

# v를 x축에 투영 → 3 (x 성분)
(3, 4) · (1, 0) = 3
```

**예시 — 로봇 속도를 전진/측면 성분으로 분리:**

```python
velocity   = np.array([0.5, 0.3])    # 현재 속도 벡터
forward    = np.array([1, 0])        # 전진 방향
lateral    = np.array([0, 1])        # 측면 방향

v_forward  = np.dot(velocity, forward)   # 전진 속도 성분
v_lateral  = np.dot(velocity, lateral)   # 측면 속도 성분
```

---

## 3. 행렬과 좌표 변환

행렬은 좌표를 변환하는 도구이다.

```python
# 행렬 * 벡터
A = [[a, b],
     [c, d]]
v = (x, y)

결과 = (ax + by, cx + dy)
```

### 3-1. 특수 행렬

```
단위행렬: 아무 변화도 주지 않는 행렬
[[1, 0],
 [0, 1]]

0행렬: 모든 값을 0으로 만드는 행렬
[[0, 0],
 [0, 0]]
```

### 3-2. 변환 행렬 종류

#### 이동변환 (Translation)

```
점 = (1, 2)
이동량 = (3, 4)

결과 = (1 + 3, 2 + 4) = (4, 6)
```

#### Scaling (확대/축소)

속도 scaling, 센서 거리 scaling 등에 활용

```python
M = [[2, 0],
     [0, 2]]

v = (3, 2)
v_new = M * v  # (6, 4) — 2배 확대
```

#### 방향 뒤집기 (Reflection)

```python
M = [[-1, 0],
     [ 0, 1]]

v = (3, 2)
v_new = M * v  # (-3, 2) — x축 기준 반전
```

#### 회전 (Rotation)

**로봇 기준 이동을 지도 기준 이동으로 변환**할 때 핵심으로 사용된다.

```python
# θ만큼 반시계 회전
M = [[cosθ, -sinθ],
     [sinθ,  cosθ]]

v_new = M * v
```

> 회전 행렬의 역행렬 = 회전 행렬의 전치행렬

**예시 — 로봇 로컬 속도를 월드 좌표 속도로 변환:**

```python
import numpy as np

theta = np.pi / 4   # 로봇이 45도 방향을 향하고 있음

R = np.array([[np.cos(theta), -np.sin(theta)],
              [np.sin(theta),  np.cos(theta)]])

v_local = np.array([1.0, 0.0])   # 로봇 기준 전진 속도
v_world = R @ v_local            # 월드 기준 속도 벡터
```

### 3-3. 동차좌표와 동차변환행렬 (Homogeneous Coordinates)

회전과 이동을 **하나의 행렬 곱**으로 동시에 처리하기 위해 사용한다.

```python
# 2D 동차좌표
(x, y) → (x, y, 1)

# 2D 동차변환행렬 (회전 + 이동 통합)
[[r11, r12, tx],
 [r21, r22, ty],
 [  0,   0,  1]]
```

**예시 — 회전 후 이동을 한 번에 처리:**

```python
import numpy as np

theta = np.pi / 4   # 45도 회전
tx, ty = 2.0, 1.0  # 이동량

T = np.array([
    [np.cos(theta), -np.sin(theta), tx],
    [np.sin(theta),  np.cos(theta), ty],
    [0,              0,             1 ]
])

p_local = np.array([1.0, 0.0, 1.0])  # 동차좌표로 표현
p_world = T @ p_local                # 월드 좌표로 변환
```

---

## 4. 좌표계(Frame)와 tf2

### 4-1. ROS2 표준 좌표계 계층

```
map → odom → base_link → sensor
```

| frame | 설명 |
|---|---|
| map | 지도 기준 (고정) |
| odom | 이동 누적 기준 |
| base_link | 로봇 본체 기준 |
| sensor | 센서 기준 |

> 좌표 변환에는 방향이 있다.
> 반대 방향으로 변환하려면 역변환이 필요하다.

### 4-2. 좌표축 기본 규칙 (REP-103)

| 축 | 방향 |
|---|---|
| x | 전방 |
| y | 좌측 |
| z | 위쪽 |

### 4-3. tf2

ROS2의 좌표계(Frame) 관리 시스템. 필요한 좌표 변환을 자동으로 계산해 준다.

```python
# 데이터 흐름
joint_state_publisher  : 관절 각도 발행
        ↓
    /joint_states
        ↓
robot_state_publisher  : URDF와 관절 상태를 이용해 각 링크의 위치 계산
        ↓
   /tf, /tf_static     : 계산된 좌표계를 전체 시스템에 제공
```

---

## 5. 변환(Transformation)

```python
# 기본 표현 [R: 회전 행렬, T: 이동 벡터]
world = R * local + T

# 동차행렬 형태
[[R, T],
 [0, 1]]
```

**예시 — 센서 좌표 → 월드 좌표:**

```python
# 센서가 로봇에서 (0.2, 0, 0) 위치에 0도 회전으로 부착된 경우
R = np.eye(2)
T = np.array([0.2, 0.0])

p_sensor = np.array([1.0, 0.5])        # 센서가 감지한 점
p_robot  = R @ p_sensor + T            # 로봇 기준 좌표로 변환
```

---

## 6. Pose

```
pose = position + orientation
```

| 구성 요소 | 설명 |
|---|---|
| position | 위치 `(x, y, z)` |
| orientation | 회전 정보 (쿼터니언 또는 오일러 각도) |

> ROS2에서 `geometry_msgs/Pose`는 position(Point) + orientation(Quaternion)으로 구성된다.

---

## 7. 불확실성과 추정

### 7-1. 공분산 (Covariance)

추정값의 불확실성을 나타낸다. 값이 작을수록 더 신뢰할 수 있는 추정이다.

> ROS2 `PoseWithCovariance`에서 사용 — 위치 추정이 얼마나 확실한지 6×6 행렬로 표현

### 7-2. 오차제곱법 (Least Squares)

오차 제곱의 합이 가장 작아지는 해를 찾는 방법.
→ 센서 노이즈 때문에 점들이 완벽한 직선 위에 있지 않을 때, 가장 그럴듯한 직선을 찾는 방법

> LiDAR 포인트 클라우드에서 벽면 직선 추정, 오도메트리 보정 등에 활용

### 7-3. 고유값과 고유벡터 (Eigenvalue / Eigenvector)

```
고유벡터: 변환 후에도 방향이 유지되는 특별한 방향
고유값:   그 방향으로 얼마나 늘어나거나 줄어드는지 나타내는 값

큰 고유값 → 그 방향으로 불확실성이 큼
작은 고유값 → 그 방향으로 불확실성이 작음
```

> 공분산 행렬의 고유벡터/고유값으로 불확실성의 방향과 크기를 시각화할 수 있다 (오차 타원).
