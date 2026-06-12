# Nav2

로봇이 지도 위에서 목표 위치까지 스스로 이동하게 하기 위한 ROS2 패키지

---

## Nav2 실행 전 반드시 준비해야 하는 것

1. 로봇 모델 또는 TF 구조
2. LiDAR / Odometry / IMU 같은 센서 데이터
3. map 또는 SLAM
4. AMCL 또는 다른 localization 방식
5. planner / controller / costmap 설정
6. nav2_params.yaml

---

## Costmap

로봇의 이동 가능 영역과 장애물 정보를 표현하는 2D Grid Map

### Costmap의 종류

- Global Costmap: 전체 경로 계획용
- Local Costmap: 가까운 장애물 회피와 주행 제어용

---

## Server

- Planner Server: goal과 planner plugin을 사용해 전역 경로를 계산하며, global costmap도 함께 관리
- Controller Server: 경로를 따라가기 위한 제어 요청을 처리하고 local costmap을 호스팅

---

## BT (Behavior Tree)

내비게이션 작업의 실행 순서와 복구 행동을 구성

---

## Recovery Behavior

로봇이 막히거나 실패했을 때 다시 시도할 수 있게 하기 위한 행동

---

## Lifecycle Node

노드의 상태를 명확히 관리하여 안전하게 시작하고 종료하기 위한 노드 관리 방식

```
unconfigured
→ inactive
→ active
→ finalized
```

---

## YAML 파일

- nav2_params.yaml: Nav2 서버들의 동작 파라미터를 설정