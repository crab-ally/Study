# 강화학습 & Stable-Baselines3

## 1. 강화학습이란? PyTorch vs Stable-Baselines3

강화학습은 "Agent가 환경과 상호작용하며 보상을 최대화하는 행동을 스스로 학습"하는 방법입니다.

수식과 알고리즘을 처음부터 직접 구현하는 대신, 검증된 형태로 바로 사용할 수 있게 해주는 것이 **Stable-Baselines3**입니다.

| | 설명 |
|---|---|
| **PyTorch** | 딥러닝 모델(신경망)을 직접 만들고 학습시키는 저수준 도구 |
| **Stable-Baselines3** | PyTorch를 기반으로 검증된 강화학습 알고리즘(DQN, PPO 등)을 구현해 놓은 고수준 라이브러리 |

> 💡 PyTorch가 "재료와 도구"라면, SB3는 "이미 완성된 레시피"입니다. 로봇 제어처럼 실무에서는 알고리즘을 처음부터 구현하기보다, 검증된 SB3 알고리즘을 가져다 쓰고 **환경 설계와 보상 설계**에 집중하는 것이 효율적입니다.

---

## 2. 핵심 개념

| 요소 | 설명 | 로봇 제어 예시 |
| --- | --- | --- |
| **Environment** | Agent가 상호작용하는 세계 | Gazebo 시뮬레이션 속 TurtleBot 환경 |
| **Observation** | Agent가 보는 입력값 | LiDAR 센서값, 카메라 이미지, 현재 위치 |
| **Action** | Agent가 내리는 결정 | 좌/우 바퀴 속도, 로봇팔 관절 토크 |
| **Reward** | 행동에 대한 즉각적인 피드백(점수) | 목표 지점에 가까워지면 +, 충돌하면 - |
| **Episode** | 한 번의 시도 | 로봇이 출발지에서 목표지까지 가는 한 번의 시퀀스 |

**추가로 알아두면 좋은 용어**

| 용어 | 설명 |
| --- | --- |
| Policy (정책) | 어떤 Observation을 받았을 때 어떤 Action을 할지 결정하는 규칙(신경망) |
| Agent | 정책을 가지고 실제로 행동을 선택하는 주체 |
| Step | 환경과 한 번 상호작용하는 단위 (행동 1번 → 상태/보상 갱신) |

---

## 3. 전체 학습 흐름

```
1. 환경을 만든다.
2. 환경을 reset() 해서 초기 관측을 받는다.
3. 정책이 관측을 보고 행동을 고른다.
4. 환경이 행동을 받아 다음 상태와 보상을 돌려준다.
5. 이 데이터를 모아서 알고리즘이 정책을 업데이트한다.
6. 반복한다.
```

이 6단계가 강화학습의 전부입니다. 이후에 배우는 모든 내용(알고리즘, 하이퍼파라미터, 보상 설계)은 결국 이 루프를 **더 안정적이고 효율적으로** 만들기 위한 도구일 뿐입니다.

---

## 4. Gymnasium API 이해하기

SB3는 Gymnasium 표준 인터페이스를 기반으로 동작합니다. 핵심 함수는 딱 2개입니다.

**`env.reset()`**

- 환경을 초기 상태로 되돌림
- 시작 상태(Observation)를 반환

```python
obs, info = env.reset()
```

**`env.step(action)`**

- 행동을 환경에 적용
- 다음 상태를 계산
- 보상을 계산
- 종료 여부를 판단

```python
obs, reward, terminated, truncated, info = env.step(action)
```

**기본 루프**

```python
while True:
    action = agent(obs)
    obs, reward, terminated, truncated, _ = env.step(action)

    if terminated or truncated:
        obs = env.reset()
```

---

## 5. 종료 조건: terminated vs truncated

초보자가 가장 헷갈려하는 부분입니다.

- **`terminated`**: 목표 달성, 실패 등 **문제 자체가 끝난 상태** (예: 로봇이 목표에 도달했거나, 넘어졌거나)
- **`truncated`**: 시간 제한 등 **외부 조건으로 잘린 상태** (예: 정해진 스텝 수를 넘겨서 강제 종료)

> 💡 왜 구분할까요? `terminated`는 "환경 자체의 논리적 끝"이고, `truncated`는 "우리가 임의로 정한 제한"입니다. 이 둘을 구분해야 알고리즘이 "진짜로 끝난 것"과 "시간이 없어서 멈춘 것"을 다르게 학습할 수 있습니다.

---

## 6. 알고리즘 3종 세트: DQN, A2C, PPO

| 알고리즘 | 핵심 개념 | 한 줄 요약 | 환경 |
| --- | --- | --- | --- |
| **DQN** | Q값 테이블을 신경망으로 근사 | "이 행동 하면 점수 몇 점?" | 이산 |
| **A2C** | 정책 + 가치 함수 동시에 학습 | "이 행동을 할 확률은?" | 이산, 연속 |
| **PPO** | 정책을 안정적으로, 조금씩 업데이트 | "조금씩만 안전하게 개선" | 이산, 연속 |

> ⚠️ **보상 해킹(Reward Hacking)**: AI가 진짜 목표를 달성한 게 아니라, 보상 함수의 허점을 이용해서 보상만 높이는 현상. 예를 들어 "빨리 도착하면 +보상"만 설계하면, 로봇이 벽을 뚫고 지나가는 편법을 학습할 수도 있습니다. (11장에서 자세히 다룹니다)

### 추가

| 알고리즘 | 핵심 개념 | 환경 | 정책 |
| --- | --- | --- | --- |
| SAC | 보상 최대화 + entropy(탐색)까지 함께 최적화하는 actor-critic 알고리즘 | 연속 | stochastic, off-policy |
| DDPG | deterministic policy gradient와 DQN 계열 아이디어를 결합한 actor-critic 알고리즘 | 연속 | deterministic, off-policy |
| TD3 | DDPG를 개선한 actor-critic 알고리즘 | 연속 | deterministic, off-policy |

---

## 7. 상황별 알고리즘 선택 가이드

| 상황 | 추천 |
| --- | --- |
| 로봇 제어, 연속 제어, 안정성 중요 | PPO |
| 빠른 테스트 | A2C |
| 게임 (버튼 입력) | DQN |

| 로봇 환경 | 추천 |
| --- | --- |
| TurtleBot 이동 | PPO |
| 로봇팔 제어 | PPO |
| 간단 버튼형 시뮬레이션 | DQN |

> 💡 실무 팁: 확신이 없으면 **일단 PPO**로 시작하세요. 대부분의 연속/이산 환경에서 무난하게 동작하며, 하이퍼파라미터에 상대적으로 덜 민감합니다.

---

## 8. 나만의 커스텀 환경 만들기

실무(특히 로봇 제어)에서는 `CartPole` 같은 기본 환경이 아니라, 직접 환경을 설계해야 하는 경우가 대부분입니다.

**환경 설계에 필요한 3요소**

1. `observation_space`: 관측값의 범위와 형태 정의
2. `action_space`: 행동의 범위와 형태 정의
3. `step()` / `reset()`: 위 4단계 흐름(행동 적용 → 보상 계산 → 종료 판단) 구현

```python
import gymnasium as gym
from gymnasium import spaces
import numpy as np
from stable_baselines3 import PPO
from stable_baselines3.common.env_checker import check_env

class SimpleEnv(gym.Env):

    def __init__(self):
        super(SimpleEnv, self).__init__()

        # 상태: 위치 (연속값)
        self.observation_space = spaces.Box(
            low=-10, high=10, shape=(1,), dtype=np.float32
        )

        # 행동: -1 or +1 (좌우 이동)
        self.action_space = spaces.Discrete(2)

        self.state = None
        self.goal = 5.0

    def reset(self, seed=None, options=None):
        super().reset(seed=seed)

        self.state = np.array([0.0], dtype=np.float32)
        return self.state, {}

    def step(self, action):

        # 행동 적용
        if action == 0:
            self.state[0] -= 1
        else:
            self.state[0] += 1

        # 보상 설계
        distance = abs(self.goal - self.state[0])
        reward = -distance

        # 종료 조건
        terminated = distance < 0.5
        truncated = False

        return self.state, reward, terminated, truncated, {}

env = SimpleEnv()
check_env(env)  # 환경이 올바르게 구현되었는지 확인

model = PPO("MlpPolicy", env, verbose=1)
model.learn(total_timesteps=10000)

obs, _ = env.reset()

for _ in range(50):
    action, _ = model.predict(obs)
    obs, reward, terminated, truncated, _ = env.step(action)

    print("state:", obs, "reward:", reward)

    if terminated:
        print("목표 도달!")
        break
```

> 💡 이 예제는 1차원 위치 제어이지만, 구조는 TurtleBot 위치 제어나 로봇팔 관절 제어와 동일합니다. `observation_space`를 LiDAR 벡터로, `action_space`를 관절 토크 범위로 바꾸면 그대로 로봇 제어 환경이 됩니다.

---

## 9. 강화학습이 실패하는 이유

실무에서 강화학습이 잘 안 될 때, 대부분 아래 3가지 중 하나입니다.

1. **보상 설계 실패**: 원하는 행동을 유도하지 못하는 보상 함수
2. **하이퍼파라미터 설정 오류**: 학습이 발산하거나 너무 느림
3. **상태/행동 정의 문제**: Observation/Action이 문제를 풀기에 부족하거나 과도함

이 순서대로 원인을 점검하는 것이 효율적인 디버깅 순서입니다. (13장 참고)

---

## 10. 보상 설계 (Reward Shaping)

보상 설계는 강화학습에서 가장 중요하면서도 가장 어려운 부분입니다.

**좋은 보상 설계의 3원칙**

1. **Dense Reward (촘촘한 보상)**: 매 step마다 피드백을 줘서 학습 신호를 자주 제공
2. **방향성 있는 보상**: 목표 방향으로 유도하는 보상 (예: 목표와의 거리 감소 시 +)
3. **악용 방지**: 편법 행동을 차단하는 페널티 설계 (7장의 보상 해킹 방지)

> 💡 로봇 제어 예시: 단순히 "목표 도달 시 +100"만 주면(Sparse Reward) 학습 초반에 로봇이 우연히 목표에 도달할 확률이 거의 없어 학습이 안 됩니다. 대신 "매 step마다 목표와의 거리가 줄어들면 +" 같은 Dense Reward를 추가하면 학습이 훨씬 빨라집니다.

---

## 11. 하이퍼파라미터 이해하기

| 하이퍼파라미터 | 역할 | 너무 작을 때 | 너무 클 때 |
| --- | --- | --- | --- |
| **gamma (감가율)** | 미래 보상을 얼마나 중요하게 볼 것인가 | 단기 행동만 학습 | 학습 느림 |
| **learning_rate (학습률)** | 정책을 얼마나 빠르게 바꿀 것인가 | 학습 안됨 | 발산 |
| **batch_size** | 한 번의 Gradient 업데이트에 사용할 데이터 개수 | noisy | 안정적이지만 메모리 多 |
| **n_steps** (PPO, A2C) | 학습할 때 사용할 샘플 개수 | 빠르지만 불안정 | 안정적이지만 느림 |
| **train_freq** (DQN) | 학습할 때 사용할 샘플 개수 | 빠르지만 불안정 | 안정적이지만 느림 |

```python
model = PPO(
    "MlpPolicy",        # Multi-Layer Perceptron 기반 정책 → 입력데이터가 숫자 벡터 형태
    env,
    learning_rate=3e-4, # 기본값, 발산하면 낮추기
    gamma=0.99,         # 장기적 목표일수록 높게
    n_steps=2048,       # 안정성 우선이면 크게
    batch_size=64,
    verbose=1           # 로그 출력 수준: 0(없음), 1(기본), 2(상세)
)
```

> 💡 실무 팁: 한 번에 여러 하이퍼파라미터를 바꾸지 마세요. 하나씩 바꾸면서 `rollout/ep_rew_mean` 그래프의 변화를 관찰하는 것이 원인을 특정하는 가장 확실한 방법입니다.  

> ent_coef: 정책 기반, 정책이 다양한 행동을 선택하도록 유도하여 탐색을 장려  
> ε (epsilon): 가치 기반, 일정 확률로 무작위 행동 선택하여 탐색을 장려

---

## 12. 디버깅 방법론

학습이 잘 안 될 때 아래 순서로 점검합니다.

1. **reward 그래프 확인**: TensorBoard의 `ep_rew_mean`이 상승하는지 확인
2. **행동 직접 보기**: `render()` 또는 영상으로 실제 Agent의 행동을 눈으로 확인
3. **랜덤 정책과 비교**: 학습된 정책이 랜덤보다 나은지 비교 (기본적인 sanity check)
4. **환경 단순화**: 문제를 더 쉬운 버전으로 줄여서 알고리즘 자체의 문제인지 확인

> 💡 이 순서는 10장의 "실패 3대 원인"과 연결됩니다. reward 그래프가 아예 안 오르면 보상 설계 문제, 그래프는 오르는데 행동이 이상하면 상태/행동 정의 문제, 학습이 불안정하게 튀면 하이퍼파라미터 문제일 가능성이 높습니다.

---

## 13. 학습 결과 평가하기

학습이 끝나면 반드시 정량적으로 평가합니다.

```python
from stable_baselines3.common.evaluation import evaluate_policy

mean_reward, std_reward = evaluate_policy(
    model,
    env,
    n_eval_episodes=10
)

print("평균 reward:", mean_reward)
print("표준편차:", std_reward)
```

- **평균 reward**: 정책의 전반적인 성능
- **표준편차**: 정책의 일관성 (표준편차가 크면 운에 따라 성능 편차가 큼 → 불안정한 정책)

> 💡 강화학습은 무작위성이 크기 때문에 성능 평가시 여러 번 실행하는 것이 좋습니다.

### Episode 단위 정보 기록

```python
from stable_baselines3.common.monitor import Monitor
import gymnasium as gym

env = gym.make("CartPole-v1")
env = Monitor(env, filename="logs/monitor.csv")
```

---

## 14. 로그 기록 (TensorBoard)

TensorBoard는 학습 로그를 그래프로 확인하는 도구입니다.

```python
# 로그 활성화
model = PPO(
    "MlpPolicy",
    env,
    verbose=1,
    tensorboard_log="./logs"  # 해당 폴더에 로그 기록
)
```

```bash
# 실행
tensorboard --logdir=./logs
```

`http://localhost:6006` 접속 후 확인할 주요 지표:

| 지표 | 의미 |
| --- | --- |
| `rollout/ep_rew_mean` | 평균 보상 (핵심 지표, 우상향해야 정상) |
| `rollout/ep_len_mean` | 에피소드 평균 길이 |
| `train/loss` | 학습 안정성 (급격히 튀면 발산 신호) |

---

## 15. 영상으로 결과 확인하기

숫자만으로는 Agent가 실제로 "말이 되는" 행동을 하는지 알기 어렵습니다. 반드시 영상으로도 확인합니다.

```python
from gymnasium.wrappers import RecordVideo

env = gym.make("CartPole-v1", render_mode="rgb_array")

env = RecordVideo(
    env,
    video_folder="./videos",  # 해당 폴더에 영상 기록
    episode_trigger=lambda x: True
)
```

---

## 16. 전체 통합 예제

지금까지 배운 내용을 하나로 합친 전체 파이프라인입니다. 이 구조를 템플릿으로 삼아 실무 환경에 맞게 확장하면 됩니다.

```python
import gymnasium as gym
from stable_baselines3 import PPO
from stable_baselines3.common.evaluation import evaluate_policy
from gymnasium.wrappers import RecordVideo

# 1. 환경 생성
env = gym.make("CartPole-v1")

# 2. 모델 생성 (로그 포함)
model = PPO(
    "MlpPolicy",
    env,
    verbose=1,
    tensorboard_log="./logs"
)

# 3. 학습
model.learn(total_timesteps=10000)

# 4. 평가
mean_reward, std_reward = evaluate_policy(model, env, n_eval_episodes=10)
print("평균:", mean_reward, "표준편차:", std_reward)

# 5. 영상 저장
video_env = gym.make("CartPole-v1", render_mode="rgb_array")
video_env = RecordVideo(video_env, "./videos", episode_trigger=lambda x: True)

obs, _ = video_env.reset()
for _ in range(500):
    action, _ = model.predict(obs)
    obs, _, terminated, truncated, _ = video_env.step(action)

    if terminated or truncated:
        obs, _ = video_env.reset()

video_env.close()
```

---

## 17. 모델 저장과 로드

```python
# 학습된 모델 저장
model.save("ppo_cartpole")

# 학습된 모델 로드
model = PPO.load("ppo_cartpole")
```

> 💡 환경 설정을 기록해야 나중에 같은 환경 설정을 불러와서 사용할 수 있습니다. (yaml, json 등)

---

## 18. ROS2 + 강화학습

지금까지는 시뮬레이터 없이 SB3 기본 환경(`CartPole` 등)이나 직접 만든 `SimpleEnv`로 연습했습니다. 실제 로봇에 적용하려면 **ROS2를 Gym Environment로 감싸는 것**이 핵심입니다. 9장에서 배운 커스텀 환경 구조가 그대로 확장됩니다.

```
[ Stable-Baselines3 ]
        ↓
    (action)
        ↓
[ ROS2 Node ]
        ↓
[ Robot / Simulator ]
        ↓
   (sensor data)
        ↓
[ State Processing ]
        ↓
     (reward)
        ↓
[ RL Environment ]
        ↓
[ SB3 학습 업데이트 ]
```

**ROS2 개념 ↔ RL 개념 매핑**

| ROS2 | RL |
| --- | --- |
| Topic (sensor) | Observation |
| Topic (cmd_vel) | Action |
| Node | Environment |
| Service | Reset |
| Action | Episode control |

### 18-1. 기본 구조: ROS2 전체를 step() 안에 넣기

```python
class ROS2Env(gym.Env):

    def __init__(self):
        super().__init__()

        self.observation_space = ...
        self.action_space = ...

    def reset(self):
        # ROS2 서비스 호출 (환경 초기화)
        return state, {}

    def step(self, action):

        # 1. ROS2 토픽으로 행동 전송
        # publish(cmd_vel)

        # 2. 센서 데이터 수신
        # subscribe(sensor)

        # 3. 상태 구성
        state = ...

        # 4. 보상 계산
        reward = ...

        # 5. 종료 조건
        terminated = ...
        truncated = False

        return state, reward, terminated, truncated, {}
```

### 18-2. 데이터 흐름

```
# Action
SB3 → action → ROS2 Publisher → cmd_vel

# Observation
LiDAR / Camera / Joint → ROS2 Subscriber → state vector

# Reward
목표 거리 / 충돌 / 안정성 → reward 계산

# Reset
ROS2 Service → 위치 초기화
```

### 18-3. TurtleBot 예시

9장 `SimpleEnv`의 1차원 위치 제어를 실제 TurtleBot 이동 문제로 확장한 형태입니다.

```python
# Observation
state = [
    distance_to_goal,
    angle_to_goal,
    lidar_min_distance
]

# Action
action = [
    linear_velocity,
    angular_velocity
]

# Reward
reward = -distance_to_goal

if collision:
    reward -= 100

if goal_reached:
    reward += 100

# 종료 조건
terminated = collision or goal_reached
```

> 💡 11장의 보상 설계 3원칙과 연결됩니다: `-distance_to_goal`은 Dense Reward, 충돌 페널티는 악용 방지, 목표 도달 보너스는 방향성 있는 보상에 해당합니다.

**전체 코드 흐름**

1. SB3가 action 생성
2. ROS2에 publish
3. 로봇 움직임
4. 센서 데이터 수신
5. state 구성
6. reward 계산
7. SB3 업데이트

### 18-4. 시스템 아키텍처

```
[ SB3 Trainer ]
        ↓
[ Gym Wrapper (ROS2Env) ]
        ↓
[ ROS2 Bridge Node ]
        ↓
[ Simulator (Gazebo / MuJoCo) ]
        ↓
[ Sensor Streams ]
        ↓
[ Reward Engine ]
        ↓
[ Logging / Monitoring ]
```

### 18-5. 실무 고려 사항

1. **동기화 문제**
   - `step()` 타이밍이 중요
   - ROS2는 비동기 구조 → 강제로 동기화 필요

2. **센서 노이즈**
   - 실제 환경은 시뮬레이션과 다름 (Sim-to-Real Gap)

3. **지연(latency)**
   - action → 반응까지 지연이 존재

4. **reward 설계**
   - 현실적이어야 함 (11장 보상 설계 원칙 적용)

5. **안전성**
   - 로봇 폭주 방지 장치 필수 (예: 속도 제한, 비상 정지 조건)

**핵심 흐름 한 줄 요약**

```
SB3 → Env → Action → Robot → Sensor → State → Reward → SB3
```

---