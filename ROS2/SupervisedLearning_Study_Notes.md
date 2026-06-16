# Supervised Learning

정답이 포함된 데이터를 가지고, 입력이 들어왔을 때 정답을 예측하는 규칙을 학습하는 방법

```text
입력값 + 정답 → 학습(fit) → 모델

입력값 → 모델 → 예측값(predict)
```

- 입력값 = 특징(feature), 독립변수, X
- 정답 = 라벨(label), 타깃(target), 종속변수, y

---

# 1. 전체 흐름

```text
1. 데이터 준비
      ↓
2. 데이터 분리 (Train/Test)
      ↓
3. 모델 학습 (fit)
      ↓
4. 예측 (predict)
      ↓
5. 평가 (Accuracy, F1 등)
      ↓
6. 개선 (과적합 해결, 하이퍼파라미터 조정)
```

---

# 2. 문제 유형

## 2-1. 분류 (Classification)

결과가 "카테고리(종류)"인 경우

예시

- 고양이 / 강아지 분류
- 스팸 메일 분류
- 암 진단

대표 알고리즘

- Logistic Regression
- Decision Tree
- Random Forest

---

## 2-2. 회귀 (Regression)

결과가 "숫자(연속값)"인 경우

예시

- 집값 예측
- 주가 예측
- 온도 예측

대표 알고리즘

- Linear Regression
- Random Forest Regressor

---

# 3. 데이터 준비

예시 데이터

| 공부시간 | 출석률 | 과제제출 | 합격여부 |
|---|---|---|---|
| 5 | 80 | 1 | 1 |
| 2 | 50 | 0 | 0 |
| 7 | 90 | 1 | 1 |

```text
X = [공부시간, 출석률, 과제제출]
y = 합격여부
```

```python
import pandas as pd

df = pd.DataFrame(data)

# 입력값(특징)
X = df[["study_hours", "attendance", "assignment"]]

# 정답(라벨)
y = df["pass"]
```

### Feature와 Data 개수 차이

```text
feature 수 = 입력 항목 수
data 수 = 행(row) 개수
```

예시

```text
100명 학생 데이터

공부시간
출석률
과제제출

→ feature = 3개
→ data = 100개
```

---

# 4. 데이터 분리

학습용 데이터와 평가용 데이터를 분리

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,   # 20% 평가용
    random_state=42  # 항상 같은 방식으로 분리 (재현성)
)
```

---

# 5. 학습 (Training)

정답과 예측의 차이를 줄이도록 규칙을 계속 수정하는 과정

용어

- 예측값(Prediction): 모델이 계산한 값
- 실제값(Ground Truth): 정답
- 오차(Error): 실제값 - 예측값
- 손실 함수(Loss Function): 오차들을 하나의 숫자로 만드는 함수

---

## 5-1. 손실 함수 (Loss Function)

모델이 얼마나 틀렸는지 숫자로 계산하는 함수

### 평균제곱오차 (Mean Squared Error, MSE)

```text
(1/n) * Σ(실제값 - 예측값)^2
```

특징

- 오차가 클수록 큰 패널티
- 값이 작을수록 좋음

---

## 5-2. model.fit(X, y) 내부 동작

```text
1. w, b 초기화
2. 예측값 계산
3. 손실 계산
4. 손실이 줄어드는 방향으로 w, b 수정
5. 반복
```

### 선형 모델 예시

```text
예측값 = wX + b
```

- w = 가중치(weight)
- b = 편향(bias)

---

# 6. 과적합 (Overfitting)

모델이 학습 데이터를 지나치게 외운 상태

```text
Train 성능 ↑
Test 성능 ↓
```

원인

- 데이터가 너무 적음
- 모델이 너무 복잡함
- 노이즈까지 학습
- Feature가 너무 많음

해결 방법

- 데이터 늘리기
- 모델 단순화
- 정규화(Regularization)
- 교차검증(Cross Validation)

---

## 6-1. 정규화 (Regularization)

가중치(파라미터)가 지나치게 커지는 것을 막기 위해 패널티를 부여

목적

```text
복잡한 모델
      ↓
단순한 모델
      ↓
과적합 감소
```

대표 예시

- L1 Regularization (Lasso)
- L2 Regularization (Ridge)

---

## 6-2. 교차검증 (Cross Validation)

데이터를 여러 번 나눠서 학습과 평가를 반복

예시: 5-Fold Cross Validation

```text
1 2 3 4 | 5 평가
1 2 3 5 | 4 평가
1 2 4 5 | 3 평가
1 3 4 5 | 2 평가
2 3 4 5 | 1 평가
```

장점

- 데이터 활용 효율 증가
- 성능 평가 신뢰성 향상

---

# 7. 평가 (Evaluation)

## 7-1. Confusion Matrix (혼동 행렬)

분류 모델의 성능을 시각적으로 확인하는 표

| 실제 \ 예측 | 긍정 | 부정 |
|---|---|---|
| 긍정 | TP | FN  |
| 부정 | FP | TN |

- TP (True Positive): 실제 긍정 → 긍정 예측 (정답)
- TN (True Negative): 실제 부정 → 부정 예측 (정답)
- FP (False Positive): 실제 부정 → 긍정 예측 (오탐, Type 1 Error)
- FN (False Negative): 실제 긍정 → 부정 예측 (미탐, Type 2 Error)

---

## 7-2. 평가지표

- Accuracy (정확도): 전체 중에서 맞춘 비율 `(TP + TN) / (TP + TN + FP + FN)`
- Precision (정밀도): 긍정으로 예측한 것 중에서 실제 긍정인 비율 `TP / (TP + FP)`
- Recall (재현율): 실제 긍정인 것 중에서 긍정으로 예측한 비율 `TP / (TP + FN)`
- F1-Score (F1 점수): 정밀도와 재현율의 조화평균 `2 * (Precision * Recall) / (Precision + Recall)`

---

### 평가 코드

```python
from sklearn.metrics import (
    accuracy_score,
    precision_score,
    recall_score,
    f1_score
)

print("Accuracy:", accuracy_score(y_test, pred))
print("Precision:", precision_score(y_test, pred))
print("Recall:", recall_score(y_test, pred))
print("F1:", f1_score(y_test, pred))
```

---

# 8. 대표 알고리즘

## 8-1. Logistic Regression

입력을 확률(0~1)로 변환한 뒤 분류

```text
입력
 ↓
확률 계산
 ↓
Threshold 비교
 ↓
0 또는 1 분류
```

```python
from sklearn.linear_model import LogisticRegression

model = LogisticRegression()
model.fit(X_train, y_train)
```

장점

- 빠름
- 해석이 쉬움

단점

- 비선형 데이터에 약함
- Feature Scaling 필요

---

## 8-2. Decision Tree

질문(규칙)을 계속 던지며 분류

예시

```text
출석률 > 70 ?
 ├─ Yes → 과제 제출?
 │      ├─ Yes → 합격
 │      └─ No  → 불합격
 └─ No → 불합격
```

```python
from sklearn.tree import DecisionTreeClassifier

model = DecisionTreeClassifier()
model.fit(X_train, y_train)
```

장점

- 해석이 쉬움
- 비선형 데이터 처리 가능
- Feature Scaling 불필요

단점

- 과적합되기 쉬움

---

## 8-3. Ensemble

여러 모델의 결과를 결합하여 성능을 높이는 방법

대표 예시

- Random Forest
- Gradient Boosting
- XGBoost
- LightGBM
- AdaBoost

```text
여러 모델
      ↓
결과 결합
      ↓
더 안정적인 예측
```

---

## 8-4. Random Forest

Decision Tree 여러 개를 묶은 앙상블(Ensemble) 모델

```text
Tree 1
Tree 2
Tree 3
.../
Tree N
   ↓
다수결(Voting)
   ↓
최종 예측
```

```python
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier()
model.fit(X_train, y_train)
```

장점

- 과적합 감소
- 높은 성능
- 비선형 데이터 처리 가능

단점

- 모델이 복잡함
- 학습/예측이 상대적으로 느림
- 해석이 어려움

---