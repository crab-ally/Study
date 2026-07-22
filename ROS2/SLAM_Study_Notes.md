# SLAM (Simultaneous Localization And Mapping)

위치 추정(Localization)과 지도 작성(Mapping)을 동시에 수행하는 기술

> 위치(Position): 로봇이 어디에 있는가  
> 자세(Pose): 로봇이 어느 방향을 보고 있는가

SLAM의 목표

```text
센서 데이터
      ↓
현재 위치 추정
      ↓
지도 생성
      ↓
위치 오차 보정
```

---

# 1. SLAM에 사용되는 센서

SLAM은 여러 센서를 이용해 주변 환경을 인식하고 위치를 추정한다.

## 1-1. LiDAR

레이저를 발사하고 반사되어 돌아오는 시간을 측정하여 거리를 계산하는 센서

```text
sensor_msgs/msg/LaserScan       ← ROS2에서 LiDAR 데이터를 담는 메시지 타입
/scan                           ← LiDAR 데이터를 발행하는 토픽 이름
[laser, base_scan, laser_frame] ← LiDAR 센서의 좌표계 (관례적 이름들)
```

### sensor_msgs/msg/LaserScan

- sensor_msgs: 패키지 이름
- msg: 패키지의 모듈(폴더)
- LaserScan: 메시지 타입(class)

### LaserScan 주요 필드

| 필드 | 의미 |
|--------|--------|
| ranges | 거리 측정값 배열 (inf - 측정 범위 안에 물체가 없음, NaN - 유효하지 않은 측정값) |
| angle_min / angle_max | 시작 각도 / 종료 각도 |
| range_min / range_max | 최소 측정 거리 / 최대 측정 거리 |

---

## 1-2. IMU

관성 센서

**측정 정보**

- 가속도
- 각속도
- 기울기

**장점**

- 회전 감지에 강함

**단점**

- 드리프트 발생

> 드리프트(Drift): 시간이 지날수록 오차가 누적되는 현상

---

## 1-3. Wheel Odometry

바퀴 엔코더를 이용한 이동량 계산

**장점**

- 부드럽고 빠름

**단점**

- 바퀴 미끄러짐에 약함

---

## 1-4. Camera

영상 기반 환경 인식

**장점**

- 풍부한 시각 정보

**단점**

- 조명 변화에 민감

---

## 1-5. Sensor Fusion

여러 센서를 결합하여 더 정확한 위치를 추정

예시

```text
Wheel Odometry
      +
IMU
      +
LiDAR
```

---

# 2. 로봇 이동 명령

로봇은 선속도와 각속도를 이용하여 움직인다.

```text
geometry_msgs/msg/Twist ← 이동명령을 담는 메시지 타입
/cmd_vel                ← 이동명령을 발행하는 토픽 이름
```

## 주요 필드

| 필드 | 의미 |
|--------|--------|
| linear.x | 전진/후진 속도 (m/s) |
| angular.z | 회전 속도 (rad/s) |

예시

```text
linear.x  = 0.3
angular.z = 0.0
```

→ 직진

```text
linear.x  = 0.0
angular.z = 1.0
```

→ 제자리 회전

---

# 3. Odometry

센서(주로 엔코더)를 이용해 계산한 위치 추정값

```text
nav_msgs/msg/Odometry ← Odometry 메시지 타입
/odom                 ← Odometry를 발행하는 토픽 이름 (관례)
```

## 주요 필드

| 필드 | 의미 |
|--------|--------|
| pose | 위치 + 자세 |
| twist | 선속도 + 각속도 |

## 한계

Odometry는 시간이 지날수록 오차가 누적된다.

```text
실제 위치
      ↑
    오차 증가
      ↑
Odometry
```

따라서 LiDAR, IMU 등을 이용해 지속적으로 보정해야 한다.

---

# 4. TF2 (좌표계 관리)

여러 좌표계(Frame) 간의 관계를 관리하는 ROS2 라이브러리

## 대표적인 TF 구조

```text
map → odom → base_link → laser
```

## 주요 좌표계

| 좌표계 | 의미 |
|----------|----------|
| map | 전역 지도 기준 |
| odom | Odometry 기준 |
| base_link | 로봇 본체 기준 |
| laser | LiDAR 기준 |

---

## TF 토픽

```text
/tf_static ← 정적인 좌표계 변환 정보
/tf        ← 동적인 좌표계 변환 정보
```

---

## map → odom 이 필요한 이유

Odometry는 시간이 지나면서 오차가 누적된다.

SLAM은 LiDAR 데이터를 이용해 실제 위치를 추정하고

```text
map → odom
```

변환을 지속적으로 수정하여 위치 오차를 보정한다.

---

## header.frame_id

메시지가 어떤 좌표계를 기준으로 하는지 나타낸다.

예시

```python
msg.header.frame_id = "laser"
```

---

## 센서 시간 동기화

SLAM은 센서 데이터와 TF의 시간이 일치해야 한다.

시간이 맞지 않으면

```text
Lookup would require extrapolation
```

과 같은 TF 오류가 발생할 수 있다.

---

# 5. 지도(Map)

```text
nav_msgs/msg/OccupancyGrid ← 지도 메시지 타입
/map                       ← 지도를 발행하는 토픽 이름
```

---

## Occupancy Grid Map

공간을 격자로 나눈 지도

각 셀은 다음 상태 중 하나를 가진다.

| 상태 | 의미 | RViz 색상 |
|---|---|---|
| Free | 빈 공간 | 흰색 |
| Occupied | 장애물 | 검은색 |
| Unknown | 미탐색 영역 | 회색 |

---

# 6. SLAM의 핵심 개념

## Scan Matching

현재 LiDAR 스캔을

- 기존 지도
- 이전 스캔

과 비교하여 현재 위치를 계산

```text
현재 Scan
      ↓
기존 지도와 비교
      ↓
위치 추정
```

---

## Loop Closure

이전에 방문했던 장소를 다시 발견하여 누적 오차를 보정

```text
처음 방문
      ↓
이동
      ↓
다시 방문
      ↓
오차 수정
```

---

## SLAM 위치 보정 흐름

```text
Odometry
      ↓
Scan Matching
      ↓
Loop Closure
      ↓
정확한 위치
```

---

# 7. RViz2

ROS2 시각화 도구

SLAM 결과를 확인할 때 사용한다.

- 2D Goal Pose: Nav2 목표 위치를 지정할 때 주로 사용하는 버튼
- 2D Pose Estimate: AMCL 초기 위치를 지정할 때 주로 사용하는 버튼

---

## 자주 보는 항목

| Display |
|---|
| Map |
| LaserScan |
| TF |
| RobotModel |
| Odometry |

---

## SLAM 결과 확인

추가할 Display

```text
Map
LaserScan
TF
RobotModel
Odometry
```

---

## LiDAR 확인

```text
LaserScan
Topic: /scan
```

---

## 지도 확인

```text
Map
Topic: /map
```

---

## TF 구조 확인

```text
TF
```

---

# 8. slam_toolbox

ROS2 Humble에서 가장 널리 사용되는 2D SLAM 패키지

> Cartographer는 Google이 개발한 2D/3D SLAM 패키지

---

## 설치

```bash
sudo apt install ros-humble-slam-toolbox
```

---

## 온라인 모드

로봇이 움직이면서 실시간으로 지도를 생성

```bash
ros2 launch slam_toolbox online_async_launch.py
```

---

### async 의미

센서 수신과 SLAM 계산을 비동기적으로 처리

즉,

- 센서 데이터 수신 속도
- SLAM 계산 속도

가 정확히 같지 않아도 동작할 수 있다.

---

## 생성 결과

```text
nav_msgs/msg/OccupancyGrid
/map
```

---

## SLAM에 필요한 입력

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
ros2 run tf2_ros tf2_echo base_link laser   # TF 변환 실시간 확인
```

---

## 주요 파라미터

### use_sim_time

시뮬레이터 시간과 ROS2 시간을 일치시킨다.

```text
use_sim_time:=true
```

TF와 센서 데이터의 시간 동기화를 위해 중요하다.

---

### scan_topic

SLAM 노드가 사용할 LiDAR 토픽

예시

```text
scan_topic: /scan
```

---

## YAML 파라미터 파일

SLAM 동작 설정 파일

설정 예

- 프레임 이름
- 토픽 이름
- SLAM 파라미터

실행

```bash
ros2 launch slam_toolbox online_async_launch.py \
params_file:=mapper_params_online_async.yaml
```

---

# 9. 지도 저장 및 불러오기

## nav2_map_server

지도 저장 및 로드 기능 제공

설치

```bash
sudo apt install ros-humble-nav2-map-server
```

---

## 지도 저장

현재 `/map` 저장

```bash
ros2 run nav2_map_server map_saver_cli -f my_map
```

생성 파일

```text
my_map.pgm  ← 실제 지도 이미지
my_map.yaml ← 지도 메타데이터
```

---

## my_map.yaml

| 항목 | 의미 |
|--------|--------|
| image | 지도 이미지 경로 |
| resolution | m/pixel |
| origin | 지도 원점 |
| occupied_thresh | 장애물 임계값 |
| free_thresh | 빈 공간 임계값 |

---

# 10. AMCL (Localization)

Adaptive Monte Carlo Localization

이미 만들어진 지도에서 현재 위치만 추정

> Mapping 없이 Localization만 수행

---

## 동작 흐름

```text
저장된 지도 로드
        ↓
AMCL 실행 (지도와 LaserScan 데이터 필요)
        ↓
2D Pose Estimate
        ↓
초기 위치와 방향 지정
        ↓
LiDAR와 지도 매칭
        ↓
위치 추정
```

> 2D Pose Estimate: 로봇의 초기 위치와 방향을 지정하는 도구
> 2D Goal Pose: Nav2에서 로봇에게 목표 위치와 방향을 지정하는 도구

/initialpose: 로봇의 초기 위치를 알려주는 토픽
→ RViz2에서 2D Pose Estimate 버튼을 누르면 내부적으로 /initialpose 메시지가 발행

- 메시지 타입: `geometry_msgs/msg/PoseWithCovarianceStamped`

---

## SLAM과 AMCL 차이

| 항목 | SLAM | AMCL |
|---|---|---|
| 지도 생성 | O | X |
| 위치 추정 | O | O |
| 기존 지도 필요 | X | O |

---

# 11. rosbag

ROS2 토픽 데이터를 기록하고 재생하는 도구

---

## 지도 생성 시 기록 권장 토픽

```text
/scan
/odom
/tf
/tf_static
```

시뮬레이션이라면

```text
/clock
```

도 함께 기록

---