# MoveIt2

로봇 팔이 목표 위치까지 안전하게 움직이는 경로를 계산해 주는 ROS2 planning framework

---

## 전체 흐름

```text
URDF (로봇 구조)
   ↓
SRDF (MoveIt 설정)
   ↓
Planning Group 선택
   ↓
목표 위치(Pose) 설정
   ↓
IK 계산
   ↓
Planner 경로 생성
   ↓
충돌 검사
   ↓
경로(Trajectory) 생성
   ↓
실행
```

- URDF
- SRDF
- Planning Scene
- Motion Planner
- Kinematics Solver
- ros2_control: 실제로 로봇을 움직이는 실행 계층

---

## 핵심 개념

1. Robot Description: 로봇의 구조, 관절, 링크, 센서 등을 정의한 XML 파일 (URDF)
2. Planning Group: 로봇 팔의 관절들을 그룹화한 것
3. End Effector: 로봇 팔의 끝 부분
4. Planning Scene: 주변 환경 정보
5. Motion Planning: 목표 위치까지의 경로를 계산하는 알고리즘

---

## 로봇팔 이동

1. Joint Space: 관절 각도를 이용한 이동
2. Task Space: End Effector의 위치와 자세를 이용한 이동

사람은 좌표로 명령(Task Space) → 기구학 → MoveIt2가 관절 각도로 변환(Joint Space)

---

## 기구학

- 정기구학: 관절 각도 → End Effector 위치/자세
- 역기구학: End Effector 위치/자세 → 관절 각도

---

## 파일

- URDF: 로봇의 구조, 관절, 링크, 센서 등을 정의한 XML 파일
- SRDF: URDF 위에 얹는 MoveIt 동작 설정 파일
    - Planning Group 정의
    - End Effector 정의
    - 충돌 무시 설정
    - 기본 자세 정의

---

## 실행 구조

```text
launch 실행
  ↓
Move Group (두뇌)
  ↓
Planning Scene (환경)
  ↓
RViz (UI)
  ↓
사용자 입력 (목표 위치)
  ↓
경로 생성 → 시각화 → 실행
```

---

## Rviz

```text
(사용자) 목표 지정
  ↓
(MoveIt) IK 계산
  ↓
(Planner) 경로 생성
  ↓
충돌 검사
  ↓
실행
```

--

## Planning Scene

로봇이 인식하는 가상의 세계

구성 요소

- 로봇 모델
- 주변 환경 (테이블, 박스 등)
- 충돌 정보
- Attached Object

---

## 노드

- move_group: MoveIt의 두뇌

```
RViz2 또는 Python 코드
        ↓
    move_group
        ↓
Motion Planner / Kinematics / Collision Checking
        ↓
Trajectory 생성
        ↓
Controller 또는 Gazebo / 실제 로봇
```

```
1. 현재 관절 상태 확인
2. 목표 위치에 대한 역기구학 계산
3. 관절 제한 확인
4. 충돌 검사
5. 경로 계획
6. trajectory 생성
7. 실행 요청
```