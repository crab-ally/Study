# SLAM (Simultaneous Localization And Mapping)

위치와 자세 추정, 지도 작성을 동시에 수행하는 기술

> 위치: 로봇이 어디에 있는지
> 자세: 로봇이 어느 방향을 보고 있는지

---

## 1. LiDAR

레이저를 쏘고 반사되어 돌아오는 시간을 측정해 주변 물체까지의 거리를 계산하는 센서.

```
sensor_msgs/msg/LaserScan       ← ROS2에서 LiDAR 데이터를 담는 메시지 타입
/scan                           ← LiDAR 데이터를 발행하는 토픽 이름
[laser, base_scan, laser_frame] ← LiDAR 센서의 좌표계 (관례적 이름들)
```

**sensor_msgs/msg/LaserScan**

- sensor_msgs: 패키지 이름
- msg: 패키지의 모듈(폴더)
- LaserScan: 메시지 타입(class)

**필드**

- ranges: 각 방향별 거리 측정값 목록 (inf: 측정 가능 물체 없음, NaN: 유효하지 않은 측정값)
- angle_min / angle_max: 측정 시작 각도와 끝 각도
- range_min / range_max: 측정 가능한 최소 거리와 최대 거리

---

## 2. 이동명령

로봇에게 선속도(linear)와 각속도(angular)를 전달하는 메시지.

```
geometry_msgs/msg/Twist ← 이동명령을 담는 메시지 타입
/cmd_vel                ← 이동명령을 발행하는 토픽 이름
```

| 필드 | 의미 |
|---|---|
| `linear.x` | 전진 / 후진 속도 (m/s) |
| `angular.z` | 좌 / 우 회전 각속도 (rad/s) |

---

## 3. Odometry

로봇의 센서(주로 바퀴 엔코더)를 이용해 추정한 위치·자세·속도 정보.
시간이 지날수록 오차가 누적되므로 단독으로 사용하지 않고 SLAM과 함께 보정한다.

```
nav_msgs/msg/Odometry ← Odometry 메시지 타입
/odom                 ← Odometry를 발행하는 토픽 이름 (관례)
```

**필드**

- pose: 로봇의 위치와 자세
- twist: 로봇의 선속도, 각속도 정보

---

## 4. TF2 — 좌표계 관리

여러 좌표계 간의 변환 관계를 관리하는 ROS2 라이브러리.

```
map → odom → base_link → laser
```

```
/tf_static, /tf     ← 좌표 변환 정보를 발행하는 토픽
```

> /tf_static: 정적인 좌표계 변환 정보
> /tf: 동적인 좌표계 변환 정보

| 좌표계 | 설명 |
|---|---|
| `map` | 전역 지도 기준 좌표계 |
| `odom` | 이동 기준 좌표계. 오차 누적 → `map → odom` 변환으로 보정 |
| `base_link` | 로봇 본체의 기준 좌표계 |
| `laser` | LiDAR 센서 기준 좌표계 |

> `odom`은 바퀴 엔코더 기반이라 미끄러짐 등으로 오차가 쌓인다.
> SLAM이 `map → odom` 변환을 지속적으로 보정해 전역 위치를 유지한다.

---

## 5. 지도

```
/map ← 지도 데이터를 발행하는 토픽 이름
```

| 개념 | 설명 |
|---|---|
| Occupancy Grid Map | 공간을 격자로 나눠 각 칸을 **빈 공간 / 장애물 / 미탐색**으로 표현하는 지도 |
| Scan Matching | LiDAR 스캔 데이터를 기존 지도나 이전 스캔과 비교해 로봇 위치를 맞추는 과정 |
| Loop Closure | 로봇이 이전에 방문한 장소를 다시 인식하고 누적 오차를 보정하는 것 |

> SLAM의 정확도는 Scan Matching → Loop Closure 순으로 점진적으로 개선된다.

---

## 6. RViz2

지도, 로봇 위치, LiDAR 스캔, TF를 시각적으로 확인하기 위해 사용하는 ROS2 시각화 도구.

| 주로 시각화하는 항목 |
|---|
| `/map` — Occupancy Grid Map |
| `/scan` — LiDAR 스캔 포인트 |
| `/odom` — 로봇 추정 위치 |
| TF 트리 — 좌표계 간 변환 관계 |

### 6-1. SLAM 결과를 확인할 때 추가해야 하는 Display

- Map → Fixed Frame 설정
- LaserScan
- TF
- RobotModel
- Odometry

---

## 7. 토픽 · 메시지 타입 요약

| 토픽 | 메시지 타입 | 내용 |
|---|---|---|
| `/scan` | `sensor_msgs/msg/LaserScan` | LiDAR 거리 데이터 |
| `/cmd_vel` | `geometry_msgs/msg/Twist` | 로봇 이동명령 (선속도 / 각속도) |
| `/odom` | `nav_msgs/msg/Odometry` | 추정 위치 · 자세 · 속도 |
| `/map` | `nav_msgs/msg/OccupancyGrid` | Occupancy Grid Map |

---

## 센서

- IMU: 로봇의 회전, 기울기, 가속도 정보 제공
- Sensor Fusion: 여러 센서 정보를 결합해 더 안정적인 추정값을 만드는 것

| 센서 | 장점 | 단점 |
|---|---|---|
| Wheel Odometry | 부드럽고 빠름 | 미끄러짐에 약함 |
| IMU | 회전 감지에 강함 | 드리프트 발생 |
| LiDAR | 벽/장애물 인식 우수 | 유리/반사체에 약함 |
| Camera | 풍부한 시각 정보 | 조명 변화에 약함 |

> 드리프트: 시간이 지날수록 측정 오차가 누적되어 실제 값과 점점 달라지는 현상  
> 센서 데이터와 TF 변환의 시간이 맞아야 정확한 위치 계산이 가능함 → 센서 시간 정보

**필드**

- header.frame_id: 기준 좌표계

---

## slam_toolbox

ROS2 Humble에서 2D SLAM을 실습할 때 가장 많이 사용하는 패키지

### 1. 설치

```bash
sudo apt install ros-humble-slam-toolbox
```

### 2. 온라인 모드

로봇이 움직이면서 실시간으로 지도를 만들 때 사용

```bash
ros2 launch slam_toolbox online_async_launch.py
```

> async: 센서 데이터 입력과 SLAM 처리를 비동기적으로 처리
> → 센서 데이터가 들어오는 속도와 SLAM 계산 처리 속도가 항상 정확히 같지 않아도 동작할 수 있는 방식

### 3. 토픽

```
nav_msgs/msg/OccupancyGrid  ← 메시지 타입
/map                        ← 생성된 지도
```

### 4. 필요한 입력 데이터

```
/scan : LiDAR 거리 정보
→ ros2 topic echo /scan

/odom : 로봇 이동 추정

/tf   : 좌표계 관계
→ ros2 run tf2_tools view_frames            # TF 구조 확인
→ ros2 run tf2_ros tf2_echo base_link laser # base_link에서 laser로 가는 좌표 변환을 실시간으로 확인
```

### 5. 중요한 것

- use_sim_time: 시뮬레이션 시간과 ROS2 노드 시간이 일치해야 TF와 센서 데이터가 맞기 때문
→ use_sim_time:=true

- scan_topic: SLAM 노드가 어떤 LiDAR 토픽을 읽을지 결정하기 때문

### 6. YAML 파라미터 파일

SLAM 동작 옵션과 프레임 이름, 토픽 이름 등을 설정하기 위해 사용

```bash
ros2 launch slam_toolbox online_async_launch.py \
params_file:=mapper_params_online_async.yaml
```

---

## nav2_map_server

SLAM으로 만든 지도를 파일로 저장할 때 많이 사용하는 패키지

```bash
# 설치
sudo apt install ros-humble-nav2-map-server

# 지도 저장 (현재 발행 중인 /map 토픽을 파일로 저장)
# my_map.pgm: 실제 지도 이미지
# my_map.yaml: 지도 메타데이터
ros2 run nav2_map_server map_saver_cli -f my_map
```