# 딥러닝

## 1. AI, 머신러닝, 딥러닝의 관계

```
AI (인공지능)
│
│
└── Machine Learning (머신러닝)
    │
    │
    │
    └── Deep Learning (딥러닝)
```

### 왜 딥러닝이 주목받는가?

| 시대 | 방식 | 한계 |
|---|---|---|
| 전통 프로그래밍 | 사람이 규칙을 직접 작성 | 규칙이 복잡해지면 한계 |
| 전통 머신러닝 | 사람이 특징(feature)을 추출 후 학습 | 좋은 feature 설계가 어려움 |
| **딥러닝** | **데이터에서 특징을 자동으로 추출하여 학습** | **대용량 데이터와 GPU 필요** |

> **핵심 포인트:** 딥러닝은 "특징 추출"을 사람 대신 모델이 스스로 합니다.

---

## 2. 신경망의 기본 단위 — 퍼셉트론

### 2-1. 생물학적 뉴런에서 영감

우리 뇌의 뉴런은 여러 신호를 받아 일정 임계치 이상이면 다음 뉴런에 신호를 전달합니다.
퍼셉트론은 이것을 수학적으로 모방한 것입니다.

```
입력 신호들 → 가중합 계산 → 활성화 함수 → 출력
```

### 2-2. 퍼셉트론의 수식

```
y = f(Wx + b)
```

| 기호 | 의미 |
|---|---|
| x | 입력값 |
| W | 가중치 (weight) |
| b | 편향 (bias) |
| f | 활성화 함수 |
| y | 출력값 |

### 2-3. 퍼셉트론이 하는 일

퍼셉트론 1개는 **직선 하나**로 데이터를 나누는 역할을 합니다.

```
    o o                ← 고양이
  o   o
─────────────  ← 퍼셉트론이 그은 경계선
  x   x
    x x                ← 강아지
```

이것만으로는 복잡한 문제를 풀 수 없습니다. 그래서 여러 개를 쌓습니다.

---

## 3. 활성화 함수

활성화 함수가 없으면 아무리 층을 많이 쌓아도 **수학적으로 1개 층과 동일**합니다.
활성화 함수는 **비선형성**을 추가해 복잡한 패턴을 학습할 수 있게 해줍니다.

### 3-1. 주요 활성화 함수

#### Sigmoid

```
σ(x) = 1 / (1 + e^(-x))
```

- 출력 범위: **0 ~ 1**
- 확률처럼 해석 가능 → **이진 분류 출력층**에 사용
- 단점: 입력이 매우 크거나 작으면 gradient가 거의 0이 됨 (**Vanishing Gradient** 문제)

#### Tanh

```
tanh(x) = (e^x - e^(-x)) / (e^x + e^(-x))
```

- 출력 범위: **-1 ~ 1**
- Sigmoid보다 중심이 0에 가까워 학습이 약간 더 안정적

#### ReLU (Rectified Linear Unit) ← 가장 많이 사용

```
ReLU(x) = max(0, x)
```

- **계산이 빠르고 깊은 신경망에서 효과적**
- 현재 은닉층의 표준 활성화 함수

### 3-2. 어디에 어떤 함수를?

```
은닉층:  ReLU (기본)
출력층:  이진 분류 → Sigmoid
        다중 분류 → Softmax
        회귀 → 없음 (선형 출력)
```

---

## 4. 다층 신경망 (MLP)

### 4-1. 신경망 구조

```
입력층 (Input Layer)   ← 데이터 받는 곳
        ↓
은닉층 (Hidden Layer 1)  ← 특징 추출
        ↓
은닉층 (Hidden Layer 2)  ← 더 복잡한 특징 추출
        ↓
출력층 (Output Layer)   ← 최종 예측
```

> **딥러닝 = 은닉층이 여러 개 있는 신경망**

### 4-2. 층이 깊어질수록 추상적인 특징을 학습

고양이 이미지 분류를 예로 들면:

```
픽셀 (raw data)
    ↓  1번째 은닉층
선, 모서리, 색상 감지
    ↓  2번째 은닉층
눈, 귀, 코 모양 인식
    ↓  3번째 은닉층
얼굴 전체 패턴 학습
    ↓  출력층
"고양이" 판단 (확률)
```

### 4-3. 층의 깊이와 성능 트레이드오프

| 층 수 | 결과 |
|---|---|
| 너무 적음 | 표현력 부족 → 단순한 패턴만 학습 (Underfitting) |
| 적당함 | 좋은 성능 ✓ |
| 너무 많음 | 표현력 과잉 → 훈련 데이터에만 맞춰짐 (Overfitting) |

---

## 5. 딥러닝 학습 과정

딥러닝의 학습은 아래의 사이클을 반복합니다.

```
① Forward Pass     데이터 → 신경망 통과 → 예측값 생성
        ↓
② Loss 계산        예측값과 정답을 비교해 오차 수치화
        ↓
③ Backward Pass    오차의 원인을 역방향으로 추적 (역전파)
        ↓
④ 가중치 업데이트  gradient를 이용해 W, b를 조금씩 수정
        ↓
    (①로 돌아가 반복)
```

### 5-1. Epoch, Batch, Iteration

| 용어 | 의미 |
|---|---|
| Epoch | 전체 데이터를 한 번 모두 학습 |
| Batch Size | 한 번에 처리하는 데이터 수 |
| Iteration | 1 Epoch을 완료하는 데 필요한 업데이트 횟수 |

예: 데이터 1,000개, Batch Size 100 → 1 Epoch = 10 Iterations

---

## 6. 손실 함수

손실 함수(Loss Function)는 **모델의 예측이 얼마나 틀렸는지**를 수치로 나타냅니다.
학습의 목표는 이 값을 최소화하는 것입니다.

### 6-1. 문제 유형별 손실 함수

#### 분류 문제 (Classification)

**Binary Cross Entropy (BCELoss)**
- 이진 분류 (예: 스팸 메일 / 아님)
- 출력층에 Sigmoid 사용

```
BCE = -[y·log(ŷ) + (1-y)·log(1-ŷ)]
```

**Cross Entropy Loss (CrossEntropyLoss)**
- 다중 분류 (예: 고양이 / 강아지 / 새)
- 출력층에 Softmax 내장

#### 회귀 문제 (Regression)

**Mean Squared Error (MSELoss)**
- 연속적인 값 예측 (예: 집값, 온도)

```
MSE = (1/n) * Σ(y - ŷ)²
```

### 6-2. 손실 함수 선택 가이드

```
문제 유형 판단
    │
    ├── 숫자 예측 (회귀) → MSELoss
    │
    └── 분류
          ├── 2가지 중 하나 → BCELoss
          └── 3가지 이상 중 하나 → CrossEntropyLoss
```

---

## 7. 역전파와 옵티마이저

### 7-1. 역전파 (Backpropagation)

역전파는 **오차의 원인을 각 가중치에 얼마나 돌릴 것인지** 계산하는 방법입니다.

핵심 개념: **미분 (Gradient)**

```
∂Loss / ∂W  →  "W를 조금 바꾸면 Loss가 얼마나 변하는가?"
```

- gradient가 크다 → 이 W가 오차에 많이 기여 → 많이 수정
- gradient가 작다 → 이 W가 오차에 적게 기여 → 조금 수정

```
forward (예측) → loss 계산 → backward (gradient 계산) → weight 업데이트
```

### 7-2. 옵티마이저 (Optimizer)

계산된 gradient를 이용해 실제로 가중치를 업데이트하는 알고리즘입니다.

**기본 업데이트 공식:**
```
W = W - lr × gradient
```

여기서 `lr`은 **학습률(Learning Rate)** — 한 번에 얼마나 크게 이동할지.

#### SGD (Stochastic Gradient Descent)

```python
optimizer = optim.SGD(model.parameters(), lr=0.1)
```

- 구현 간단, 메모리 효율적
- 수렴이 느리고 Local Minima에 빠질 위험

#### Adam (Adaptive Moment Estimation)

```python
optimizer = optim.Adam(model.parameters(), lr=0.001)
```

- 학습률을 각 파라미터마다 자동으로 조절
- **실무에서 가장 널리 사용**
- 빠른 수렴, Local Minima 탈출 능력 우수

#### 학습률(Learning Rate)의 중요성

```
lr 너무 크면:  Loss가 발산 (왔다 갔다)
lr 너무 작으면: 학습이 너무 느림
lr 적당히:     안정적으로 최솟값으로 수렴  ✓
```

---

## 8. 과적합과 해결책

### 8-1. 과적합이란?

훈련 데이터는 완벽하게 맞추지만, **새로운 데이터는 틀리는 현상**

```
훈련 데이터 Loss: 0.01  (매우 낮음)
테스트 데이터 Loss: 0.95  (매우 높음) ← 과적합!
```

### 8-2. 데이터 분할

과적합 감지를 위해 데이터를 나눠 사용합니다.

```
전체 데이터
    │
    ├── 훈련 데이터 (Train)    70~80%  ← 학습에 사용
    ├── 검증 데이터 (Validation) 10~15%  ← 과적합 감지
    └── 테스트 데이터 (Test)   10~15%  ← 최종 성능 평가
```

### 8-3. 해결 방법

#### ① Dropout

학습 중에 무작위로 일부 뉴런을 꺼버립니다.
특정 뉴런에 의존하지 못하게 해서 **일반화 능력**을 키웁니다.

```python
nn.Dropout(p=0.3)  # 30% 뉴런을 랜덤으로 비활성화
```

> 주의: Dropout은 **학습 시에만** 동작합니다. 추론 시에는 자동으로 꺼집니다.

#### ② Batch Normalization

각 층의 출력값을 정규화해서 학습을 안정화시킵니다.
학습 속도를 높이고 과적합도 줄여주는 효과가 있습니다.

```python
nn.BatchNorm1d(num_features)
```

#### ③ Early Stopping

검증 데이터의 성능이 더 이상 개선되지 않으면 학습을 중단합니다.

```
Epoch 1:   Val Loss 0.8  ↓
Epoch 10:  Val Loss 0.3  ↓
Epoch 20:  Val Loss 0.2  ↓
Epoch 30:  Val Loss 0.25 ↑ ← 여기서 중단! (과적합 시작)
```

#### ④ 권장 층 구성 순서

```
Linear → BatchNorm → ReLU → Dropout
```

---

## 9. PyTorch 실습

### 9-1. 환경 설정

```bash
pip install torch torchvision
```

### 9-2. 퍼셉트론 구현 (AND 게이트)

AND 게이트: 두 입력이 모두 1일 때만 1을 출력

```python
import torch
import torch.nn as nn
import torch.optim as optim

# 입력 데이터 (AND 게이트)
X = torch.tensor([
    [0., 0.],
    [0., 1.],
    [1., 0.],
    [1., 1.]
])

# 정답
y = torch.tensor([
    [0.],  # 0 AND 0 = 0
    [0.],  # 0 AND 1 = 0
    [0.],  # 1 AND 0 = 0
    [1.]   # 1 AND 1 = 1
])

# 모델 정의
class Perceptron(nn.Module):
    def __init__(self):
        super().__init__()
        self.linear = nn.Linear(2, 1)   # 입력 2개 → 출력 1개

    def forward(self, x):
        return torch.sigmoid(self.linear(x))  # 0~1 사이 확률 출력

# 모델, 손실함수, 옵티마이저 설정
model = Perceptron()
criterion = nn.BCELoss() # 이진 분류 손실 함수
optimizer = optim.SGD(model.parameters(), lr=0.1)

# 학습 루프
for epoch in range(1000):
    output = model(X)           # ① Forward: 예측
    loss = criterion(output, y) # ② Loss 계산

    optimizer.zero_grad()       # gradient 초기화 (누적 방지)
    loss.backward()             # ③ Backward: gradient 계산
    optimizer.step()            # ④ 가중치 업데이트

    if epoch % 100 == 0:
        print(f"Epoch {epoch:4d} | Loss: {loss.item():.4f}")

# 결과 확인
with torch.no_grad():  # 추론 시에는 gradient 계산 불필요
    predictions = model(X)
    print("\n예측 결과:")
    for i, (inputs, pred) in enumerate(zip(X, predictions)):
        print(f"  {inputs.tolist()} → {pred.item():.4f}")
```

### 9-3. MLP 구현 (XOR 문제)

XOR은 퍼셉트론 1개로 풀 수 없는 문제입니다. 은닉층이 필요합니다.

```python
# XOR 데이터
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

### 9-4. 과적합 방지가 적용된 MLP

```python
class ImprovedMLP(nn.Module):
    def __init__(self):
        super().__init__()
        self.model = nn.Sequential(
            nn.Linear(2, 16),
            nn.BatchNorm1d(16),   # 정규화
            nn.ReLU(),
            nn.Dropout(0.3),      # 30% 드롭아웃

            nn.Linear(16, 16),
            nn.BatchNorm1d(16),
            nn.ReLU(),

            nn.Linear(16, 1),
            nn.Sigmoid()
        )

    def forward(self, x):
        return self.model(x)

model = ImprovedMLP()

# 학습/평가 모드 전환
model.train()   # Dropout, BatchNorm 활성화 (학습 시)
model.eval()    # Dropout 비활성화 (추론 시)
```

### 9-5. 학습 루프 핵심 패턴 정리

```python
# 매 epoch마다 반복하는 패턴
for epoch in range(num_epochs):

    # --- 학습 단계 ---
    model.train()
    for X_batch, y_batch in train_loader:
        output = model(X_batch)           # Forward
        loss = criterion(output, y_batch) # Loss
        optimizer.zero_grad()             # Gradient 초기화
        loss.backward()                   # Backward
        optimizer.step()                  # 업데이트

    # --- 검증 단계 ---
    model.eval()
    with torch.no_grad():
        for X_val, y_val in val_loader:
            val_output = model(X_val)
            val_loss = criterion(val_output, y_val)
```

---

## 10. 정리 및 다음 단계

### 핵심 개념 요약

```
데이터
  ↓
신경망 (퍼셉트론들의 집합)
  ↓ 활성화 함수로 비선형성 추가
특징 자동 추출 (층이 깊을수록 복잡한 특징)
  ↓ 순전파
예측값
  ↓ 손실 함수로 오차 계산
Loss
  ↓ 역전파로 gradient 계산
∂Loss/∂W
  ↓ 옵티마이저로 가중치 업데이트
더 좋은 모델
```