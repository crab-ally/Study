# 딥러닝

## 1. 딥러닝이란 무엇인가 — AI / ML / DL의 관계

```
AI (인공지능)                    "사람처럼 똑똑한 행동을 하는 모든 시스템"
 └─ Machine Learning (머신러닝)  "규칙을 직접 안 짜고 데이터로 학습"
     └─ Deep Learning (딥러닝)   "신경망 구조로, 특징(feature)도 데이터가 직접 학습"
```

### 1-1. 시대적 변화

| 시대 | 방식 | 예시 (장애물 인식) | 한계 |
|---|---|---|---|
| 전통 프로그래밍 | 사람이 규칙을 직접 작성 | "거리 < 30cm면 정지" | 규칙이 많아지면 못 버팀 (조명, 각도, 재질…) |
| 전통 머신러닝 | 사람이 특징을 뽑고, 모델이 분류만 학습 | "엣지 강도, 색상 히스토그램을 계산 후 SVM" | 좋은 feature를 사람이 설계해야 함 |
| **딥러닝** | **데이터에서 특징도 자동 추출 + 분류까지 한 번에 학습** | 카메라 이미지를 그대로 넣으면 끝 | 데이터와 GPU 연산량이 많이 필요 |

> **핵심:** 딥러닝은 "어떤 특징을 봐야 하는지"조차 사람이 정하지 않습니다. 모델이 데이터를 보면서 스스로 알아냅니다.

### 1-2. 왜 지금 딥러닝인가

- **데이터량 증가**: 센서, 카메라, 인터넷 데이터가 폭발적으로 늘어남
- **GPU 연산력**: 행렬 곱 연산을 병렬로 처리 가능해짐
- **알고리즘 발전**: ReLU, BatchNorm, Adam, Attention 등 학습을 안정화하는 기법들이 쌓임

---

## 2. 신경망의 최소 단위 — 퍼셉트론

### 2-1. 생물학적 비유

뉴런은 여러 입력 신호를 받아 합산하고, 일정 임계치를 넘으면 다음 뉴런으로 신호를 전달합니다. 퍼셉트론은 이걸 수식으로 흉내 낸 것입니다.

```
입력 신호들 → 가중합(weighted sum) → 활성화 함수 → 출력
```

> 로봇 비유: **초음파 센서 여러 개의 거리값을 가중치를 곱해 합치고, 그 합이 임계치를 넘으면 "정지" 신호를 출력**

### 2-2. 수식

```
y = f(Wx + b)
```

| 기호 | 의미 | 로봇 예시 |
|---|---|---|
| x | 입력값 | 센서 측정값 (거리, 밝기 등) |
| W | 가중치 | 각 센서를 얼마나 신뢰할지 |
| b | 편향 | 기본적으로 더하는 보정값 |
| f | 활성화 함수 | 신호를 0~1 같은 범위로 변환 |
| y | 출력값 | 최종 판단 (정지 / 진행) |

### 2-3. 퍼셉트론의 한계 — 직선 하나

퍼셉트론 1개는 데이터를 **직선(또는 평면) 하나**로만 나눕니다.

```
   o o                ← 안전 구역
 o   o
───────────  ← 퍼셉트론이 그은 경계선
 x   x
   x x                ← 충돌 위험 구역
```

문제는, 현실 데이터는 직선 하나로 안 나뉘는 경우가 훨씬 많다는 것입니다 (XOR 문제가 대표적, 4절에서 다룹니다). 그래서 여러 퍼셉트론을 층층이 쌓습니다 — 이게 바로 "딥(deep)"러닝의 시작입니다.

---

## 3. 활성화 함수 — 비선형성을 만드는 장치

활성화 함수가 없으면, 층을 100개를 쌓아도 수학적으로는 **선형 변환 1개와 동일**합니다 (선형 변환의 합성은 결국 선형 변환이기 때문). 활성화 함수가 **비선형성**을 끼워 넣어야 복잡한 패턴(곡선 형태의 경계, 다중 조건 판단 등)을 학습할 수 있습니다.

### 3-1. 주요 활성화 함수

#### Sigmoid

```
σ(x) = 1 / (1 + e^(-x))
```
- 출력 범위: **0 ~ 1** → 확률처럼 해석 가능
- 주로 **이진 분류 출력층**에 사용 (예: "이 물체가 사람인가 아닌가")
- 단점: 입력이 매우 크거나 작으면 기울기가 0에 가까워짐 → **Vanishing Gradient**

#### Tanh

```
tanh(x) = (e^x - e^(-x)) / (e^x + e^(-x))
```
- 출력 범위: **-1 ~ 1**, 중심이 0이라 Sigmoid보다 학습이 약간 안정적

#### ReLU (Rectified Linear Unit) — 은닉층 표준

```
ReLU(x) = max(0, x)
```
- 계산이 단순하고 빠름 → 깊은 신경망에서 특히 효과적
- 현재 은닉층의 기본 선택지

> 라이다 데이터를 예로 들면: 음수 거리값은 의미가 없으니 0으로 끊어버리고, 양수는 그대로 살리는 ReLU의 동작이 직관적으로 잘 맞아떨어집니다.

### 3-2. 어디에 어떤 함수를 쓰는가

```
은닉층:  ReLU (기본 선택)
출력층:  이진 분류  → Sigmoid
        다중 분류  → Softmax
        회귀(연속값) → 활성화 없음 (선형 그대로 출력)
```

---

## 4. 다층 신경망(MLP) — 층을 쌓는 이유

### 4-1. 구조

```
입력층 (Input Layer)     ← 데이터를 받는 곳
      ↓
은닉층 1 (Hidden Layer)   ← 1차 특징 추출
      ↓
은닉층 2 (Hidden Layer)   ← 더 복잡한 특징 추출
      ↓
출력층 (Output Layer)    ← 최종 예측
```

> **딥러닝 = 은닉층이 여러 개 있는 신경망**

### 4-2. 층이 깊어질수록 더 추상적인 특징을 본다

카메라로 사람을 인식하는 모바일 로봇을 예로 들면:

```
픽셀 (raw 이미지)
   ↓ 1번째 은닉층
선, 모서리, 색 경계 감지
   ↓ 2번째 은닉층
팔, 다리, 몸통 윤곽 인식
   ↓ 3번째 은닉층
사람 전체 형태 학습
   ↓ 출력층
"사람이다 / 아니다" 판단 (확률)
```

### 4-3. XOR 문제 — 퍼셉트론 1개로는 불가능한 이유

| x1 | x2 | XOR |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

이 데이터는 직선 하나로 나눌 수 없습니다 (두 영역이 대각선으로 교차). 은닉층을 1개만 추가해도 곡선/꺾인 경계를 만들 수 있어서 풀립니다. 9장의 실습 코드에서 직접 확인합니다.

### 4-4. 깊이와 성능의 트레이드오프

| 층 수 | 결과 |
|---|---|
| 너무 적음 | 표현력 부족 → 단순 패턴만 학습 (**Underfitting**) |
| 적당함 | 좋은 일반화 성능 ✓ |
| 너무 많음 (데이터 적은데) | 훈련 데이터에만 맞춰짐 (**Overfitting**) |

---

## 5. 학습 사이클 — Forward / Loss / Backward / Update

딥러닝 학습은 아래 4단계를 데이터가 끝날 때까지 반복합니다.

```
① Forward Pass     입력 → 신경망 통과 → 예측값 생성
        ↓
② Loss 계산        예측값 vs 정답 비교 → 오차를 숫자로 표현
        ↓
③ Backward Pass    오차의 원인을 역방향으로 추적 (역전파)
        ↓
④ 가중치 업데이트   gradient를 이용해 W, b를 조금씩 수정
        ↓
    (① 로 돌아가 반복)
```

### 5-1. Epoch / Batch / Iteration

| 용어 | 의미 |
|---|---|
| **Epoch** | 전체 데이터를 한 번 모두 학습 |
| **Batch Size** | 한 번에 처리하는 데이터 개수 |
| **Iteration** | 1 Epoch을 끝내는 데 필요한 업데이트 횟수 |

> 예: 라이다 스캔 데이터 1,000개, Batch Size 100 → 1 Epoch = 10 Iteration

### 5-2. 왜 한 번에 전체 데이터로 안 학습할까

전체 데이터를 한 번에 넣으면(Batch Size = 전체) 메모리가 부족해지고, 업데이트가 너무 가끔 일어나 학습이 느립니다. 그래서 적당한 크기로 잘라서(Mini-batch) 자주 업데이트합니다.

---

## 6. 손실 함수 — 무엇을 줄여야 하는가

손실 함수(Loss Function)는 **모델의 예측이 얼마나 틀렸는지**를 숫자로 나타냅니다. 학습의 목표는 이 값을 최소화하는 것입니다.

### 6-1. 문제 유형별 손실 함수

#### 분류 (Classification)

**BCELoss (Binary Cross Entropy Loss)**
- 이진 분류 (예: "장애물이다 / 아니다")
- 출력층에 Sigmoid 사용 → ŷ 출력

```
BCE = -[y·log(ŷ) + (1-y)·log(1-ŷ)]
```

**CrossEntropyLoss**
- 다중 분류 (예: "보행자 / 차량 / 자전거 / 배경")
- 출력층에 Softmax가 내장되어 있음

#### 회귀 (Regression)

**Mean Squared Error — MSELoss**
- 연속값 예측 (예: 다음 시점의 로봇 속도, 거리)

```
MSE = (1/n) · Σ(y - ŷ)²
```

### 6-2. 선택 가이드

```
문제 유형 판단
   │
   ├── 숫자 예측 (회귀)        → MSELoss
   │
   └── 분류
         ├── 2가지 중 하나     → BCELoss
         └── 3가지 이상 중 하나 → CrossEntropyLoss
```

---

## 7. 역전파와 옵티마이저

### 7-1. 역전파(Backpropagation)

역전파는 **오차의 책임을 각 가중치에 얼마나 돌릴지** 계산하는 방법입니다. 핵심은 미분(Gradient)입니다.

```
∂Loss / ∂W   →   "W를 살짝 바꾸면 Loss가 얼마나 변하는가?"
```

- gradient가 크다 → 이 가중치가 오차에 많이 기여 → 많이 수정
- gradient가 작다 → 기여가 적음 → 조금만 수정

```
forward(예측) → loss 계산 → backward(gradient 계산) → weight 업데이트
```

### 7-2. 옵티마이저(Optimizer)

계산된 gradient로 실제 가중치를 갱신하는 알고리즘입니다.

```
W = W - lr × gradient
```

`lr`은 **학습률(Learning Rate)** — 한 번에 얼마나 크게 움직일지 결정합니다.

#### SGD (Stochastic Gradient Descent)

```python
optimizer = optim.SGD(model.parameters(), lr=0.1)
```
- 구현이 단순하고 메모리 효율적
- 수렴이 느리고 Local Minima(국소 최저점)에 갇히기 쉬움

#### Adam (Adaptive Moment Estimation)

```python
optimizer = optim.Adam(model.parameters(), lr=0.001)
```
- 파라미터마다 학습률을 자동으로 조절
- **실무 기본값** — 빠른 수렴, Local Minima 탈출 능력 좋음

#### 학습률의 중요성

```
lr 너무 크면 :  Loss가 발산 (왔다갔다 튐)
lr 너무 작으면:  학습이 너무 느림
lr 적당하면  :  안정적으로 최솟값에 수렴  ✓
```

> 비유: 산을 내려가는데, lr이 너무 크면 발걸음이 너무 커서 계곡을 훌쩍 뛰어넘어 버리고, 너무 작으면 한 걸음씩 기어가서 해가 질 때까지 못 내려갑니다.

---

## 8. 과적합과 일반화

### 8-1. 과적합이란?

훈련 데이터는 완벽히 맞추지만, **새로운 데이터에서는 틀리는 현상**입니다.

```
훈련 데이터 Loss: 0.01  (매우 낮음)
테스트 데이터 Loss: 0.95  (매우 높음)  ← 과적합!
```

로봇 관점에서는 위험합니다 — 학습 환경(실험실 조명, 특정 바닥재)에서는 잘 동작하지만, 실제 현장(다른 조명, 다른 바닥)에서 갑자기 인식이 무너지는 경우가 전형적인 과적합 사례입니다.

### 8-2. 데이터 분할

```
전체 데이터
   │
   ├── 훈련 데이터 (Train)      70~80%  ← 학습에 사용
   ├── 검증 데이터 (Validation)  10~15%  ← 과적합 감지용
   └── 테스트 데이터 (Test)     10~15%  ← 최종 성능 평가
```

### 8-3. 해결 방법

#### ① Dropout

학습 중 무작위로 일부 뉴런을 꺼버려, 특정 뉴런에만 의존하지 못하게 합니다.

```python
nn.Dropout(p=0.3)  # 30% 뉴런을 랜덤하게 비활성화
```
> 주의: Dropout은 **학습 시에만** 동작하고, 추론(`model.eval()`) 시에는 자동으로 꺼집니다.

#### ② Batch Normalization

각 층의 출력을 정규화해서 학습을 안정화시킵니다. 학습 속도 향상 + 과적합 완화 효과가 있습니다.

```python
nn.BatchNorm1d(num_features)
```

#### ③ Early Stopping

검증 데이터 성능이 더 이상 좋아지지 않으면 학습을 중단합니다.

```
Epoch 1:   Val Loss 0.8  ↓
Epoch 10:  Val Loss 0.3  ↓
Epoch 20:  Val Loss 0.2  ↓
Epoch 30:  Val Loss 0.25 ↑  ← 여기서 중단! (과적합 시작 신호)
```

#### ④ 데이터 증강 (Data Augmentation) — 참고

이미지를 회전·반전·밝기 변화시켜 데이터를 늘리는 방법도 과적합 완화에 효과적입니다. 로봇 카메라 데이터처럼 조명·각도 변화가 큰 환경에서는 특히 중요합니다.

#### ⑤ 권장 층 구성 순서

```
Linear → BatchNorm → ReLU → Dropout
```

> Linear: 입력 데이터를 다음 층이 처리할 수 있는 형태로 변환하기 위해 사용 `y = Wx + b`

---

## 9. PyTorch 실습 ① — 퍼셉트론부터 과적합 방지 MLP까지

### 9-1. 환경 설정

```bash
pip install torch torchvision
```

### 9-2. 퍼셉트론 구현 — 장애물 감지 AND 로직

두 센서가 **모두** 장애물을 감지했을 때만 "정지"로 판단하는 단순 로직을 퍼셉트론으로 학습합니다 (AND 게이트와 동일한 구조).

```python
import torch
import torch.nn as nn
import torch.optim as optim

# 입력: [센서1 감지여부, 센서2 감지여부]
X = torch.tensor([
    [0., 0.],
    [0., 1.],
    [1., 0.],
    [1., 1.]
])

# 정답: 둘 다 감지해야 "정지"(1)
y = torch.tensor([
    [0.],  # 둘 다 미감지 → 진행
    [0.],  # 센서2만 감지 → 진행
    [0.],  # 센서1만 감지 → 진행
    [1.]   # 둘 다 감지 → 정지
])

class Perceptron(nn.Module):
    def __init__(self):
        super().__init__()
        self.linear = nn.Linear(2, 1)   # 입력 2개 → 출력 1개

    def forward(self, x):
        return torch.sigmoid(self.linear(x))  # 0~1 확률 출력

model = Perceptron()
criterion = nn.BCELoss()
optimizer = optim.SGD(model.parameters(), lr=0.1)

for epoch in range(1000):
    output = model(X)            # ① Forward
    loss = criterion(output, y)  # ② Loss

    optimizer.zero_grad()        # gradient 초기화 (누적 방지)
    loss.backward()              # ③ Backward
    optimizer.step()             # ④ 가중치 업데이트

    if epoch % 100 == 0:
        print(f"Epoch {epoch:4d} | Loss: {loss.item():.4f}")

with torch.no_grad():  # 추론 시 gradient 계산 불필요
    predictions = model(X)
    print("\n예측 결과:")
    for inputs, pred in zip(X, predictions):
        print(f"  센서 상태 {inputs.tolist()} → 정지 확률 {pred.item():.4f}")
```

### 9-3. MLP 구현 — XOR (퍼셉트론 1개로 안 풀리는 문제)

```python
X = torch.tensor([[0.,0.],[0.,1.],[1.,0.],[1.,1.]])
y = torch.tensor([[0.],[1.],[1.],[0.]])  # XOR

class MLP(nn.Module):
    def __init__(self):
        super().__init__()
        self.model = nn.Sequential(
            nn.Linear(2, 4),   # 입력층 → 은닉층1
            nn.ReLU(),
            nn.Linear(4, 4),   # 은닉층1 → 은닉층2
            nn.ReLU(),
            nn.Linear(4, 1),   # 은닉층2 → 출력층
            nn.Sigmoid()
        )

    def forward(self, x):
        return self.model(x)

model = MLP()
criterion = nn.BCELoss()
optimizer = optim.Adam(model.parameters(), lr=0.01)

for epoch in range(3000):
    output = model(X)
    loss = criterion(output, y)

    optimizer.zero_grad()
    loss.backward()
    optimizer.step()

    if epoch % 500 == 0:
        print(f"Epoch {epoch:4d} | Loss: {loss.item():.4f}")
```

### 9-4. 과적합 방지 MLP — 센서 노이즈를 견디는 모델

```python
class ImprovedMLP(nn.Module):
    def __init__(self):
        super().__init__()
        self.model = nn.Sequential(
            nn.Linear(2, 16),
            nn.BatchNorm1d(16),
            nn.ReLU(),
            nn.Dropout(0.3),

            nn.Linear(16, 16),
            nn.BatchNorm1d(16),
            nn.ReLU(),

            nn.Linear(16, 1),
            nn.Sigmoid()
        )

    def forward(self, x):
        return self.model(x)

model = ImprovedMLP()

model.train()   # Dropout, BatchNorm 활성화 (학습 시)
model.eval()    # Dropout 비활성화, BatchNorm은 누적 통계 사용 (추론 시)
```

### 9-5. 학습 루프 표준 패턴 (이후 모든 모델에 동일하게 적용됨)

```python
for epoch in range(num_epochs):

    # --- 학습 단계 ---
    model.train()
    for X_batch, y_batch in train_loader:
        output = model(X_batch)            # Forward
        loss = criterion(output, y_batch)  # Loss
        optimizer.zero_grad()              # Gradient 초기화
        loss.backward()                    # Backward
        optimizer.step()                   # 업데이트

    # --- 검증 단계 ---
    model.eval()
    with torch.no_grad():
        for X_val, y_val in val_loader:
            val_output = model(X_val)
            val_loss = criterion(val_output, y_val)
```

> 이 패턴은 CNN, LSTM, Transformer 모두 동일합니다. **모델 구조만 바뀌고, 학습 루프는 거의 그대로 재사용**됩니다 — 이게 PyTorch가 편한 이유 중 하나입니다.

---

## 10. CNN — 이미지를 위한 신경망

### 10-1. MLP로 이미지를 처리하면 생기는 문제

1. 이미지의 공간적 구조(주변 픽셀과의 관계)를 무시함 — 픽셀을 한 줄로 펴버리기 때문
2. 파라미터 수가 너무 많아짐 (예: 224×224 이미지 → 입력 노드 약 5만 개)
3. 과적합 위험이 큼

CNN은 이미지를 평평하게 펴지 않고, **공간 구조를 유지한 채** 학습합니다.

### 10-2. 핵심 구성 요소

1. **Convolution**: 작은 필터(커널)를 이미지 위에서 슬라이딩하며 특징 추출
2. **Feature Map**: 필터를 통과한 결과 (엣지, 질감, 색 패턴 등)
3. **Pooling**: 크기를 줄이면서 중요한 정보만 유지 (계산량 감소 + 위치 변화에 둔감해짐)

```
입력 이미지
 → Conv → ReLU → Pooling
 → Conv → ReLU → Pooling
 → Flatten
 → FC (Fully Connected)
 → 출력
```

입력 데이터 형식: `(배치, 채널, 높이, 너비)` — 예: 흑백 카메라 28×28 이미지 64장 → `(64, 1, 28, 28)`

| 층의 깊이 | 보는 것 |
|---|---|
| 초기 층 | 엣지, 색 경계 |
| 중간 층 | 패턴, 질감 (바퀴, 모서리 등) |
| 후반 층 | 객체 전체 (사람, 컵, 장애물) |

### 10-3. 코드

```python
import torch.nn as nn

class CNN(nn.Module):
    def __init__(self):
        super().__init__()

        self.conv = nn.Sequential(
            nn.Conv2d(1, 16, kernel_size=3, padding=1),  # 입력채널1 - (3x3 필터 16개) → 출력채널16
            nn.ReLU(),
            nn.MaxPool2d(2),  # 크기 절반으로 (2x2 필터)

            nn.Conv2d(16, 32, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.MaxPool2d(2)
        )

        self.fc = nn.Sequential(
            nn.Flatten(),
            nn.Linear(32 * 7 * 7, 128),
            nn.ReLU(),
            nn.Linear(128, 10) # 10개 클래스에 대한 값 출력
        )

    def forward(self, x):
        x = self.conv(x)
        x = self.fc(x)
        return x
```

> CNN은 카메라가 달린 로봇의 "눈" 역할을 담당하는 가장 기본적인 구조입니다. 객체 탐지(YOLO), 시맨틱 분할(이 픽셀이 바닥인지 장애물인지) 같은 고급 모델들도 결국 CNN을 기반으로 합니다.

---

## 11. RNN과 LSTM — 순서가 있는 데이터

### 11-1. RNN (Recurrent Neural Network)

이전 시점의 정보를 "기억"하면서 다음 입력을 처리합니다. IMU 센서값, 로봇의 시간별 위치, 관절 각도 시계열처럼 **순서가 의미를 가지는 데이터**에 적합합니다.

```
h_t = f(W·x_t + U·h_{t-1})
```

- `x_t`: 현재 시점 입력
- `h_{t-1}`: 이전 시점의 은닉 상태(기억)
- `h_t`: 현재 시점의 은닉 상태(다음으로 넘겨줄 기억)

**문제점**: 시퀀스가 길어지면 앞쪽 정보가 뒤쪽까지 잘 전달되지 않음 (**Vanishing Gradient**) — 즉, "기억력이 짧다."

### 11-2. LSTM (Long Short-Term Memory)

RNN의 기억력 문제를 게이트(gate) 구조로 해결합니다. 중요한 정보만 선택적으로 유지/삭제합니다.

| 게이트 | 역할 |
|---|---|
| Forget Gate | 버릴 정보를 결정 |
| Input Gate | 새로 저장할 정보를 결정 |
| Output Gate | 현재 시점에 출력할 정보를 결정 |

```python
import torch.nn as nn

class LSTMModel(nn.Module):
    def __init__(self):
        super().__init__()
        # 객체 생성
        self.lstm = nn.LSTM(input_size=10, hidden_size=32, batch_first=True)
        self.fc = nn.Linear(32, 1)

    def forward(self, x):
        out, _ = self.lstm(x)
        out = out[:, -1, :]   # 마지막 시점의 출력만 사용
        return self.fc(out)
```

입력 데이터 형식: `(batch, sequence_length, feature)`

```
(32, 10, 5)
→ 32개 샘플
→ 시점 10개 (예: 최근 10프레임의 IMU 측정값)
→ feature 5개 (예: 가속도 x,y,z + 각속도 x,y)
```

> 로봇 예시: 최근 10프레임의 라이다 최소거리값 시퀀스를 보고 "다음 프레임에 충돌이 발생할지"를 예측하는 모델이 바로 이 구조입니다.

---

## 12. Transformer와 Attention — 지금의 표준

### 12-1. RNN/LSTM의 한계

- 순차 처리 → 느림 (이전 시점이 끝나야 다음 시점을 계산)
- 시퀀스가 길어지면 먼 과거 정보를 기억하기 어려움
- GPU의 강점인 병렬 처리를 활용하기 어려움

Transformer는 **시퀀스 전체를 한 번에 보고 병렬로 처리**합니다.

### 12-2. Attention — 중요한 부분에 집중하기

```
"I love deep learning"

→ "love"가 "I"보다 더 핵심적인 의미를 가질 수 있음
→ "deep learning"이 문장 전체 의미에서 더 중요한 비중
```

### 12-3. Self-Attention — 단어들이 서로를 참조

```
"The animal didn't cross the street because it was tired"

→ "it" 이 무엇을 가리키는지(= animal)를 모델이 스스로 계산
```

```
Query (질문: 내가 지금 무엇을 찾고 있는가)
Key   (비교 기준: 각 단어가 가진 특징표)
Value (실제 정보: 가져올 내용)

Attention(Q, K, V) = softmax(QK^T / √d) · V
```

- `QK^T` → 단어들 간 유사도 계산
- `softmax` → 유사도를 중요도(가중치)로 변환
- `V` → 그 중요도로 실제 정보를 가중합

### 12-4. 전체 구조

```
입력
 → Embedding (단어 → 벡터)
 → Positional Encoding (순서 정보 추가, Attention은 순서를 모르기 때문)
 → Self-Attention
 → Feed Forward
 → 출력
```

### 12-5. LSTM vs Transformer

| 항목 | LSTM | Transformer |
|---|---|---|
| 처리 방식 | 순차 | 병렬 |
| 속도 | 느림 | 빠름 |
| 장기 의존성 | 약함 | 강함 |
| 학습 데이터 요구량 | 적음 | 많음 |
| 연산/메모리 비용 | 낮음 | 높음 (시퀀스 길이의 제곱에 비례, O(n^2)) |

### 12-6. 코드 (구조 확인용)

```python
import torch
import torch.nn as nn

model = nn.Transformer(
    d_model=512,
    nhead=8,
    num_encoder_layers=6
)

src = torch.rand(10, 32, 512) # 입력 데이터 (sequence, batch, feature)
tgt = torch.rand(20, 32, 512) # Decoder 입력 데이터 (sequence, batch, feature)

out = model(src, tgt)
print(out.shape)
```
 
> Transformer는 현재 LLM(거대 언어모델)뿐 아니라 비전(ViT), 음성, 심지어 로봇 정책 학습(예: RT-2, 행동 모델)에서도 표준 백본으로 쓰입니다. 다만 연산량이 크기 때문에, **임베디드 로봇에 직접 올릴 때는 경량화가 필수**입니다 (14장에서 다룹니다).

---

## 13. PyTorch 실습 ② — MNIST 분류 / 시계열 예측 / 텍스트 분류

### 13-1. CNN으로 손글씨 숫자 분류 (MNIST)

```
데이터 로드 → 전처리 → 모델 정의 → 학습 → 평가 → 저장
```

```python
from torchvision import datasets, transforms
from torch.utils.data import DataLoader
import torch.nn as nn
import torch.optim as optim
import torch


transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.5,), (0.5,))
])

# 데이터 로드
train_data = datasets.MNIST(root='./data', train=True, transform=transform, download=True)
test_data = datasets.MNIST(root='./data', train=False, transform=transform)

train_loader = DataLoader(train_data, batch_size=64, shuffle=True) # 64개씩 배치 처리
test_loader = DataLoader(test_data, batch_size=64)

# CNN 모델
class CNN(nn.Module):
    def __init__(self):
        super().__init__()
        self.conv = nn.Sequential(
            nn.Conv2d(1, 16, 3, padding=1), nn.ReLU(), nn.MaxPool2d(2),
            nn.Conv2d(16, 32, 3, padding=1), nn.ReLU(), nn.MaxPool2d(2)
        )
        self.fc = nn.Sequential(
            nn.Flatten(),
            nn.Linear(32 * 7 * 7, 128), nn.ReLU(),
            nn.Linear(128, 10)
        )

    def forward(self, x):
        return self.fc(self.conv(x))

# 학습 준비
model = CNN()
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)

# 학습
for epoch in range(5):
    for X, y in train_loader:
        output = model(X)
        loss = criterion(output, y)

        optimizer.zero_grad()
        loss.backward()
        optimizer.step()

    print(f"Epoch {epoch} 완료")

# 평가
correct, total = 0, 0
with torch.no_grad():
    for X, y in test_loader:
        output = model(X)
        _, predicted = torch.max(output, 1) # 최대값, 최대값 위치 반환
        total += y.size(0)
        correct += (predicted == y).sum().item() # item(): Tensor → Python 숫자 변환

print("Accuracy:", correct / total)

# 모델 저장
torch.save(model.state_dict(), "cnn_model.pth")
```

### 13-2. LSTM으로 시계열 예측

```
입력: 과거 데이터 (t-10 ~ t)
출력: 미래 값 (t+1)
```

**Sliding Window** 방식으로 데이터를 구성합니다.

```
[1, 2, 3, 4, 5] → 6
[2, 3, 4, 5, 6] → 7
[3, 4, 5, 6, 7] → 8
```

```
데이터 수집 → 정규화 → window 생성 → LSTM 학습 → 예측
```

```python
import numpy as np
import torch
import torch.nn as nn
import torch.optim as optim

# 예시 데이터 (실제로는 로봇 속도/거리/가속도 시계열 등을 사용)
data = np.sin(np.linspace(0, 100, 1000))
data = (data - np.mean(data)) / np.std(data)  # 정규화

# 슬라이딩 윈도우 (Sliding Window)
def create_dataset(data, seq_len=10):
    X, y = [], []
    for i in range(len(data) - seq_len):
        X.append(data[i:i+seq_len])
        y.append(data[i+seq_len])
    # unsqueeze(): 차원 추가 - (batch, seq_len, feature) 맞추기 위함
    return torch.tensor(X).float().unsqueeze(-1), torch.tensor(y).float()
X, y = create_dataset(data)

# LSTM 모델
class LSTMModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.lstm = nn.LSTM(input_size=1, hidden_size=32, batch_first=True)
        self.fc = nn.Linear(32, 1)

    def forward(self, x):
        out, _ = self.lstm(x)
        out = out[:, -1, :]
        return self.fc(out)

# 학습 준비
model = LSTMModel()
criterion = nn.MSELoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)

# 학습
for epoch in range(20):
    output = model(X)
    loss = criterion(output.squeeze(), y)

    optimizer.zero_grad()
    loss.backward()
    optimizer.step()

    print(epoch, loss.item())

# 예측
with torch.no_grad():
    pred = model(X[:10])
    print(pred)
```

### 13-3. Transformer 기반 텍스트 분류 (사전학습 모델 활용)

```
텍스트 → 토큰화 → Embedding → Transformer → 출력
```

> 토큰화: 문장을 잘게 쪼개 숫자로 변환. `"I love AI"` → `["I", "love", "AI"]`  
> Embedding: 토큰을 벡터로 변환

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification
import torch

# 모델 로드
tokenizer = AutoTokenizer.from_pretrained("distilbert-base-uncased")
model = AutoModelForSequenceClassification.from_pretrained(
    "distilbert-base-uncased", num_labels=2
)

text = ["I love AI", "This is terrible"]

# 데이터 전처리
inputs = tokenizer(text, padding=True, truncation=True, return_tensors="pt")

# 추론
with torch.no_grad():
    outputs = model(**inputs)

pred = torch.argmax(outputs.logits, dim=1)
print(pred)  # 0 or 1

# 미세조정 (Fine-tuning)
optimizer = torch.optim.Adam(model.parameters(), lr=2e-5)

for epoch in range(3):
    outputs = model(**inputs, labels=torch.tensor([1, 0]))
    loss = outputs.loss

    optimizer.zero_grad()
    loss.backward()
    optimizer.step()

    print(loss.item())
```

> 처음부터 Transformer를 직접 학습시키는 경우는 드뭅니다. 대부분 **사전학습된(pretrained) 모델을 가져와 우리 데이터에 맞게 미세조정(Fine-tuning)**하는 방식을 씁니다 — 이게 실무에서 압도적으로 많이 쓰는 패턴입니다.

---

## 14. 로보틱스 관점 — 학습된 모델을 로봇에 올리기

여기까지가 "모델을 학습시키는" 이야기였습니다. 그런데 학습이 끝났다고 끝이 아닙니다 — 로봇은 보통 GPU 서버가 아니라 **임베디드 보드(Jetson, Raspberry Pi 등)** 위에서 실시간으로 모델을 돌려야 합니다.

### 14-1. 학습 환경 vs 배포 환경

| | 학습 (Training) | 배포 (Inference on Robot) |
|---|---|---|
| 위치 | GPU 서버 / 워크스테이션 | Jetson, 임베디드 보드 |
| 목표 | 정확도 최대화 | 정확도 + **실시간성(latency)** + 전력 |
| 데이터 | 대량, 오프라인 | 센서에서 실시간 1장씩 |

### 14-2. 경량화 기법 (참고용 개요)

- **Quantization**: 가중치를 32bit float → 8bit int로 줄여 속도/메모리 절약
- **Pruning**: 영향이 적은 가중치/뉴런을 제거
- **Knowledge Distillation**: 작은 모델이 큰 모델의 출력을 모방하도록 학습
- **ONNX / TensorRT**: PyTorch 모델을 변환해 추론 속도를 높이는 표준 파이프라인

```
PyTorch 모델 (.pth)
   → ONNX 변환 (.onnx)
   → TensorRT 최적화 (Jetson 등에서 가속)
   → 로봇 런타임(ROS2 노드 등)에 탑재
```

### 14-3. ROS2와의 연결 지점

학습된 CNN(객체 인식), LSTM(충돌 예측) 모델은 보통 ROS2 노드 형태로 감싸서, 카메라 토픽을 구독(subscribe)하고 결과를 토픽으로 발행(publish)하는 구조로 들어갑니다.

```
/camera/image_raw  --(subscribe)-->  [추론 노드: CNN 모델]  --(publish)-->  /detection/result
```

> 지금까지 학습하신 MuJoCo, Gazebo, OpenCV, ROS2 통신(Topic/Service/Action) 흐름과 딥러닝 모델이 만나는 지점이 바로 여기입니다 — 시뮬레이션에서 데이터를 모으고 학습한 뒤, 동일한 ROS2 파이프라인에 모델을 끼워 넣는 구조입니다.

---

## 15. 정리 및 다음 학습 로드맵

### 15-1. 전체 흐름 한눈에 보기

```
데이터
  ↓
신경망 (퍼셉트론들의 집합)
  ↓ 활성화 함수로 비선형성 추가
특징 자동 추출 (층이 깊을수록 더 추상적인 특징)
  ↓ 순전파(Forward)
예측값
  ↓ 손실 함수로 오차 계산
Loss
  ↓ 역전파(Backward)로 gradient 계산
∂Loss/∂W
  ↓ 옵티마이저로 가중치 업데이트
더 좋은 모델
```

### 15-2. 모델 선택 치트시트

| 데이터 형태 | 추천 모델 | 
|---|---|
| 표 형태 데이터 (정형 데이터) | MLP |
| 이미지 | CNN |
| 시계열 / 센서 시퀀스 | LSTM (또는 작은 Transformer) |
| 텍스트, 자연어 | Transformer (사전학습 모델 활용) |
| 이미지 + 시퀀스 (예: 비디오) | CNN + LSTM, 또는 Video Transformer |

---