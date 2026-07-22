# Nav2 (Navigation2)

로봇이 지도 위에서 목표 위치까지 스스로 이동하게 하기 위한 ROS2 패키지

---

# 0. 전체 동작 흐름

```text
센서 - 로봇 상태와 주변 환경 측정
            ↓
Localization - 현재 위치 추정
          (AMCL)
            ↓
       Map - 환경 정보
            ↓
   Costmap 생성 - 장애물 정보
            ↓
        Goal 수신
            ↓
      BT Navigator
            ↓
Planner Server - 경로 생성
            ↓
Controller Server - 경로 추종
            ↓
         /cmd_vel
            ↓
         로봇 이동

실패 시
    ↓
Recovery Behavior
```

---

# 1. Nav2가 동작하기 위해 필요한 것

Nav2는 아래 정보들을 이용해 경로를 만들고 이동한다.

## Nav2 실행 전 반드시 준비해야 하는 것

1. 로봇 모델 또는 TF 구조
2. 센서 데이터 (LiDAR / Odometry / IMU ...)
3. map 또는 SLAM
4. AMCL 또는 다른 localization 방식
5. planner / controller / costmap 설정
6. nav2_params.yaml

---

# 2. Localization (현재 위치 추정)

Nav2는 현재 로봇 위치를 알아야 경로를 계산할 수 있다.

## AMCL

AMCL (Adaptive Monte Carlo Localization)

저장된 지도(map) 위에서 현재 로봇 위치를 추정하는 패키지

```
Particle 생성
      ↓
로봇 이동 - Odometry 수신
      ↓
LiDAR 비교 (현재 위치 - 각 Particle 위치)
      ↓
점수 부여
      ↓
확률 높은 Particle만 남김
      ↓
한 곳으로 모임
```

> Particle Cloud: 로봇 위치 후보들의 분포

---

## /initialpose

AMCL 초기 위치를 알려주는 토픽

→ RViz2에서 **2D Pose Estimate** 버튼을 누르면 내부적으로 `/initialpose` 메시지가 발행

---

# 3. 지도(Map)

## map_server

지도 파일(map.yaml, map.pgm)을 읽어서 ROS2 토픽으로 제공

### 지도 파일

- map.yaml : 지도 메타데이터
- map.pgm : 실제 지도 이미지

---

# 4. Costmap

로봇의 이동 가능 영역과 장애물 정보를 표현하는 2D Grid Map

경로 계획과 장애물 회피에 사용된다.

## Costmap의 종류

### Global Costmap

전체 경로 계획용

### Local Costmap

가까운 장애물 회피와 주행 제어용

---

## Layer

Costmap은 여러 Layer를 합쳐서 생성된다.

| 이름 | 설명 |
|---|---|
| static_layer | map server가 제공하는 정적 지도(고정된 환경 정보)를 costmap에 반영 |
| obstacle_layer | 센서 데이터를 사용해 장애물을 costmap에 표시 |
| inflation_layer | 장애물 주변에 "가까이 가면 위험하다"는 비용 영역 생성 → 벽 바로 옆으로 지나가지 않도록 여유 공간을 주는 기능 |


**obstacle_layer**

- marking: 센서가 본 장애물을 costmap에 표시
- clearing: 센서가 비어 있다고 본 공간의 장애물을 제거

**inflation_layer**

- robot_radius: 로봇을 원(circle)으로 근사
- footprint: 로봇을 다각형(polygon)으로 표현
- inflation_radius: Inflation Layer가 장애물 주변을 얼마나 넓게 위험 영역으로 표시할지 결정하는 파라미터

---

# 5. Nav2 서버(Server)

Nav2는 여러 서버(Node)가 협력하여 동작한다.

## Planner Server

goal과 planner plugin을 사용해 전역 경로를 계산

또한 global costmap도 함께 관리

## Controller Server

계산된 경로를 따라가기 위한 제어 요청을 처리

또한 local costmap을 관리

출력:

```text
/cmd_vel
geometry_msgs/msg/Twist
```

로봇 이동 명령(선속도, 각속도)을 발행

---

## BT Navigator

Behavior Tree를 실행하여 내비게이션 작업 전체를 관리하는 Nav2 서버

---

# 6. Behavior Tree (BT)

내비게이션 작업의 실행 순서와 복구 행동을 구성

XML 구조로 작성

## BT Navigator와의 관계

```text
Goal 수신
    ↓
BT Navigator
    ↓
Behavior Tree 실행
    ↓
Planner 호출
    ↓
Controller 호출
    ↓
필요 시 Recovery 실행
```

---

## bt_xml_filename

Behavior Tree를 담고 있는 XML 파일 경로

---

# 7. Recovery Behavior

로봇이 막히거나 실패했을 때 다시 시도하기 위한 행동

| Recovery Behavior | 의미 |
|---|---|
| Spin | 제자리 회전 |
| BackUp | 뒤로 이동 |
| Wait | 잠시 대기 |
| ClearCostmap | Costmap 장애물 정보 초기화 → 잘못 감지된 장애물 정보가 Costmap에 남아 있을 때 사용 |

---

# 8. Navigation Action

Nav2는 Action 기반으로 동작한다.

## NavigateToPose

하나의 목표 위치까지 이동하라는 요청

→ RViz2의 2D Goal Pose 버튼을 누르면 내부적으로 NavigateToPose 요청 전달

---

## NavigateThroughPoses

여러 개의 목표 위치를 순서대로 지나가라는 요청

---

## ComputePathToPose

목표 위치까지의 경로를 계산

---

## FollowPath

이미 계산된 경로를 따라가도록 Controller Server에 요청

---

## /goal_pose

Nav2 목표 위치를 알려주는 토픽

→ RViz2에서 **2D Goal Pose** 버튼을 누르면 내부적으로 `/goal_pose` 메시지가 발행

---

# 9. Waypoint Follower

여러 개의 경유지를 순서대로 방문하는 기능

내부적으로 NavigateThroughPoses 계열 기능을 활용

---

# 10. DWB Controller

DWB (Dynamic Window Based Controller)

여러 속도 후보를 평가하여 가장 적절한 속도 명령을 선택

목표:

- 경로 추종
- 장애물 회피
- 부드러운 주행

---

## 주요 파라미터

| 파라미터 | 설명 |
|---|---|
| max_vel_x | 로봇의 최대 전진 속도 |
| max_vel_theta | 로봇의 최대 회전 속도 |
| acc_lim_x | 로봇의 전진 가속도 제한 |
| xy_goal_tolerance | 목표 위치에 얼마나 가까이 가면 도착으로 판단할지 결정하는 거리 허용 오차 |
| yaw_goal_tolerance | 목표 방향과 로봇 방향의 허용 오차 |

---

# 11. Lifecycle Node

노드의 상태를 명확히 관리하여 안전하게 시작하고 종료하기 위한 노드 관리 방식

```text
unconfigured (설정 안 된 초기 상태)
    ↓
inactive     (설정은 됐지만 동작 안 함)
    ↓
active       (동작 중)
    ↓
finalized    (종료)
```

현재 상태 확인:

```bash
ros2 lifecycle get 노드명
```

정상 동작 상태:

```text
active
```

---

# 12. YAML 파일

## nav2_params.yaml

Nav2 서버들의 동작 파라미터를 설정

예:

- Planner 설정
- Controller 설정
- Costmap 설정
- Recovery 설정
- BT 설정

---

# 13. Bringup

Nav2 관련 노드들을 한 번에 실행하고 연결하는 과정

```bash
ros2 launch nav2_bringup bringup_launch.py \
  use_sim_time:=True \
  map:=/path/to/map.yaml \
  params_file:=/path/to/nav2_params.yaml
```

| 파라미터 | 설명 |
|---|---|
| use_sim_time | 시뮬레이션 시간 사용 여부 |
| map | 지도 파일 경로 |
| params_file | Nav2 설정 파일 경로 |

---