# Supervised Learning

정답이 포함된 데이터를 가지고, 입력이 들어왔을 때 정답을 예측하는 규칙을 학습하는 방법

```
입력값 + 정답 → 학습(fit) → 모델

입력값 → 모델 → 예측값(predict)
``` 

- 입력값 = 특징(feature), 독립변수, x
- 정답 = 라벨(label), 타깃(target), 종속변수, y

---

## 1. 분류 (Classification)

결과가 “카테고리(종류)”인 경우

예시:

- 고양이/강아지 분류
- 스팸 메일 분류
- 암 진단

---

## 2. 회귀 (Regression)

결과가 “숫자(연속값)”인 경우

예시:

- 집값 예측
- 주가 예측
- 온도 예측

---

## 3. 데이터

```
| 공부시간 | 출석률 | 과제제출 | 합격여부 |
|---|---|---|---|
| 5 | 80 | 1 | 1 |
| 2 | 50 | 0 | 0 |
| 7 | 90 | 1 | 1 |

x: [공부시간, 출석률, 과제제출]
y: 합격여부
```

```python
import pandas as pd

df = pd.DataFrame(data)

# 입력값(특징)
X = df[["study_hours", "attendance", "assignment"]]

# 정답(라벨)
y = df["pass"]
```

### 3-1. 데이터 분리

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split( # 랜덤하게 섞음
    X, y,
    test_size=0.2,   # 20% 평가용, 80% 훈련용
    random_state=42  # 재현성(항상 같은 데이터 분할)
)
```
 
---

## 4. 학습

정답과 예측의 차이를 줄이도록 규칙을 계속 수정하는 과정

- 예측값: 모델이 계산한 값
- 실제값: 정답
- 오차: 실제값 - 예측값
- 손실 함수 (Loss Function): 오차들을 하나의 숫자로 만드는 함수

### 4-1. 손실 함수 (Loss Function)

**평균제곱오차 (Mean Squared Error, MSE)**

```
(1/n) * Σ(실제값 - 예측값)^2
```

### model.fit(x,y) 내부 동작

1. 초기화: w와 b를 랜덤값으로 설정
2. 예측: 현재 w, b로 예측값 계산
3. 손실 계산: 예측값과 실제값의 차이 계산
4. 업데이트: 손실이 줄어드는 방향으로 w, b 수정
5. 반복: 손실이 더 이상 줄어들지 않을 때까지 반복

### 4-2. 과적합 (Overfitting)

모델이 학습 데이터만 너무 잘 외운 상태 → train은 잘 맞는데 test는 못 맞음

원인:

- 데이터가 너무 적음
- 모델이 너무 복잡함
- 노이즈까지 외움
- feature가 너무 많음

> feature = 입력 항목 수 != 데이터 개수

해결:

- 데이터 늘리기
- 모델 단순화
- 정규화(Regularization)
- 교차검증(Cross-validation)
