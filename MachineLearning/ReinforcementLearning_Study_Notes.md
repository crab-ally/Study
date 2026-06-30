# 강화학습(Reinforcement Learning)

## 1. 강화학습이란

강화학습(Reinforcement Learning, RL)은 **보상의 누적치를 최대화하도록 행동을 학습하는 방법**입니다.

쉽게 비유하면, 강아지에게 "앉아"를 가르칠 때 정답을 알려주는 게 아니라 잘했을 때 간식(보상)을 주는 방식과 같습니다. 강아지는 정답표 없이도 "어떤 행동을 하면 간식을 더 많이 받는지"를 스스로 시행착오를 통해 학습합니다.

강화학습이 다른 머신러닝 분야와 구분되는 핵심 특징은 다음 세 가지입니다.

| 특징 | 설명 |
|---|---|
| 순차적 의사결정 | 한 번의 선택이 끝이 아니라, 선택 → 결과 → 다음 선택으로 이어짐 |
| 지연된 보상 (Delayed Reward) | 지금 한 행동의 결과가 한참 뒤에 나타날 수 있음 (예: 체스의 한 수) |
| 정답 데이터 없음 | "정답 행동"이 주어지지 않고, 직접 경험하며 좋은 행동을 찾아야 함 |

**대표 적용 분야**: 게임 AI(Atari, AlphaGo), 로봇 제어(보행, 매니퓰레이션), 자율주행, 추천 시스템, 데이터센터 자원 관리, LLM 정렬(RLHF) 등

---

## 2. 핵심 구성 요소: Agent, Environment, State, Action, Reward

강화학습의 모든 알고리즘은 다음 다섯 가지 요소의 상호작용으로 설명됩니다.

| 요소 | 의미 | 예시 (게임) | 예시 (로봇) |
| --- | --- | --- | --- |
| Agent | 행동하는 주체 | 게임 캐릭터 | 로봇 팔, 모바일 로봇 |
| Environment | Agent가 상호작용하는 대상 | 게임 맵 | 작업 공간, 시뮬레이터(MuJoCo, Gazebo) |
| State (s) | 현재 상황에 대한 정보 | 캐릭터 위치, 체력 | 관절 각도, LiDAR 스캔, 카메라 이미지 |
| Action (a) | Agent가 취할 수 있는 행동 | 이동, 점프 | 토크 명령, 속도 명령 |
| Reward (r) | 행동에 대한 즉각적인 피드백(점수) | 적 처치 시 +1 | 목표 지점 도달 시 +10, 충돌 시 -5 |

**전체 상호작용 루프**

```
1. Agent가 Environment로부터 현재 State를 관찰
2. Agent가 Policy에 따라 Action을 선택
3. Environment가 Action을 반영하여 새로운 State와 Reward를 반환
4. Agent는 받은 Reward를 바탕으로 더 좋은 행동을 하도록 업데이트
5. 반복
```

이 루프가 강화학습 전체를 관통하는 가장 기본적인 구조입니다. 이후 모든 알고리즘(Q-Learning, DQN, PPO 등)은 결국 "4번 업데이트를 어떻게 더 잘 할 것인가"에 대한 서로 다른 해법일 뿐입니다.

---

## 3. 지도학습 vs 비지도학습 vs 강화학습

| 구분 | 지도학습 (Supervised) | 비지도학습 (Unsupervised) | 강화학습 (RL) |
|---|---|---|---|
| 데이터 | 정답(label) 있음 | 정답 없음, 구조만 있음 | 정답 없음, 보상만 있음 |
| 목표 | 정답을 맞추기 | 데이터의 숨겨진 패턴 찾기 | 누적 보상을 최대화 |
| 피드백 시점 | 즉시 (매 샘플마다 정답 비교) | 없음 | 지연될 수 있음 (Delayed Reward) |
| 데이터 생성 방식 | 미리 수집됨 (고정) | 미리 수집됨 (고정) | Agent가 행동하며 직접 생성 |
| 예시 | 이미지 분류, 회귀 | 군집화, 차원 축소 | 게임 플레이, 로봇 제어 |

가장 중요한 차이는 **데이터가 고정되어 있는지 여부**입니다. 지도학습/비지도학습은 정해진 데이터셋을 보고 학습하지만, 강화학습은 Agent가 행동을 바꾸면 앞으로 만나게 될 데이터(경험) 자체가 바뀝니다. 이 때문에 강화학습은 "데이터 분포가 학습 중에 계속 변하는" 독특한 학습 문제를 풉니다.

---

## 4. MDP (Markov Decision Process)

강화학습 문제는 수학적으로 **MDP(마르코프 결정 과정)** 로 정의됩니다. MDP는 다음 5개 요소의 튜플 `(S, A, P, R, γ)` 로 표현됩니다.

| 요소 | 의미 |
|---|---|
| S (State space) | 가능한 모든 상태의 집합 |
| A (Action space) | 가능한 모든 행동의 집합 |
| P (Transition Probability) | `P(s'\|s,a)`: 상태 s에서 행동 a를 했을 때 s'로 이동할 확률 |
| R (Reward function) | `R(s,a)`: 상태 s에서 행동 a를 했을 때 받는 보상 |
| γ (Discount factor) | 미래 보상을 현재 가치로 환산하는 할인율 (0~1) |

**마르코프 성질(Markov Property)**: "다음 상태는 오직 현재 상태와 현재 행동에만 의존하며, 과거 이력과는 무관하다"는 가정입니다.

```
P(s_{t+1} | s_t, a_t) = P(s_{t+1} | s_t, a_t, s_{t-1}, a_{t-1}, ..., s_0, a_0)
```

> 로봇 예시: 다음 순간 로봇의 위치는 "지금 위치 + 지금 낸 속도 명령"만으로 결정되며, 5초 전에 어디 있었는지는 중요하지 않습니다. (단, State에 충분한 정보 — 속도, 가속도 등 — 가 포함되어 있어야 이 가정이 성립합니다.)

**왜 MDP를 먼저 알아야 하는가**: 앞으로 배울 Policy, Value Function, Bellman Equation은 모두 "MDP가 주어졌을 때 최적의 행동 전략을 찾는 방법"입니다. MDP는 강화학습 전체의 수학적 무대입니다.

---

## 5. Policy, Value Function, Q-Value

### Policy (π) — 행동 전략

Policy는 상태를 입력받아 행동을 결정하는 함수입니다.

```
π(s) → a          (Deterministic, 결정적: 항상 같은 행동)
π(a|s) → 확률      (Stochastic, 확률적: 행동을 확률적으로 선택)
```

| 유형 | 설명 | 사용 예 |
|---|---|---|
| Deterministic | 같은 상태에서 항상 같은 행동 | Q-Learning, DQN의 최종 정책 |
| Stochastic | 같은 상태에서도 확률적으로 다른 행동 | Policy Gradient, PPO |

### Value Function V(s) — 상태의 가치

특정 상태 s에서 시작해서 Policy를 따라갔을 때 **앞으로 얻을 것으로 기대되는 총 누적 보상**입니다.

```
V(s) = E[ R_t + γR_{t+1} + γ²R_{t+2} + ... | s_t = s ]
```

### Q-Value (Action-Value Function) Q(s,a)

상태 s에서 **행동 a를 했을 때**의 기대 누적 보상입니다. V(s)와 다르게 "어떤 행동을 했을 때"까지 포함합니다.

```
Q(s,a) = E[ R_t + γR_{t+1} + ... | s_t = s, a_t = a ]
```

### V(s)와 Q(s,a)의 관계

```
V(s) = max_a Q(s,a)
```

> 상태 s에서의 가치는, 그 상태에서 선택 가능한 모든 행동 중 가장 좋은 행동의 Q값과 같습니다. 이 관계는 이후 챕터 12의 Dueling DQN에서 다시 핵심적으로 사용됩니다.

---

## 6. Bellman Equation

Bellman Equation은 강화학습 전체를 떠받치는 가장 중요한 수식입니다. 핵심 아이디어는 단순합니다.

> **현재 가치 = 지금 받는 보상 + (할인된) 미래 가치**

```
Q(s,a) = r(s,a) + γ * max_a' Q(s',a')
```

| 요소 | 의미 |
| --- | --- |
| r | 지금 행동으로 받는 즉시 보상 |
| γ (감마) | 미래를 얼마나 중요하게 볼지 정하는 할인율 (0에 가까울수록 근시안적, 1에 가까울수록 장기적) |
| s' | 행동 a 이후 도달하는 다음 상태 |
| max_a' Q(s',a') | 다음 상태에서 가능한 가장 좋은 선택의 가치 |

이 수식이 중요한 이유는, **"전체 미래"를 직접 다 계산하지 않고도** "지금 보상 + 다음 상태의 가치"만 알면 현재의 가치를 계산할 수 있게 해준다는 점입니다. 이런 재귀적 구조 덕분에 Q-Learning, DQN 같은 알고리즘이 한 스텝씩 점진적으로 학습할 수 있습니다.

---

## 7. Monte Carlo vs TD Learning

Bellman Equation의 "미래 가치"를 실제로 추정하는 방법에는 크게 두 가지 접근이 있습니다.

| 구분 | Monte Carlo (MC) | TD (Temporal Difference) |
|---|---|---|
| 업데이트 시점 | 에피소드가 끝난 후 | 매 스텝마다 (끝까지 안 기다림) |
| 사용하는 값 | 실제로 받은 전체 보상 합 | 다음 상태의 "추정값"으로 부트스트랩 |
| 분산(Variance) | 높음 (실제 경험 그대로 사용) | 낮음 |
| 편향(Bias) | 없음 | 약간 있음 (추정값을 추정값으로 업데이트) |
| 학습 속도 | 느림 (에피소드 끝까지 기다림) | 빠름 (온라인 학습 가능) |

```
MC:  V(s) ← V(s) + α * (G_t - V(s))          # G_t: 에피소드 끝까지의 실제 누적 보상
TD:  V(s) ← V(s) + α * (r + γV(s') - V(s))   # 다음 상태 추정치로 즉시 업데이트
```

> 실무에서는 TD 방식이 압도적으로 많이 쓰입니다. 에피소드가 매우 길거나(예: 로봇이 몇 시간 동안 동작), 끝나지 않는 환경(continuing task)에서는 MC를 적용하기 어렵기 때문입니다. 다음 챕터의 Q-Learning은 TD 방식의 대표적인 예입니다.

---

## 8. Q-Learning

Q-Learning은 **Q값을 반복적으로 업데이트해서 최적 정책을 찾는** TD 기반 알고리즘입니다.

```
Q(s,a) ← Q(s,a) + α * (r + γ * max_a' Q(s',a') - Q(s,a))
```

| 요소 | 의미 |
| --- | --- |
| α (알파) | 학습률: 한 번에 얼마나 업데이트할지 |
| r | 받은 보상 |
| γ | 미래 중요도 |
| Q(s,a) | 현재 추정 중인 Q값 |
| (r + γ·max Q − Q) | **TD Error**: "실제로 겪어보니 알게 된 값"과 "기존 추정값"의 차이 |

**TD Error가 핵심입니다.** 이 값이 양수면 "생각보다 좋았다"는 뜻이므로 Q값을 올리고, 음수면 "생각보다 나빴다"는 뜻이므로 Q값을 내립니다.

**Q-Learning 흐름**

```
1. 상태 s 관찰
2. 행동 a 선택
3. 보상 r 수신, 다음 상태 s' 도달
4. Q(s,a) ← Q(s,a) + α(r + γ·max Q(s',a') − Q(s,a))
5. s ← s', 반복
```

**한계**: Q-Learning은 모든 (s, a) 쌍에 대한 값을 **표(Q-table)** 로 저장합니다. 상태 공간이 작을 때는 문제없지만, 카메라 이미지나 연속적인 로봇 관절 각도처럼 상태가 무한히 많을 경우 표로 저장하는 것이 불가능합니다. 이 한계를 해결하기 위해 챕터 11의 DQN이 등장합니다.

---

## 9. Exploration vs Exploitation & ε-greedy

학습 중인 Agent는 항상 다음 딜레마에 부딪힙니다.

| 전략 | 설명 |
|---|---|
| Exploration (탐험) | 아직 안 해본 새로운 행동을 시도 |
| Exploitation (활용) | 지금까지 가장 좋았던 행동을 반복 |

> 너무 Exploitation만 하면 더 좋은 행동이 있어도 못 찾고 지역 최적해에 갇힙니다. 너무 Exploration만 하면 학습한 지식을 활용하지 못해 비효율적입니다.

**ε-greedy 전략**: 가장 널리 쓰이는 균형 방법입니다.

```
ε(엡실론) 확률로 → 무작위 행동 (Exploration)
1-ε 확률로     → 현재 Q값이 가장 큰 행동 (Exploitation)
```

학습이 진행될수록 ε을 점점 줄여나갑니다 (예: `epsilon *= 0.995` 매 에피소드마다). 학습 초반에는 아직 Q값이 부정확하므로 탐험을 많이 하고, 후반에는 학습된 지식을 신뢰하고 활용 비중을 늘리는 것이 일반적인 전략입니다.

---

## 10. On-policy vs Off-policy: SARSA vs Q-Learning

Q-Learning의 업데이트 식을 다시 보면 흥미로운 특징이 있습니다.

```
Q-Learning:  Q(s,a) ← Q(s,a) + α(r + γ·max_a' Q(s',a') − Q(s,a))   # 다음 행동 a'은 "최선"으로 가정
SARSA:       Q(s,a) ← Q(s,a) + α(r + γ·Q(s',a') − Q(s,a))          # 다음 행동 a'은 "실제로 선택한" 행동
```

| 구분 | Q-Learning | SARSA |
|---|---|---|
| 분류 | Off-policy | On-policy |
| 업데이트에 쓰는 다음 행동 | 가능한 최선의 행동(max) | 실제로 ε-greedy로 선택한 행동 |
| 학습 대상 | 최적 정책 (행동 방식과 무관하게) | 지금 따르고 있는 정책 그 자체 |
| 특징 | 더 적극적이고 공격적인 학습 | 더 보수적이고 안전한 학습 |

**Off-policy**는 "행동하는 정책"과 "학습하는 정책"이 달라도 됩니다. 즉, 과거의 경험(Replay Buffer에 저장된 오래된 데이터)을 재사용해 학습할 수 있다는 뜻이며, 이것이 챕터 11에서 다룰 DQN의 Experience Replay가 가능한 이유입니다. 반면 SARSA 같은 On-policy 알고리즘은 지금 정책으로 직접 만든 데이터로만 학습해야 합니다.

---

## 11. DQN (Deep Q-Network)

### 등장 배경

Q-Learning은 Q-table로 모든 (s,a) 값을 저장합니다. 하지만 카메라 이미지, LiDAR 스캔, 연속적인 관절 각도처럼 상태가 매우 많거나 연속적인 경우 표로 저장하는 것이 불가능합니다.

**해결책**: Q(s,a)를 표가 아니라 **신경망(Neural Network)이 예측**하도록 만듭니다.

```
입력: 상태 (s)
출력: 각 행동에 대한 Q값 (행동이 4개면 출력도 4개)
```

### DQN을 안정적으로 학습시키는 두 가지 장치

**① Experience Replay**

데이터가 시간 순서대로 그대로 들어오면 데이터 간 상관관계가 높아 학습이 불안정합니다. 따라서 경험 `(s, a, r, s')`을 버퍼에 저장해두고, 학습할 때(신경망 업데이트)는 그중에서 **무작위로 샘플링**합니다.

```
(s, a, r, s') 저장 → Replay Buffer → 랜덤 샘플링 → 학습
```

효과: 데이터 다양성 ↑, 학습 안정성 ↑, 과적합 위험 ↓, 오래된 경험 재사용 가능(Off-policy이기 때문)

**② Target Network**

학습 중인 네트워크(Main Network) 하나로 "예측값"과 "정답(타겟)"을 동시에 계산하면, 타겟 자체가 매 스텝 흔들리면서 학습이 발산하기 쉽습니다.

| 네트워크 | 역할 | 업데이트 빈도 |
| --- | --- | --- |
| Main Network | 매 스텝 학습(경사하강법) | 매 스텝 |
| Target Network | 타겟 계산의 기준값 제공 | 일정 주기(예: 10 에피소드)마다 Main의 가중치를 복사 |

```
y = r + γ * max_a' Q(s', a'; θ_old)     # θ_old: Target Network의 파라미터
loss = (Q(s,a;θ) - y)^2                 # θ: Main Network의 파라미터
```

### DQN 전체 흐름

```
1. 상태 s 입력
2. ε-greedy로 행동 선택
3. 환경 실행 → (r, s') 수신
4. (s, a, r, s')를 Replay Buffer에 저장
5. Replay Buffer에서 미니배치 랜덤 샘플링
6. Target Network로 타겟 값 y 계산
7. Main Network 예측값과 y 사이의 Loss 계산
8. Main Network만 경사하강법으로 업데이트
9. 일정 주기마다 Target Network ← Main Network
```

---

## 12. DQN의 한계와 개선: Double DQN, Dueling DQN

DQN에는 두 가지 잘 알려진 한계가 있습니다.

| 문제 | 원인 |
|---|---|
| ① Q값 과대평가 (Overestimation) | `max` 연산이 노이즈가 낀 값 중에서도 "가장 큰 값"을 고르기 때문에, 실제보다 체계적으로 크게 평가되는 편향이 생김 |
| ② V(s)와 A(s,a)를 구분 못함 | "이 상태 자체가 좋은가"와 "이 상태에서 이 행동이 특히 좋은가"를 분리하지 못함 |

### Double DQN (DDQN) — ①을 해결

**행동을 고르는 네트워크**와 **그 행동의 가치를 평가하는 네트워크**를 분리합니다.

```
일반 DQN:    Q(s,a) = r + γ · max_a' Q(s', a'; θ_target)               # 선택과 평가를 같은 네트워크가 함
Double DQN:  Q(s,a) = r + γ · Q(s', argmax_a' Q(s', a'; θ_main); θ_target)  # 선택은 Main, 평가는 Target
```

| 단계 | 담당 네트워크 |
| --- | --- |
| 행동 선택 (argmax) | Main Network |
| 그 행동의 가치 평가 | Target Network |

이렇게 분리하면 한쪽 네트워크의 과대평가 노이즈가 다른 쪽에서 그대로 보정되어, 과대평가가 줄고 학습이 더 안정적입니다.

### Dueling DQN — ②를 해결

Q값을 두 부분으로 분리해서 학습합니다.

```
Q(s,a) = V(s) + A(s,a)
```

| 요소 | 의미 |
| --- | --- |
| V(s) | 상태 자체의 가치 (어떤 행동을 하든 상관없이 이 상태가 얼마나 좋은가) |
| A(s,a) | Advantage: 이 상태에서 이 행동이 평균보다 얼마나 더 좋은가 (상대적 가치) |

> 예: 자율주행에서 "장애물이 없는 직선 도로"라는 상태(State)는 핸들을 어느 쪽으로 틀든 큰 차이가 없으므로 V(s)가 학습의 대부분을 담당합니다. 반면 "갈림길" 상태에서는 어느 방향으로 가는지가 결정적이므로 A(s,a)가 중요해집니다. 이렇게 분리하면 행동 간 차이가 크지 않은 상황에서 더 효율적으로 학습합니다.

---

## 13. Policy Gradient

지금까지(Q-Learning, DQN)는 **Value 기반(Value-based)** 방법이었습니다 — Q값을 먼저 잘 추정하고, 거기서 행동을 간접적으로 고릅니다(`argmax`).

**Policy Gradient**는 접근 자체가 다릅니다. Policy 함수 자체를 신경망으로 직접 표현하고, 기대 보상을 최대화하는 방향으로 **직접** 학습합니다.

```
목표: max_θ E[R]    (θ: Policy Network의 파라미터)
```

| 구분 | Value 기반 (DQN) | Policy 기반 (Policy Gradient) |
| --- | --- | --- |
| 네트워크 출력 | 각 행동의 Q값 | 각 행동을 선택할 확률 |
| 행동 결정 방식 | Q값이 가장 큰 행동 선택 (간접적) | 확률 분포에서 직접 샘플링 (직접적) |
| 연속 행동(Continuous Action) | 다루기 어려움 | 자연스럽게 다룰 수 있음 |
| 대표 활용 | 게임(이산 행동) | 로봇 제어(연속 행동), 자율주행 |

**왜 연속 행동에 유리한가**: DQN은 모든 행동에 대해 Q값을 일일이 계산해서 그중 최댓값을 골라야 합니다. 로봇 관절 토크처럼 행동이 연속적인 실수값이면 "모든 행동"을 나열할 수 없어 `argmax` 자체가 어렵습니다. 반면 Policy Gradient는 행동의 확률 분포(예: 정규분포의 평균과 분산)를 직접 출력하므로 연속 행동에도 자연스럽게 적용됩니다.

**한계**: Policy Gradient는 분산(Variance)이 매우 커서 학습이 불안정한 경우가 많습니다. 이를 보완하기 위해 챕터 14의 Actor-Critic 구조가 등장합니다.

---

## 14. Actor-Critic

Actor-Critic은 **Value 기반**과 **Policy 기반**의 장점을 결합한 구조입니다.

| 역할 | 담당 | 설명 |
| --- | --- | --- |
| Actor | Policy Network | 현재 상태에서 어떤 행동을 할지 결정 |
| Critic | Value Network | Actor가 한 행동이 얼마나 좋았는지 평가 |

**학습 흐름**

```
1. Actor가 상태 s를 보고 행동 a를 선택
2. 환경으로부터 보상 r, 다음 상태 s' 수신
3. Critic이 "이 행동이 평균보다 얼마나 좋았는지"(Advantage)를 평가
4. Critic의 평가를 바탕으로 Actor를 업데이트 (좋았으면 그 행동의 확률을 높이는 방향)
5. Critic 자신도 더 정확한 평가를 하도록 함께 업데이트
```

**Advantage**: 순수 Policy Gradient는 "이 행동으로 받은 보상 그 자체"를 기준으로 학습해서 분산이 큽니다. Actor-Critic은 "평균보다 얼마나 더 좋았는가"라는 상대적 기준(Advantage)을 사용해 분산을 줄입니다.

```
A(s,a) = Q(s,a) - V(s)
```

> 직관: 시험 점수 80점이 "좋은 점수인지"는 그 자체로는 알 수 없습니다. 평균이 90점이면 80점은 나쁜 편이고, 평균이 60점이면 80점은 아주 좋은 편입니다. Advantage는 바로 이 "평균(V(s)) 대비 상대 평가"를 수치화한 것입니다.

Actor-Critic은 그 자체로도 사용되지만, 더 중요하게는 챕터 15의 PPO를 비롯한 현대 강화학습 알고리즘 대부분의 **기본 골격**이 됩니다.

---

## 15. PPO (Proximal Policy Optimization)

### 등장 배경

기존 Policy Gradient(및 단순 Actor-Critic)에는 치명적인 문제가 있습니다: 한 번 업데이트할 때 정책이 **너무 크게 바뀌면** 성능이 갑자기 무너지고 다시는 회복하지 못하는 경우가 많습니다 (정책 붕괴).

PPO의 핵심 아이디어는 단순합니다.

> **"좋은 방향이면 조금만 바꾸고, 너무 많이 바뀌려고 하면 강제로 잘라버린다(clip)."**

### 핵심 수식

```
r(θ) = π_new(a|s) / π_old(a|s)                         # 정책이 얼마나 변했는지 비율
L_clip(θ) = E[ min( r(θ)·A , clip(r(θ), 1-ε, 1+ε)·A ) ]
```

| 요소 | 의미 |
| --- | --- |
| r(θ) | 새 정책과 옛 정책의 행동 선택 확률 비율 |
| A | Advantage (이 행동이 얼마나 좋았는가) |
| clip(r(θ), 1−ε, 1+ε) | r(θ)를 [1−ε, 1+ε] 범위로 강제 제한 (보통 ε=0.2) |
| min(...) | 두 값 중 더 "보수적인"(작은) 쪽을 선택해 과도한 업데이트를 억제 |

### 구성 요소

```
1. Actor (Policy Network)
2. Critic (Value Network)
3. 일정 구간(Trajectory) 동안 수집한 경험 데이터
4. Advantage 계산
5. Clip 기반 손실 함수로 여러 번 미니배치 업데이트
```

| 특징 | 설명 |
| --- | --- |
| 안정성 | 매우 높음 (현재까지 나온 on-policy 알고리즘 중 최상위권) |
| 구현 난이도 | Actor-Critic 대비 약간 더 복잡하지만 비교적 쉬움 |
| 적용 범위 | 이산 행동 + 연속 행동 모두 가능 |
| 실무 활용도 | 로봇 제어, 자율주행, 게임 AI, RLHF(LLM 정렬) 등 가장 널리 사용됨 |

PPO는 "이론적으로 가장 정교한 알고리즘"은 아니지만, **안정성과 구현 단순성의 균형**이 매우 좋아 산업 현장에서 사실상 표준처럼 쓰이는 알고리즘입니다.

---

## 16. 실습: PyTorch로 DQN 구현 (CartPole)

### 환경 준비

```bash
pip install gymnasium torch numpy
```

### 전체 구조

```
1. Environment (Gymnasium)
2. Q-Network (MLP)
3. Replay Buffer
4. Target Network
5. 학습 루프
```

### 코드

```python
import random
import numpy as np
import torch
import torch.nn as nn
import gymnasium as gym
from collections import deque

# 1. Q-Network 정의
class QNetwork(nn.Module):
    def __init__(self, state_dim, action_dim):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(state_dim, 128),
            nn.ReLU(),
            nn.Linear(128, 128),
            nn.ReLU(),
            nn.Linear(128, action_dim)
        )

    def forward(self, x):
        return self.net(x)

# 2. Replay Buffer 정의
class ReplayBuffer:
    def __init__(self, capacity=10000):
        self.buffer = deque(maxlen=capacity)

    def push(self, transition):
        self.buffer.append(transition)

    def sample(self, batch_size):
        batch = random.sample(self.buffer, batch_size)
        s, a, r, s_, done = zip(*batch)
        return (
            torch.tensor(np.array(s), dtype=torch.float32),
            torch.tensor(a, dtype=torch.int64),
            torch.tensor(r, dtype=torch.float32),
            torch.tensor(np.array(s_), dtype=torch.float32),
            torch.tensor(done, dtype=torch.float32),
        )

    def __len__(self):
        return len(self.buffer)

# 3. 한 스텝 학습 함수
def train_step(model, target_model, optimizer, batch, gamma=0.99):
    states, actions, rewards, next_states, dones = batch

    q_values = model(states).gather(1, actions.unsqueeze(1)).squeeze(1)

    with torch.no_grad():
        next_q = target_model(next_states).max(1)[0]
        target = rewards + gamma * next_q * (1 - dones)

    loss = nn.functional.mse_loss(q_values, target)

    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
    return loss.item()

# 4. 메인 학습 루프
def main():
    env = gym.make("CartPole-v1")
    state_dim = env.observation_space.shape[0]
    action_dim = env.action_space.n

    model = QNetwork(state_dim, action_dim)
    target_model = QNetwork(state_dim, action_dim)
    target_model.load_state_dict(model.state_dict())

    optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
    buffer = ReplayBuffer()

    epsilon = 1.0
    epsilon_min = 0.05
    batch_size = 64

    for episode in range(500):
        state, _ = env.reset()
        done = False
        total_reward = 0

        while not done:
            # ε-greedy 행동 선택
            if random.random() < epsilon:
                action = env.action_space.sample()
            else:
                with torch.no_grad():
                    q_values = model(torch.tensor(state, dtype=torch.float32))
                    action = torch.argmax(q_values).item()

            next_state, reward, terminated, truncated, _ = env.step(action)
            done = terminated or truncated

            buffer.push((state, action, reward, next_state, float(done)))
            state = next_state
            total_reward += reward

            if len(buffer) > batch_size:
                batch = buffer.sample(batch_size)
                train_step(model, target_model, optimizer, batch)

        # 타겟 네트워크 주기적 업데이트
        if episode % 10 == 0:
            target_model.load_state_dict(model.state_dict())

        epsilon = max(epsilon_min, epsilon * 0.995)

        if episode % 20 == 0:
            print(f"Episode {episode}, Reward: {total_reward}, Epsilon: {epsilon:.3f}")

    env.close()

if __name__ == "__main__":
    main()
```

**코드 포인트 정리**

| 코드 부분 | 대응 개념 |
|---|---|
| `epsilon` 으로 분기 | 챕터 9: ε-greedy |
| `buffer.push(...)`, `buffer.sample(...)` | 챕터 11: Experience Replay |
| `target_model(next_states).max(1)[0]` | 챕터 11: Target Network 기반 타겟 계산 |
| `target_model.load_state_dict(...)` | 챕터 11: 주기적 Target Network 동기화 |
| `mse_loss(q_values, target)` | 챕터 8: TD Error를 줄이는 학습 |

---

## 17. 실습: PyTorch로 PPO 구현

### 구조

```
1. Actor-Critic 네트워크 (공유 backbone)
2. 일정 길이의 trajectory 수집
3. Advantage(Return - Value) 계산
4. Clip 기반 손실로 여러 epoch 업데이트
```

### 코드

```python
import torch
import torch.nn as nn
import gymnasium as gym

# 1. Actor-Critic 네트워크
class ActorCritic(nn.Module):
    def __init__(self, state_dim, action_dim):
        super().__init__()
        self.shared = nn.Sequential(
            nn.Linear(state_dim, 128),
            nn.ReLU()
        )
        self.actor = nn.Sequential(
            nn.Linear(128, action_dim),
            nn.Softmax(dim=-1)
        )
        self.critic = nn.Linear(128, 1)

    def forward(self, x):
        x = self.shared(x)
        return self.actor(x), self.critic(x)

# 2. 행동 선택
def select_action(model, state):
    state = torch.tensor(state, dtype=torch.float32)
    probs, value = model(state)
    dist = torch.distributions.Categorical(probs)
    action = dist.sample()
    return action.item(), dist.log_prob(action).item(), value.item()

# 3. Advantage 및 Return 계산 (Monte Carlo 방식 — 챕터 7 참고)
def compute_returns_and_advantages(rewards, values, gamma=0.99):
    returns = []
    G = 0
    for r in reversed(rewards):
        G = r + gamma * G
        returns.insert(0, G)

    returns = torch.tensor(returns, dtype=torch.float32)
    values = torch.tensor(values, dtype=torch.float32)
    advantages = returns - values
    return returns, advantages

# 4. PPO 업데이트
def ppo_update(model, optimizer, states, actions, old_log_probs,
                returns, advantages, epsilon=0.2, epochs=4):

    for _ in range(epochs):
        probs, values = model(states)
        dist = torch.distributions.Categorical(probs)
        new_log_probs = dist.log_prob(actions)

        ratio = torch.exp(new_log_probs - old_log_probs)

        surr1 = ratio * advantages
        surr2 = torch.clamp(ratio, 1 - epsilon, 1 + epsilon) * advantages

        actor_loss = -torch.min(surr1, surr2).mean()
        critic_loss = nn.functional.mse_loss(values.squeeze(), returns)

        loss = actor_loss + 0.5 * critic_loss

        optimizer.zero_grad()
        loss.backward()
        optimizer.step()

# 5. 메인 학습 루프
def main():
    env = gym.make("CartPole-v1")
    state_dim = env.observation_space.shape[0]
    action_dim = env.action_space.n

    model = ActorCritic(state_dim, action_dim)
    optimizer = torch.optim.Adam(model.parameters(), lr=0.0003)

    for episode in range(500):
        state, _ = env.reset()
        states, actions, rewards, old_log_probs, values = [], [], [], [], []
        done = False
        total_reward = 0

        while not done:
            action, log_prob, value = select_action(model, state)
            next_state, reward, terminated, truncated, _ = env.step(action)
            done = terminated or truncated

            states.append(state)
            actions.append(action)
            rewards.append(reward)
            old_log_probs.append(log_prob)
            values.append(value)

            state = next_state
            total_reward += reward

        returns, advantages = compute_returns_and_advantages(rewards, values)

        ppo_update(
            model, optimizer,
            torch.tensor(states, dtype=torch.float32),
            torch.tensor(actions, dtype=torch.int64),
            torch.tensor(old_log_probs, dtype=torch.float32),
            returns, advantages
        )

        if episode % 20 == 0:
            print(f"Episode {episode}, Reward: {total_reward}")

    env.close()

if __name__ == "__main__":
    main()
```

**DQN 실습과의 핵심 차이**

| 비교 항목 | DQN (16번) | PPO (17번) |
|---|---|---|
| 네트워크 출력 | Q값 | 행동 확률 + 상태 가치 |
| 데이터 사용 방식 | Replay Buffer에서 재사용 (Off-policy) | 매 episode마다 새로 수집해 사용 후 버림 (On-policy) |
| 업데이트 안정화 장치 | Target Network | Clip(ratio) |

---

## 18. 알고리즘 총정리 비교표

| 알고리즘 | 분류 | 행동 공간 | On/Off-policy | 안정성 | 대표 활용처 |
|---|---|---|---|---|---|
| Q-Learning | Value 기반 | 이산 (소규모) | Off-policy | 낮음 (상태 많아지면 불가) | 작은 격자 환경, 교육용 예제 |
| SARSA | Value 기반 | 이산 | On-policy | 중간 | 안전이 중요한 단순 환경 |
| DQN | Value 기반 | 이산 | Off-policy | 중간 | Atari 게임 |
| Double / Dueling DQN | Value 기반 | 이산 | Off-policy | 중상 | DQN 성능 개선이 필요한 모든 곳 |
| Policy Gradient | Policy 기반 | 이산 + 연속 | On-policy | 낮음 (분산 큼) | 이론적 토대 |
| Actor-Critic | Policy + Value | 이산 + 연속 | On-policy | 중간 | PPO 등의 기반 구조 |
| PPO | Policy + Value | 이산 + 연속 | On-policy | 매우 높음 | 로봇 제어, 자율주행, RLHF |
| (참고) SAC, TD3 | Policy + Value | 연속 | Off-policy | 높음 | 연속 제어 특화 (챕터 20 참고) |

**선택 가이드**
- 행동이 이산적이고 환경이 비교적 단순하다 → DQN 계열로 시작
- 행동이 연속적이다(로봇 토크, 자율주행 조향각 등) → PPO 또는 SAC/TD3
- 안정성과 구현 편의성이 최우선이다 → PPO
- 데이터(시뮬레이션 스텝)를 최대한 적게 쓰고 싶다(Sample Efficiency 중요) → Off-policy 계열(DQN, SAC, TD3)이 Replay Buffer 재사용으로 유리

---

## 19. 로봇 제어에 강화학습 적용하기

로봇 강화학습은 일반 게임 RL과 몇 가지 중요한 차이가 있습니다.

| 차이점 | 게임 RL | 로봇 RL |
|---|---|---|
| 행동 공간 | 대부분 이산적 (상/하/좌/우) | 대부분 연속적 (관절 토크, 속도) |
| 환경 리셋 비용 | 거의 0 (즉시 재시작) | 실제 로봇은 리셋 비용이 큼 → 시뮬레이션 학습 후 전이 |
| 안전 제약 | 없음 | 충돌, 관절 한계 등 물리적 안전 제약 존재 |
| 관측(State) | 게임 내부 변수 | LiDAR, 카메라, IMU, 관절 인코더 등 센서 융합 |

### 일반적인 로봇 RL 파이프라인

```
1. 시뮬레이터에서 학습 (MuJoCo / Gazebo / Isaac Sim 등)
2. PPO 또는 SAC로 Policy 학습
3. Domain Randomization (마찰, 질량, 센서 노이즈 등을 무작위화하여 일반화 성능 확보)
4. Sim-to-Real Transfer: 학습된 Policy를 실제 로봇에 배포
5. 실제 환경에서 미세 조정(Fine-tuning) 또는 안전 모니터링과 함께 운용
```

| 시뮬레이터 | 특징 | 적합한 용도 |
|---|---|---|
| MuJoCo | 빠른 물리 연산, 정밀한 접촉(contact) 모델링 | 매니퓰레이션, 보행 로봇 |
| Gazebo (Classic) | ROS2와 긴밀한 통합, 센서 플러그인 풍부 | 모바일 로봇, SLAM과 결합한 RL |
| Isaac Sim / Isaac Lab | GPU 병렬 시뮬레이션 (수천 개 환경 동시 실행) | 대규모 PPO 학습, 휴머노이드 |

### ROS2와의 연결 지점

실제 로봇에 RL Policy를 배포할 때는 보통 학습된 신경망을 ONNX/TensorRT로 변환한 뒤, ROS2 노드 안에서 다음과 같은 흐름으로 실행합니다.

```
[센서 토픽 구독: /scan, /odom, /camera] 
        ↓
   State 전처리 (정규화 등)
        ↓
   Policy Network 추론 (ONNX Runtime / TensorRT)
        ↓
   Action 후처리 (범위 클리핑, 안전 필터)
        ↓
[제어 명령 퍼블리시: /cmd_vel, /joint_trajectory]
```

> 핵심은 **학습은 시뮬레이션에서, 추론은 실제 ROS2 노드에서** 일어난다는 점입니다. 학습용 PyTorch 모델과 배포용 추론 엔진(ONNX/TensorRT)을 분리해서 관리하는 것이 실무에서 일반적인 패턴입니다.

---