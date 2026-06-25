# Deep Learning (딥러닝)

```text
AI (인공지능)
│
└── Machine Learning (머신러닝)
    │
    └── Deep Learning (딥러닝)
```

**딥러닝 = 머신러닝의 한 종류**

---

## 1. 정의

인공 신경망(Artificial Neural Network)을 여러 층(layer)으로 깊게 쌓아 데이터의 특징(feature)을 자동으로 학습하는 방법.

```text
데이터  
  ↓  
신경망  
  ↓  
특징 자동 추출  
  ↓  
예측 결과
```

---

## 2. 신경망 구조

기본 구조:

```text
입력층(Input Layer)
        ↓
은닉층(Hidden Layer)
        ↓
출력층(Output Layer)
```

> 딥러닝은 은닉층이 여러 개

### 2-1. 신경망 계산

하나의 층에서는 다음 계산을 수행

`y = f(Wx + b)`

- x: 입력값
- W: 가중치(weight)
- b: 편향(bias)
- f: 활성화 함수(activation function)
- y: 출력값

---

## 3. 딥러닝 학습 과정

예: 고양이 이미지 분류

```text
픽셀 데이터
    ↓
선, 모서리 특징 추출
    ↓
눈, 귀 같은 패턴 학습
    ↓
얼굴 형태 학습
    ↓
고양이 판단
```

즉:

```text
픽셀
 ↓
저수준 특징
 ↓
중간 특징
 ↓
고수준 특징
 ↓
결과
```

여러 층을 통과하면서 점점 복잡한 특징을 학습합니다.

---

### 3-1. 딥러닝 특징

| 특징 | 설명 |
|---|---|
| 자동 특징 추출 | 사람이 feature를 직접 만들 필요 없음 |
| 비선형 학습 | 복잡한 패턴 학습 가능 |
| 대규모 데이터에 강함 | 데이터가 많을수록 성능 향상 |
| 높은 표현력 | 복잡한 데이터 처리 가능 |

---

### 3-2. 층의 깊이와 성능

층이 너무 적으면:

표현력 부족  
→ 복잡한 패턴 학습 어려움


층이 너무 많으면:

표현력 증가  
→ 과적합 가능

---

## 퍼셉트론 (Perceptron)

딥러닝의 가장 기본 단위 (뉴런 1개)

`y = f(Wx + b)`

```text
입력 x
 → 가중치 곱 (Wx)
 → 편향 더하기 (+b)
 → 활성화 함수 f()
 → 출력 y
```

## PyTorch

```python
import torch
import torch.nn as nn
import torch.optim as optim

# 입력 데이터
X = torch.tensor([
    [0., 0.],
    [0., 1.],
    [1., 0.],
    [1., 1.]
])

# 정답
y = torch.tensor([
    [0.],
    [0.],
    [0.],
    [1.]
])

# 모델 생성
model = Perceptron()

# 손실 함수 + 옵티마이저 설정
# 옵티마이저: w = w - lr * gradient
criterion = nn.BCELoss()       # Binary Cross Entropy
optimizer = optim.SGD(model.parameters(), lr=0.1)
```

> 딥러닝 연산은 대부분 실수 연산이라서 float으로 선언한다.
> 손실 함수: 모델의 예측값이 정답과 얼마나 다른지 측정하는 기준
> 옵티마이저: 계산된 오차를 이용해서 w, b를 업데이트하는 알고리즘

### 모델

```python
class Perceptron(nn.Module):
    def __init__(self):
        super().__init__()
        self.linear = nn.Linear(2, 1)  # 입력 2개 → 출력 1개

    # 모델이 입력을 받았을 때 실행됨
    def forward(self, x):
        return torch.sigmoid(self.linear(x))  # activation: 0~1로 변환
```

### 학습 루프

```python
for epoch in range(1000):

    # 모델 예측
    output = model(X)

    # loss 계산
    loss = criterion(output, y)

    # Gradient 초기화 - PyTorch는 기본적으로 gradient를 누적
    optimizer.zero_grad()

    # 역전파 - 어떤 w가 얼마나 영향을 줬는지 계산
    loss.backward()

    # 업데이트 - w, b를 loss가 줄어드는 방향으로 업데이트
    optimizer.step()

    if epoch % 100 == 0:
        print(f"Epoch {epoch}, Loss: {loss.item()}")
```

> 그래디언트: 현재 w, b를 얼마나 어느 방향으로 수정해야 loss(오차)가 줄어드는지 알려주는 값

### 결과 확인

```python
# gradient(미분 계산)를 끄고 모델 실행 - 계산 그래프 생성 X, gradient 저장 X
with torch.no_grad():
    print(model(X))
```

## 다층 신경만 (Multi Layer Perceptron)

퍼셉트론 1개 = 직선 하나 → 선형 분류

```text
입력층
  ↓
은닉층 (Hidden Layer 1)
  ↓
은닉층 (Hidden Layer 2)
  ↓
출력층
```

> 은닉층: 여러 개의 뉴런(퍼셉트론)을 모아놓은 층

중간층이 특징(feature)을 변환한다

```text
입력 → 선 → 곡선 → 복잡한 경계
```

```text
h1 = f(W1x + b1)
h2 = f(W2h1 + b2)
y  = f(W3h2 + b3)
```

## 활성화 함수 (Activation Function)

신경망에서 비선형성을 추가하는 함수.
입력을 받아서 다음 층으로 넘길 값을 변환하는 함수.

| 함수 | 특징 |
|---|---|
| Sigmoid | 0~1 |
| Tanh | -1~1 |
| ReLU | 가장 많이 사용 |

### sigmoid

용도: 이진 분류 마지막 출력층

`1 / (1+e^-x)`

단점:

- 큰 값에서 gradient가 거의 0이 됨 → 학습 느려짐 (vanishing gradient)

### tanh

`(e^x - e^-x) / (e^x + e^-x)`

### ReLU

`max(0, x)`

장점:

- 계산 빠름
- gradient 문제가 Sigmoid보다 적음
- 깊은 신경망에서 잘 작동

단점:

- 0보다 작은 값은 모두 0이 됨 → 학습 느려질 수 있음

## MLP

```python
class MLP(nn.Module):
    def __init__(self):
        super().__init__()
        self.model = nn.Sequential(
            nn.Linear(2, 4),
            nn.ReLU(),
            nn.Linear(4, 4),
            nn.ReLU(),
            nn.Linear(4, 1),
            nn.Sigmoid()
        )

    def forward(self, x):
        return self.model(x)
```

## SGD vs Adam

### SGD (Stochastic Gradient Descent)

```text
매 스텝마다 w, b를 조금씩 업데이트

장점:
- 구현 간단
- 메모리 적게 사용

단점:
- 학습 속도 느림
- local minima에 빠지기 쉬움
```

### Adam (Adaptive Moment Estimation)

```text
매 스텝마다 w, b를 적절히 업데이트

장점:
- 학습 속도 빠름
- local minima에 빠지지 않음

단점:
- 구현 복잡
- 메모리 많이 사용
```

## 손실함수

모델이 얼마나 틀렸는지 수치로 표현

분류: 

- Binary → BCELoss
- Multi → CrossEntropyLoss

회귀:

- MSELoss (평균제곱오차): `1/n * (y - y_pred)^2`

## 역전파

오차를 뒤로 전달해서 가중치를 업데이트

```text
forward (예측) → loss 계산 → backward → weight 업데이트
```

핵심: 미분(gradient) `∂loss / ∂w`

- gradient 크다 → 많이 수정
- gradient 작다 → 조금 수정

## 전체 흐름

```text
데이터 → 모델 → 예측 → loss → gradient → weight 업데이트
```