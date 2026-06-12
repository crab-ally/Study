# SLAM (Simultaneous Localization And Mapping)

위치 추정(Localization)과 지도 작성(Mapping)을 동시에 수행하는 기술

> 위치: 로봇이 어디에 있는지
>
> 자세(Pose): 로봇이 어느 방향을 보고 있는지

---

# 1. SLAM에 사용되는 센서

SLAM은 여러 센서 정보를 이용하여 로봇의 위치를 추정하고 지도를 생성한다.

## 1-1. LiDAR

레이저를 쏘고 반사되어 돌아오는 시간을 측정하여 주변 물체까지의 거리를 계산하는 센서

```text
sensor_msgs/msg/LaserScan       ← ROS2에서 LiDAR 데이터를 담는 메시지 타입
/scan                           ← LiDAR 데이터를 발행하는 토픽 이름
[laser, base_scan, laser_frame] ← LiDAR 센서의 좌표계 (관례적 이름들)
```

### sensor_msgs/msg/LaserScan

- sensor_msgs: 패키지 이름
- msg: 패키지의 모듈(폴더)
- LaserScan: 메시지 타입(class)

### 필드

- ranges: 각 방향별 거리 측정값 목록
  - inf: 측정 가능한 범위 내에 물체 없음
  - NaN: 유효하지 않은 측정값

- angle_min / angle_max: 측정 시작 각도 / 측정 종료 각도

- range_min / range_max: 측정 가능한 최소 거리 / 측정 가능한 최대 거리

---

## 1-2. 기타 센서

- IMU: 로봇의 회전, 기울기, 가속도 정보 제공
- Sensor Fusion: 여러 센서 정보를 결합하여 더 안정적인 추정값 생성

| 센서 | 장점 | 단점 |
|--------|--------|--------|
| Wheel Odometry | 부드럽고 빠름 | 미끄러짐에 약함 |
| IMU | 회전 감지에 강함 | 드리프트 발생 |
| LiDAR | 벽/장애물 인식 우수 | 유리/반사체에 약함 |
| Camera | 풍부한 시각 정보 | 조명 변화에 약함 |

> 드리프트(Drift): 시간이 지날수록 측정 오차가 누적되어 실제 값과 점점 달라지는 현상

---

# 2. 로봇 이동 명령

로봇에게 선속도(Linear Velocity)와 각속도(Angular Velocity)를 전달하는 메시지

```text
geometry_msgs/msg/Twist ← 이동명령을 담는 메시지 타입
/cmd_vel                ← 이동명령을 발행하는 토픽 이름
```

| 필드 | 의미 |
|--------|--------|
| linear.x | 전진 / 후진 속도 (m/s) |
| angular.z | 좌 / 우 회전 각속도 (rad/s) |

---

# 3. Odometry

로봇의 센서(주로 바퀴 엔코더)를 이용해 추정한 위치·자세·속도 정보

시간이 지날수록 오차가 누적되므로 단독으로 사용하지 않고 SLAM과 함께 보정한다.

```text
nav_msgs/msg/Odometry ← Odometry 메시지 타입
/odom                 ← Odometry를 발행하는 토픽 이름 (관례)
```

## 필드

- pose: 로봇의 위치와 자세

- twist: 로봇의 선속도와 각속도

---

# 4. TF2 (좌표계 관리)

여러 좌표계 간의 변환 관계를 관리하는 ROS2 라이브러리

```text
map → odom → base_link → laser
```

```text
/tf_static ← 정적인 좌표계 변환 정보
/tf        ← 동적인 좌표계 변환 정보
```

## 주요 좌표계

| 좌표계 | 설명 |
|----------|----------|
| map | 전역 지도 기준 좌표계 |
| odom | 이동 기준 좌표계 |
| base_link | 로봇 본체 기준 좌표계 |
| laser | LiDAR 센서 기준 좌표계 |

### map → odom 이 필요한 이유

`odom`은 바퀴 엔코더 기반 위치 추정값이라 시간이 지날수록 오차가 누적된다.

SLAM은 Scan Matching 등을 이용해 실제 위치를 계산하고,

```text
map → odom
```

변환을 지속적으로 보정하여 전역 위치를 유지한다.

---

## 센서 시간 동기화

센서 데이터와 TF 변환의 시간이 맞아야 정확한 위치 계산이 가능하다.

### header.frame_id

기준 좌표계(Frame)를 의미한다.

---

# 5. 지도(Map)

```text
/map ← 지도 데이터를 발행하는 토픽 이름
```

## Occupancy Grid Map

공간을 격자로 나누고 각 칸을 다음과 같이 표현하는 지도

- 빈 공간 (Free)
- 장애물 (Occupied)
- 미탐색 영역 (Unknown)

메시지 타입

```text
nav_msgs/msg/OccupancyGrid
```

---

## Scan Matching

현재 LiDAR 스캔 데이터를

- 기존 지도
- 이전 스캔 데이터

와 비교하여 로봇 위치를 추정하는 과정

---

## Loop Closure

로봇이 과거에 방문했던 장소를 다시 인식하여

누적된 위치 오차를 보정하는 과정

---

> SLAM의 정확도는
>
> Scan Matching → Loop Closure
>
> 과정을 거치며 점진적으로 향상된다.

---

# 6. ROS2에서 자주 사용하는 SLAM 관련 토픽

| 토픽 | 메시지 타입 | 내용 |
|--------|--------|--------|
| /scan | sensor_msgs/msg/LaserScan | LiDAR 거리 데이터 |
| /cmd_vel | geometry_msgs/msg/Twist | 로봇 이동 명령 |
| /odom | nav_msgs/msg/Odometry | 추정 위치·자세·속도 |
| /map | nav_msgs/msg/OccupancyGrid | Occupancy Grid Map |

---

# 7. RViz2

지도, 로봇 위치, LiDAR 스캔, TF를 시각적으로 확인하기 위한 ROS2 시각화 도구

## 주로 시각화하는 항목

| 항목 |
|--------|
| /map |
| /scan |
| /odom |
| TF Tree |

---

## SLAM 결과를 확인할 때 추가하는 Display

- Map
- LaserScan
- TF
- RobotModel
- Odometry

### Fixed Frame

보통 다음과 같이 설정한다.

```text
map
```

---

# 8. slam_toolbox

ROS2 Humble에서 가장 많이 사용하는 2D SLAM 패키지

---

## 8-1. 설치

```bash
sudo apt install ros-humble-slam-toolbox
```

---

## 8-2. 온라인 모드

로봇이 움직이면서 실시간으로 지도를 생성

```bash
ros2 launch slam_toolbox online_async_launch.py
```

### async 의미

센서 데이터 입력과 SLAM 계산을 비동기적으로 처리

즉,

- 센서 데이터 수신 속도
- SLAM 계산 속도

가 정확히 같지 않아도 동작할 수 있다.

---

## 8-3. 생성되는 지도

```text
nav_msgs/msg/OccupancyGrid ← 메시지 타입
/map                       ← 생성된 지도 토픽
```

---

## 8-4. 필요한 입력 데이터

### LiDAR

```text
/scan
```

확인

```bash
ros2 topic echo /scan
```

---

### Odometry

```text
/odom
```

---

### TF

```text
/tf
```

확인

```bash
ros2 run tf2_tools view_frames              # TF 구조 확인

ros2 run tf2_ros tf2_echo base_link laser   # 두 좌표계 간 변환 실시간 확인
```

---

## 8-5. 중요한 파라미터

### use_sim_time

시뮬레이터 시간과 ROS2 시간을 일치시킨다.

TF와 센서 데이터의 시간 동기화를 위해 중요하다.

```text
use_sim_time:=true
```

---

### scan_topic

SLAM 노드가 어떤 LiDAR 토픽을 사용할지 결정

---

## 8-6. YAML 파라미터 파일

SLAM 동작 옵션

- 프레임 이름
- 토픽 이름
- SLAM 파라미터

등을 설정하기 위해 사용

```bash
ros2 launch slam_toolbox online_async_launch.py \
params_file:=mapper_params_online_async.yaml
```

---

# 9. nav2_map_server

SLAM으로 만든 지도를 파일로 저장할 때 사용하는 패키지

---

## 설치

```bash
sudo apt install ros-humble-nav2-map-server
```

---

## 지도 저장

현재 발행 중인 `/map` 토픽을 파일로 저장

```bash
ros2 run nav2_map_server map_saver_cli -f my_map
```

생성 파일

```text
my_map.pgm  ← 실제 지도 이미지
my_map.yaml ← 지도 메타데이터
```