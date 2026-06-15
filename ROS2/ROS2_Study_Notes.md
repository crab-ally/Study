# ROS2

---

## 명령어

```bash
ros2 topic list # 토픽 목록 확인
ros2 topic echo 토픽명 # 토픽 내용 확인
ros2 topic info 토픽명 # 토픽 정보 확인
ros2 topic hz 토픽명 # 토픽 주파수(발행주기) 확인

ros2 node list # 노드 목록 확인
ros2 node info 노드명 # 노드 정보 확인

ros2 service list # 서비스 목록 확인
ros2 service call 서비스명 [서비스 인자] # 서비스 호출

ros2 action list # 액션 목록 확인
ros2 action info 액션명 # 액션 정보 확인
ros2 action send_goal 액션명 [액션 인자] # 액션 요청
```

## 심화

```bash
ros2 run tf2_tools view_frames # TF 구조 그래프 형태로 시각화

ros2 param list # 현재 실행 중인 노드들의 파라미터 목록 확인
ros2 param get 노드명 파라미터명 # 파라미터 값 확인
```