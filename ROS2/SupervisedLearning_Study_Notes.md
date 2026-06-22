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

# 3. 데이터

## 3-1. 데이터 준비

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
from sklearn.datasets import make_classification

# 기존 데이터 - Scikit-Learn의 입력 데이터는 2차원 형태여야 한다
df = pd.DataFrame(data)
X = df[["study_hours", "attendance", "assignment"]]
y = df["pass"]

# 분류용 가짜 데이터 - X(숫자 데이터), y(클래스)
X, y = make_classification(
    n_samples=10,       # 생성할 데이터 개수
    n_features=4,       # 특징 개수
    n_classes=2,        # 분류 클래스 개수 (y)
    weights=[0.9, 0.1], # 데이터 불균형 (기본값: [0.5, 0.5])
    random_state=42
)

# 회귀용 가짜 데이터 - X(숫자 데이터), y(숫자 데이터)
X, y = make_regression(
    n_samples=10,
    n_features=3,
    noise=5,            # 노이즈(오차)
    random_state=42
)
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

## 3-2. 데이터 분리

학습용 데이터와 평가용 데이터를 분리

```python
from sklearn.model_selection import train_test_split

# 셔플 → 분리
X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,   # 20% 평가용
    random_state=42, # 항상 같은 방식으로 분리 (재현성)
    stratify=y       # 클래스 비율 유지하면서 분리
)
```

---

## 3-3. 데이터 전처리

데이터 품질을 높여 모델이 더 안정적으로 학습하도록 하기 위해서 수행하는 작업

```text
X와 y, train/test 분리
      ↓
학습 데이터에만 전처리 fit (데이터 누수 방지)
      ↓
학습 데이터와 테스트 데이터 transform
      ↓
모델 학습
```

1. 결측치 처리: 데이터 누락으로 인한 학습 오류 방지

```python
from sklearn.impute import SimpleImputer

# 객체 생성 (mean: 평균값, median: 중앙값, most_frequent: 최빈값)
imputer = SimpleImputer(strategy="mean")

# fit(): 필요한 값 계산 + transform(): 데이터 변환
X_filled = imputer.fit_transform(X)
```

- `strategy="mean"`: 평균값
- `strategy="most_frequent"`: 최빈값

2. 범주형 처리: 범주형 데이터를 수치형 데이터로 변환

```python
from sklearn.preprocessing import OneHotEncoder
from sklearn.preprocessing import LabelEncoder

# 원-핫 인코딩: X의 범주형 변수
encoder = OneHotEncoder(sparse_output=False)
X_encoded = encoder.fit_transform(X)

# 라벨 인코딩: y(정답 라벨)의 범주형 변수 [자동 사전식 정렬]
encoder = LabelEncoder()
y_encoded = encoder.fit_transform(y)
y_decoded = encoder.inverse_transform(y_encoded) # 디코딩
```

3. 스케일링: 변수 간 스케일을 통일하여 모델 학습 안정성 및 성능 향상

```python
from sklearn.preprocessing import StandardScaler
from sklearn.preprocessing import MinMaxScaler

# 표준화(Standardization): 각 특성의 평균을 0, 표준편차를 1에 가깝게 변환
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# 정규화(Normalization): 0~1 사이의 값으로 변환
scaler = MinMaxScaler()
X_scaled = scaler.fit_transform(X)
```

> 데이터 전처리 후 반환된 데이터는 numpy 배열 형태이므로, 필요시 DataFrame으로 변환하여 확인
> `X_scaled_df = pd.DataFrame(X_scaled, columns=X.columns)`

---

## 3-4. 컬럼 별 전처리와 파이프라인

```python
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.pipeline import Pipeline

preprocessor = ColumnTransformer(
    transformers = [
        ("num", StandardScaler(), ["speed","temperature"]),
        ("cat", OneHotEncoder(), ["mode"])
    ]
)

# 파이프라인 생성 (전처리 + 모델을 하나로 묶어서 관리)
pipeline = Pipeline([
    ("preprocessor", preprocessor),
    ("model", LogisticRegression())
])

# 학습 (파이프라인 호출)
pipeline.fit(X_train, y_train)

# 예측 (파이프라인 호출)
y_pred = pipeline.predict(X_test)
```

---

## 3-5. 데이터 증강 (SMOTE)

소수 클래스의 가상 데이터를 생성하여 클래스 불균형 문제를 완화하는 오버샘플링 기법

```python
# 반드시 train 데이터에만 적용
from imblearn.over_sampling import SMOTE

smote = SMOTE()
X_res, y_res = smote.fit_resample(X_train, y_train)
```

---

## 3-6. Scikit-Learn 내장 데이터셋

교육용 데이터셋

```python
from sklearn.datasets import load_iris

iris = load_iris()

print("입력 데이터 크기:", iris.data.shape)     # (150, 4)
print("정답 데이터 크기:", iris.target.shape)   # (150,)
print("특징 이름:", iris.feature_names)
print("클래스 이름:", iris.target_names)
```

---

# 4. 학습 (Training)

정답과 예측의 차이를 줄이도록 규칙을 계속 수정하는 과정

용어

- 예측값(Prediction): 모델이 계산한 값
- 실제값(Ground Truth): 정답
- 오차(Error): 실제값 - 예측값
- 손실 함수(Loss Function): 오차들을 하나의 숫자로 만드는 함수

---

## 4-1. 손실 함수

모델이 얼마나 틀렸는지 숫자로 계산하는 함수

### 평균절대오차 (Mean Absolute Error, MAE)

`(1/n) * Σ|실제값 - 예측값|`

```python
from sklearn.metrics import mean_absolute_error

mae = mean_absolute_error(y_test, y_pred) # 2.13 → 약 2.13% 정도 틀림
```

### 평균제곱오차 (Mean Squared Error, MSE)

`(1/n) * Σ(실제값 - 예측값)^2`

```python
from sklearn.metrics import mean_squared_error

mse = mean_squared_error(y_test, y_pred)
```

### 평균제곱근오차 (Root Mean Squared Error, RMSE)

실제 데이터의 단위와 동일한 크기의 오차를 나타냄

`√[(1/n) * Σ(실제값 - 예측값)^2]`

```python
from sklearn.metrics import root_mean_squared_error

rmse = root_mean_squared_error(y_test, y_pred)
```

---

## 4-2. model.fit(X, y) 내부 동작

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

# 5. 대표 알고리즘

## 분류모델

### 기본 명령어

```python
# 학습된 클래스 확인
model.classes_

# 예측 클래스 2차원 배열 반환
model.predict(X_test)

# 예측 클래스별 확률 확인
model.predict_proba(X_test)
```

---

### 5-1. Logistic Regression

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

### 5-2. Decision Tree

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
from sklearn.tree import export_text

model = DecisionTreeClassifier(max_depth=3) # 깊이 제한 (기본값: None)
model.fit(X_train, y_train)

print(export_text(model, feature_names=list(X.columns))) # 트리 구조 출력
```

장점

- 해석이 쉬움
- 비선형 데이터 처리 가능
- Feature Scaling 불필요

단점

- 과적합되기 쉬움

---

### 5-3. Ensemble

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

### 5-4. Random Forest

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

model1 = RandomForestClassifier()
model1.fit(X_train, y_train)

# 데이터 불균형 시 적은 클래스에 더 큰 중요도를 부여
model2 = RandomForestClassifier(class_weight="balanced") 
model2.fit(X_train, y_train)

# n_estimators: 트리의 개수 (기본값: 100)
model=RandomForestClassifier(n_estimators=100)

# 특성 중요도: 각 특징이 예측에 얼마나 기여했는지
model.feature_importances_
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

### 5-5. KNeighborsClassifier

가장 가까운 k개의 이웃을 보고 다수결로 분류

```python
from sklearn.neighbors import KNeighborsClassifier

model = KNeighborsClassifier(n_neighbors=k) # 기본값: 5
model.fit(X_train, y_train)
```

장점

- 매우 쉽고 직관적
- 데이터가 적을 때 성능이 괜찮음
- 별도의 복잡한 학습 과정이 거의 없음

단점

- 데이터가 많아지면 느림
- 스케일링에 민감

---

### 5-6. SVC (Support Vector Classifier)

데이터를 가장 잘 구분할 수 있는 최적의 결정 경계(초평면)를 찾아 새로운 데이터를 분류.
SVM 기반 분류 모델.

```python
from sklearn.svm import SVC

model = SVC() # 기본값: C=1.0
model.fit(X_train, y_train)
y_pred = model.predict(X_test)

model = SVC(probability=True) # predict_proba() 사용 시 필요
model.fit(X_train, y_train)
y_pred_proba = model.predict_proba(X_test)

model = SVC(class_weight="balanced") # 데이터 불균형 시
```

장점

- 적은 데이터에서도 성능 좋음
- 고차원 데이터(특성이 많은 데이터)에 강함
- 커널 함수를 통해 비선형 문제도 해결 가능

단점

- 데이터 양이 많아질수록 학습 속도가 느려짐
- 파라미터 튜닝 필요
- 모델의 결과를 해석하기 어려움
- 특성 스케일에 민감 → 스케일링 필요

---

### 5-7. GaussianNB

각 클래스에 속할 확률을 계산한 뒤, 가장 높은 확률을 가진 클래스로 데이터를 분류
(각 특성의 값이 독립적이며 정규분포를 따른다고 가정)

```python
from sklearn.naive_bayes import GaussianNB

model = GaussianNB()
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
```

장점

- 학습 및 예측 속도가 빠름
- 적은 데이터에서도 안정적인 성능을 보임
- 고차원 데이터 처리에 유리함

단점

- 가정(특성 간 독립성, 데이터가 정규분포를 따름)이 실제 데이터와 맞지 않으면 성능 저하
- 복잡한 데이터 패턴을 표현하는 데 한계가 있음

---

### 5-8. ExtraTrees

Random Forest와 비슷하지만 더 무작위성이 강한 트리 앙상블 모델

```python
from sklearn.ensemble import ExtraTreesClassifier

model = ExtraTreesClassifier()
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
```

장점:

- 빠름
- 과적합 감소 효과
- feature가 많을 때 좋음

단점:

- Random Forest보다 항상 좋은 것은 아님
- 해석 어려움

---

### 5-9. Gradient Boosting

Decision Tree를 여러 개 이어 붙여서 성능을 개선하는 앙상블 모델

```python
from sklearn.ensemble import GradientBoostingClassifier

model = GradientBoostingClassifier()
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
```

장점

- 높은 성능 가능
- 비선형 패턴 학습
- 특징 중요도 확인 가능

단점

- 하이퍼파라미터에 민감
- 학습 시간이 길 수 있음
- 과적합 주의

---

## 회귀 모델

### 5-8. Linear Regression

입력(X)과 출력(y)의 관계를 직선(선형식)으로 모델링하여 숫자를 예측

```python
from sklearn.linear_model import LinearRegression

model = LinearRegression()
model.fit(X_train, y_train)

print("기울기(계수):", model.coef_)
print("절편:", model.intercept_)
```

장점

- 해석이 쉬움
- 구현이 간단함
- 학습이 빠름

단점

- 비선형 데이터에 약함
- Feature Scaling 필요
- 이상치에 민감

---

### 5-9. Ridge Regression

선형 회귀에 L2 규제를 추가해 과적합을 줄이는 모델

```python
from sklearn.linear_model import Ridge

model = Ridge(alpha=1.0) # 규제정도 (기본값: 1.0)
model.fit(X_train,y_train)
```

장점

- 과적합을 줄일 수 있음
- 모든 특성을 유지하면서 계수의 크기를 조절함
- 일반 선형회귀보다 예측 성능이 향상되는 경우가 많음

단점

- 불필요한 특성의 계수를 0으로 만들지 못함
- 하이퍼파라미터 튜닝 필요
- 선형 관계를 가정하므로 복잡한 비선형 패턴을 잘 표현하지 못함
- 모델 해석이 일반 선형회귀보다 다소 어려워질 수 있음

---

### 5-10. Lasso Regression

선형 회귀에 L1 규제를 추가하여 과적합을 줄이고,
불필요한 변수를 제거해 모델을 단순화하는 모델

```python
from sklearn.linear_model import Lasso

model = Lasso(alpha=1.0) # 규제정도 (기본값: 1.0)
model.fit(X_train,y_train)
```

장점

- 불필요한 특성의 계수를 0으로 만들어 특성 제거 효과 (Feature Selection) → 모델 해석 용이
- Ridge 회귀보다 과적합을 더 효과적으로 줄일 수 있음

단점

- 상관관계가 높은 특성들 중 일부를 임의로 제거할 수 있음
- 하이퍼파라미터 튜닝 필요
- 선형 관계를 가정하므로 복잡한 비선형 패턴을 잘 표현하지 못함

#### L1, L2 규제

- L1 규제: 계수의 절댓값의 합에 패널티를 부여 → 불필요한 특성의 계수를 0으로 만듦
- L2 규제: 계수의 제곱의 합에 패널티를 부여 → 계수의 크기를 줄임

---

### 5-11. Polynomial Regression

선형 회귀와 다항식을 결합한 모델

```python
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression

# x → [x, x²] 변환 / degree: 다항식 차수
poly = PolynomialFeatures(degree=2)
X_poly = poly.fit_transform(X)

# y = b0 + b1·x + b2·x²
model.fit(X_poly, y)
```

장점

- 비선형(곡선) 관계를 표현 가능
- 구현이 간단함 (PolynomialFeatures + LinearRegression)

단점

- 차수(degree)가 너무 크면 과적합(Overfitting) 발생
- 특성 수가 급격히 증가하여 계산량 증가

---

### 5-12. Decision Tree Regressor

질문을 따라가며 숫자값을 예측하는 나무 구조 회귀 모델

```python
from sklearn.tree import DecisionTreeRegressor

model = DecisionTreeRegressor(max_depth=4, random_state=42)
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
```

장점

- 모델 구조를 시각화할 수 있어 해석이 쉬움
- 데이터의 비선형 관계를 잘 학습함
- 데이터 전처리(정규화, 스케일링)가 거의 필요 없음
- 특성 간의 복잡한 상호작용을 표현할 수 있음

단점

- 과적합 발생 쉬움
- 데이터가 조금만 바뀌어도 모델 구조가 크게 달라질 수 있음
- 깊이가 깊어질수록 일반화 성능이 떨어질 수 있음

---

### 5-13. Random Forest Regressor

여러 개의 결정 트리를 학습한 후, 그 결과들을 평균내어 최종 예측값을 도출하는 모델

```python
from sklearn.ensemble import RandomForestRegressor

model = RandomForestRegressor(n_estimators=100, random_state=42)
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
```

장점

- 단일 결정 트리의 과적합 문제를 해결하여 일반화 성능이 우수함
- 다양한 데이터 패턴을 학습할 수 있어 예측 성능이 좋음
- 특성 중요도(Feature Importance)를 제공하여 모델 해석이 가능함
- 병렬 처리가 가능하여 학습 속도가 빠름

단점

- 모델이 복잡하여 단일 결정 트리보다 해석이 어려움
- 메모리 사용량이 비교적 많음

---

### 5-14. Gradient Boosting Regressor

이전 트리의 오차를 보완하는 방식으로 순차적으로 트리를 학습하는 앙상블 회귀 모델

```python
from sklearn.ensemble import GradientBoostingRegressor

model = GradientBoostingRegressor(random_state=42)
model.fit(X_train, y_train)
```

장점

- 높은 예측 성능을 보이는 경우가 많음
- 비선형 데이터와 복잡한 패턴을 잘 학습함
- 특성 스케일링(정규화)이 필요하지 않음

단점

- 학습 속도가 비교적 느림
- 하이퍼파라미터 튜닝이 중요함
- 모델 구조가 복잡하여 해석이 어려움

---

# 6. 평가 (Evaluation)

## 분류

### 6-1. Confusion Matrix (혼동 행렬)

분류 모델의 성능을 시각적으로 확인하는 표

| 실제 \ 예측 | 긍정 | 부정 |
|---|---|---|
| 긍정 | TP | FN |
| 부정 | FP | TN |

- TP (True Positive): 실제 긍정 → 긍정 예측 (정답)
- TN (True Negative): 실제 부정 → 부정 예측 (정답)
- FP (False Positive): 실제 부정 → 긍정 예측 (오탐, Type 1 Error)
- FN (False Negative): 실제 긍정 → 부정 예측 (미탐, Type 2 Error)

```python
from sklearn.metrics import confusion_matrix

cm = confusion_matrix(y_test, y_pred)
print(cm)

tn, fp, fn, tp = cm.ravel()
print("TN:", tn)
print("FP:", fp)
print("FN:", fn)
print("TP:", tp)
```

---

### 6-2. 분류 평가지표

- Accuracy (정확도): 전체 중에서 맞춘 비율 `(TP + TN) / (TP + TN + FP + FN)`

> 데이터 불균형 시 주의

- Precision (정밀도): 긍정으로 예측한 것 중에서 실제 긍정인 비율 `TP / (TP + FP)`
- Recall (재현율): 실제 긍정인 것 중에서 긍정으로 예측한 비율 `TP / (TP + FN)`
- F1-Score (F1 점수): 정밀도와 재현율의 조화평균 `2 * (Precision * Recall) / (Precision + Recall)`
- ROC-AUC: 여러 임계값에서 양성과 음성을 구분하는 모델의 능력을 평가하는 지표
- ROC Curve: ROC-AUC를 시각화한 그래프
    - FPR: False Positive Rate (X축) `FP / (FP + TN)`
    - TPR: True Positive Rate (Y축) `TP / (TP + FN)`
    - Thresholds: 임계값
- Precision-Recall Curve: 정밀도와 재현율의 관계를 시각화한 그래프
    - Precision: 정밀도 (Y축) `TP / (TP + FP)`
    - Recall: 재현율 (X축) `TP / (TP + FN)`
    - Thresholds: 임계값

---

### 6-3. 분류 평가 코드

```python
from sklearn.metrics import (
    accuracy_score,
    precision_score,
    recall_score,
    f1_score,
    classification_report,
    roc_auc_score,
    roc_curve,
    precision_recall_curve
)

pred = model.predict(X_test) # 반환값: 배열

print("Accuracy:", accuracy_score(y_test, pred))
print("Precision:", precision_score(y_test, pred))
print("Recall:", recall_score(y_test, pred))
print("F1:", f1_score(y_test, pred))
print(classification_report(y_test, pred)) # 지표를 한 번에 출력

# 확률값 필요!
prob = model.predict_proba(X_test)[:, 1] # 클래스 1일 확률만 추출
print("ROC-AUC:", roc_auc_score(y_test, prob))

fpr, tpr, thresholds = roc_curve(y_true, danger_probabilities)
print("FPR:", fpr)
print("TPR:", tpr)
print("Thresholds:", thresholds)

precision, recall, thresholds = precision_recall_curve(
    y_true,
    danger_probabilities
)
print("Precision:", precision)
print("Recall:", recall)
print("Thresholds:", thresholds)
```

> model.score(X_test, y_test) : accuracy 출력

---

## 회귀

### 6-4. 회귀 평가지표

- R² Score (결정계수): 모델이 목표값의 변동을 얼마나 잘 설명하는가

```python
from sklearn.metrics import r2_score

r2 = r2_score(y_test, y_pred) # (0 ~ 1) - 1에 가까울수록 좋음
```

> model.score(X_test, y_test) : R² Score 출력

- mae
- mse
- rmse

---

# 7. 과적합 (Overfitting), 일반화 (Generalization)

**과적합**: 모델이 학습 데이터를 지나치게 외운 상태

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

**일반화**: 학습하지 않은 새로운 데이터에 대해서도 좋은 성능을 내는 능력

---

## 7-1. 정규화 (Regularization)

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

## 7-2. 교차검증 (Cross Validation)

학습 데이터를 여러 번 나눠서 학습과 평가를 반복

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

```python
from sklearn.model_selection import cross_val_score

scores = cross_val_score(model, X, y, cv=5) # cv: 폴드 개수 (기본값: 5)
```

---

## 7-3. 하이퍼파라미터 튜닝

### 하이퍼파라미터

모델이 학습하기 전에 사람이 정해주는 설정값

```text
KNN의 n_neighbors
DecisionTree의 max_depth
RandomForest의 n_estimators
SVM의 C, kernel
GradientBoosting의 learning_rate
```

### GridSearchCV

여러 하이퍼파라미터 조합을 자동으로 비교

```python
from sklearn.model_selection import GridSearchCV
from sklearn.ensemble import RandomForestClassifier

# 탐색할 하이퍼파라미터
param_grid= {
    "n_estimators": [50,100,200],
    "max_depth": [3,5,None]
}

grid=GridSearchCV(
    RandomForestClassifier(random_state=42),
    param_grid=param_grid,
    cv=5
)

grid.fit(X_train,y_train)

best_model = grid.best_estimator_ # 최적의 파라미터로 학습된 모델

print(grid.best_params_)  # 최적의 하이퍼파라미터 조합
```

### Pipeline + GridSearchCV

```python
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.pipeline import Pipeline
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import GridSearchCV

# 컬럼별 전처리
preprocessor = ColumnTransformer(
    transformers=[
        ("num", StandardScaler(), ["speed", "temperature"]),
        ("cat", OneHotEncoder(), ["mode"])
    ]
)

# 파이프라인 생성 (전처리 + 모델)
pipeline = Pipeline([
    ("preprocessor", preprocessor),
    ("model", RandomForestClassifier(random_state=42))
])

# 탐색할 하이퍼파라미터
# Pipeline 내부의 특정 단계 파라미터를 GridSearchCV에서 조절하려면 "단계이름__파라미터이름" 형식으로 지정
param_grid = {
    "model__n_estimators": [50, 100, 200],
    "model__max_depth": [3, 5, None]
}

# GridSearchCV
grid = GridSearchCV(
    estimator=pipeline,
    param_grid=param_grid,
    cv=5
)

# 학습 (전처리 + 교차 검증 + 최적 모델 학습)
grid.fit(X_train, y_train)

# 최적의 하이퍼파라미터
print(grid.best_params_)

# 최적의 모델
best_model = grid.best_estimator_

# 예측
y_pred = best_model.predict(X_test)
```

---

# 8. 모델 저장

## Python 라이브러리

- joblib : Python 객체를 파일로 저장하고 불러오는 라이브러리

```python
import joblib

# 저장
joblib.dump(model,"robot_fault_model.pkl")

# 불러오기
loaded_model = joblib.load("robot_fault_model.pkl")
```