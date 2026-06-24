# 지도학습(Supervised Learning)

정답이 있는 데이터를 보여주면서 → 규칙을 스스로 찾게 하는 것

```text
입력값 + 정답 → 학습(fit) → 모델

입력값 → 모델 → 예측값(predict)
```

| 용어 | 다른 이름 | 예시 |
|------|----------|------|
| 입력값 | 특징(feature), 독립변수, X | 공부시간, 출석률 |
| 정답 | 라벨(label), 타겟(target), 종속변수, y | 합격/불합격 |
| 모델 | 알고리즘 | 규칙을 담은 상자 |

---

## CHAPTER 1. 지도학습의 두 가지 유형

### 1-1. 분류 (Classification) — 답이 "종류"일 때

예측 결과가 카테고리(종류)인 경우

| 예시 | 입력(X) | 예측(y) |
|------|--------|--------|
| 스팸 메일 분류 | 메일 내용, 발신자 | 스팸 / 정상 |
| 암 진단 | 혈액 수치, 나이 | 양성 / 음성 |
| 고양이 vs 강아지 | 이미지 픽셀 | 고양이 / 강아지 |

### 1-2. 회귀 (Regression) — 답이 "숫자"일 때

예측 결과가 연속적인 수치인 경우

| 예시 | 입력(X) | 예측(y) |
|------|--------|--------|
| 집값 예측 | 면적, 위치, 층수 | 3억 2천만원 |
| 주가 예측 | 거래량, 전날 종가 | 87,500원 |
| 온도 예측 | 날짜, 위치, 기압 | 23.4℃ |

---

## CHAPTER 2. 전체 흐름

```
데이터 준비
    ↓
데이터 분리 (Train / Test)
    ↓
데이터 전처리 (결측치, 인코딩, 스케일링)
    ↓
모델 선택 & 학습 (fit)
    ↓
예측 (predict)
    ↓
평가 (Accuracy, F1, RMSE 등)
    ↓
개선 (하이퍼파라미터 튜닝, 교차검증)
```

---

## CHAPTER 3. 데이터 준비

### 3-1. 데이터가 어떻게 생겼는지 이해하기

| 공부시간 | 출석률 | 과제제출 | **합격여부** |
|---------|-------|---------|------------|
| 5 | 80 | 1 | **1** |
| 2 | 50 | 0 | **0** |
| 7 | 90 | 1 | **1** |

- **X (입력)**: 공부시간, 출석률, 과제제출 → feature
- **y (정답)**: 합격여부 → label

```python
import pandas as pd

data = {
    "study_hours": [5, 2, 7],
    "attendance": [80, 50, 90],
    "assignment": [1, 0, 1],
    "pass": [1, 0, 1]
}

df = pd.DataFrame(data)

X = df[["study_hours", "attendance", "assignment"]]  # 입력
y = df["pass"]                                       # 정답
```

> ⚠️ Scikit-Learn은 X가 반드시 **2차원 배열(행렬)** 형태여야 합니다.  
> `df["컬럼"]`은 1차원, `df[["컬럼"]]`은 2차원이니 대괄호 두 개를 꼭 쓰세요.

---

### 3-2. 연습용 가짜 데이터 만들기

실제 데이터가 없을 때 Scikit-Learn으로 연습용 데이터를 만들 수 있습니다.

```python
from sklearn.datasets import make_classification, make_regression

# 분류용
X, y = make_classification(
    n_samples=1000,     # 행(데이터) 개수
    n_features=5,       # 열(feature) 개수
    n_classes=2,        # 클래스 개수
    random_state=42     # 재현성 고정
)

# 회귀용
X, y = make_regression(
    n_samples=1000,
    n_features=3,
    noise=10,           # 오차(노이즈) 정도
    random_state=42
)
```

> 💡 `random_state=42`는 "항상 같은 결과를 내라"는 씨앗값입니다. 어떤 숫자든 상관없어요.

---

### 3-3. 데이터 증강

소수 클래스의 가상 데이터를 생성하여 클래스 불균형 문제를 완화하는 오버샘플링 기법

```python
# 반드시 train 데이터에만 적용
from imblearn.over_sampling import SMOTE

smote = SMOTE()
X_res, y_res = smote.fit_resample(X_train, y_train)
```

---

## CHAPTER 4. 데이터 분리

- **Train 데이터**: 모델이 학습하는 데이터
- **Test 데이터**: 모델이 평가하는 데이터

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,      # 20% 평가용
    random_state=42,    # 재현성 고정
    stratify=y          # 클래스 비율 유지 (분류 문제에서 권장)
)
```

> 💡 `stratify=y`: 예를 들어 합격:불합격 = 7:3이라면, train/test 모두 7:3 비율을 유지합니다. 데이터 불균형 시 특히 중요합니다.

---

## CHAPTER 5. 데이터 전처리

데이터 품질을 높여 모델이 더 안정적으로 학습하도록 하기 위해서 수행하는 작업

> ⚠️ **절대 원칙**: 전처리는 **Train 데이터로만 fit()** 하고, Test 데이터는 **transform()만** 적용  
> 이유: Test 데이터 정보가 학습에 새어 들어가는 "데이터 누수(Data Leakage)"를 막기 위해

### 5-1. 결측치 처리

```python
from sklearn.impute import SimpleImputer

imputer = SimpleImputer(strategy="mean")   # 평균값으로 채우기

imputer.fit(X_train)                       # Train으로만 계산
X_train = imputer.transform(X_train)
X_test  = imputer.transform(X_test)        # Test는 transform만
```

> `strategy="median"`: 중앙값 / `strategy="most_frequent"`: 최빈값

### 5-2. 범주형 처리

머신러닝 모델은 숫자만 이해합니다.

```python
from sklearn.preprocessing import OneHotEncoder, LabelEncoder

# X에 있는 범주형 변수 → 원-핫 인코딩
# "서울" → [1, 0, 0], "부산" → [0, 1, 0], "제주" → [0, 0, 1]
encoder = OneHotEncoder(sparse_output=False)
X_encoded = encoder.fit_transform(X_train)

# y(정답)가 문자인 경우 → 라벨 인코딩
# "합격" → 1, "불합격" → 0
le = LabelEncoder()
y_train = le.fit_transform(y_train)
y_test  = le.transform(y_test)
```

> `y_decoded = le.inverse_transform(y_encoded)`: 디코딩

### 5-3. 스케일링

공부시간(1~10)과 출석률(0~100)은 단위가 다릅니다. 이 차이가 모델을 망칩니다.

```python
from sklearn.preprocessing import StandardScaler, MinMaxScaler

# 표준화: 평균=0, 표준편차=1로 변환 (단위 제거)
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled  = scaler.transform(X_test)

# 정규화: 0~1 사이로 변환
scaler = MinMaxScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled  = scaler.transform(X_test)
```

> 💡 스케일링이 **필수**인 알고리즘: KNN, SVM, Logistic Regression  
> 스케일링이 **불필요**한 알고리즘: Decision Tree, Random Forest (트리 기반)

### 5-4. 파이프라인 — 컬럼 별 전처리 + 모델을 하나로 묶기

```python
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.linear_model import LogisticRegression

preprocessor = ColumnTransformer(
    transformers = [
        ("num", StandardScaler(), ["speed","temperature"]),
        ("cat", OneHotEncoder(), ["mode"])
    ]
)

pipeline = Pipeline([
    ("preprocessor", preprocessor),
    ("model", LogisticRegression())
])

pipeline.fit(X_train, y_train)          # 전처리 + 학습 한 번에
y_pred = pipeline.predict(X_test)       # 전처리 + 예측 한 번에
```

---

## CHAPTER 6. 모델 선택 & 학습

### 6-1. Scikit-Learn의 공통 인터페이스

Scikit-Learn의 모든 모델은 사용법이 동일합니다.

```python
model = 모델명()                # 1. 모델 객체 생성
model.fit(X_train, y_train)    # 2. 학습
y_pred = model.predict(X_test) # 3. 예측
```

#### model.fit(X, y) 내부 동작

```text
1. w, b 초기화
2. 예측값 계산
3. 손실 계산
4. 손실이 줄어드는 방향으로 w, b 수정
5. 반복
```

- w = 가중치(weight)
- b = 편향(bias)

---

### 6-2. 분류 알고리즘

#### 기본 명령어

```python
# 학습된 클래스 확인
model.classes_

# 예측 클래스 2차원 배열 반환
model.predict(X_test)

# 예측 클래스별 확률 확인
model.predict_proba(X_test)
```

#### Logistic Regression — 확률로 분류

입력을 0~1 사이 확률로 변환한 뒤 분류합니다.

```
입력값 → 선형 계산 → 시그모이드 함수 → 확률(0~1) → 0.5(threshold) 기준 분류
```

```python
from sklearn.linear_model import LogisticRegression

model = LogisticRegression()
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
```

| 장점 | 단점 |
|------|------|
| 빠르고 단순 | 비선형 데이터에 약함 |
| 결과 해석이 쉬움 | 스케일링 필요 |

---

#### Decision Tree — 스무고개로 분류

```
출석률 > 70% ?
 ├─ Yes → 과제 제출했나?
 │      ├─ Yes → 합격 ✅
 │      └─ No  → 불합격 ❌
 └─ No → 불합격 ❌
```

```python
from sklearn.tree import DecisionTreeClassifier, export_text

model = DecisionTreeClassifier(max_depth=3)   # 깊이 제한으로 과적합 방지
model.fit(X_train, y_train)

# 트리 구조 출력
print(export_text(model, feature_names=list(X.columns)))
```

| 장점 | 단점 |
|------|------|
| 결과를 눈으로 볼 수 있음 | 과적합되기 매우 쉬움 |
| 스케일링 불필요 | 데이터가 조금 바뀌면 구조 전체가 바뀜 |

---

#### Random Forest — 나무를 여러 개 모아서 투표

Decision Tree 여러 개를 만들고, 다수결로 최종 예측합니다. Decision Tree의 과적합 문제를 해결합니다.

```
Tree 1 → 합격
Tree 2 → 합격
Tree 3 → 불합격
Tree 4 → 합격
      ↓
다수결 → 합격 ✅
```

```python
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(
    n_estimators=100,       # 나무 개수 (기본값: 100)
    random_state=42
)
model.fit(X_train, y_train)

# 특성 중요도 - 각 특징이 예측에 얼마나 기여했는지 [트리 기반 모델에서만 사용 가능]
print(model.feature_importances_)
```

| 장점 | 단점 |
|------|------|
| 높은 성능 | 느림 |
| 과적합 감소 | 해석 어려움 |
| feature 중요도 제공 | 모델이 복잡함 |

---

#### 기타 분류 알고리즘 요약

| 알고리즘 | 핵심 아이디어 | 언제 쓰나 |
|---------|------------|---------|
| KNeighborsClassifier | 가장 가까운 k개 이웃 다수결 | 데이터가 적을 때 |
| SVC | 데이터를 나누는 최적의 경계선 | 고차원, 소규모 데이터 |
| GaussianNB | 확률 계산으로 분류 | 빠른 베이스라인 필요 시 |
| GradientBoosting | 이전 오차를 다음 트리가 보완 | 높은 성능이 필요할 때 |

---

### 6-3. 회귀 알고리즘

#### Linear Regression

입력(X)과 출력(y)의 관계를 직선(선형식)으로 모델링하여 숫자를 예측

```
예측값 = w₁·공부시간 + w₂·출석률 + b
```

```python
from sklearn.linear_model import LinearRegression

model = LinearRegression()
model.fit(X_train, y_train)

print("계수(기울기):", model.coef_)
print("절편:", model.intercept_)
```

#### Ridge / Lasso — 과적합 방지 선형 회귀

Linear Regression에 **규제(Regularization)**를 추가한 버전입니다.

```python
from sklearn.linear_model import Ridge, Lasso

# Ridge: 모든 feature의 계수를 줄임 (L2 규제)
ridge = Ridge(alpha=1.0)

# Lasso: 불필요한 feature 계수를 0으로 만듦 (L1 규제, Feature Selection 효과)
lasso = Lasso(alpha=1.0)
```

#### Polynomial Regression

선형 회귀에 다항식을 결합한 모델

```python
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression

# x → [x, x²] 변환 / degree: 다항식 차수
poly = PolynomialFeatures(degree=2)
X_poly = poly.fit_transform(X)

# y = b0 + b1·x + b2·x²
model.fit(X_poly, y)
```

| 장점 | 단점 |
|------|------|
| 비선형(곡선) 관계 표현 가능 | 차수(degree)가 너무 크면 과적합 발생 |

#### 기타 회귀 알고리즘 요약

| 알고리즘 | 핵심 아이디어 | 언제 쓰나 |
|---------|------------|---------|
| Decision Tree Regressor | 트리 구조로 숫자 예측 | 비선형 데이터 |
| Random Forest Regressor | 여러 트리의 평균 | 안정적인 성능 필요 시 |
| Gradient Boosting Regressor | 오차 순차 보완 | 높은 성능 필요 시 |
| KNeighborsRegressor | 가까운 k개 평균 | 단순한 베이스라인 |

---

## CHAPTER 7. 평가

### 7-1. 분류 평가

#### Confusion Matrix

```
           예측: 합격  예측: 불합격
실제: 합격     TP(50)      FN(10)
실제: 불합격    FP(5)      TN(35)
```

- **TP(True Positive)**: 실제 긍정 → 긍정 예측
- **TN(True Negative)**: 실제 부정 → 부정 예측
- **FP(False Positive)**: 실제 부정 → 긍정 예측 (오탐, Type 1 Error)
- **FN(False Negative)**: 실제 긍정 → 부정 예측 (미탐, Type 2 Error)

#### 4가지 핵심 지표

| 지표 | 공식 | 의미 | 중요한 상황 |
|------|------|------|-----------|
| **Accuracy** | (TP+TN) / 전체 | 전체 정답률 | 클래스 균형 시 |
| **Precision** | TP / (TP+FP) | 양성 예측의 정확도 | 오탐이 비싼 경우 (스팸 필터) |
| **Recall** | TP / (TP+FN) | 실제 양성 탐지율 | 미탐이 위험한 경우 (암 진단) |
| **F1-Score** | 2 × P × R / (P+R) | Precision과 Recall의 균형 | 클래스 불균형 시 |

```python
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score,
    f1_score, classification_report, confusion_matrix
)

y_pred = model.predict(X_test)

# Confusion Matrix
cm = confusion_matrix(y_test, y_pred)
tn, fp, fn, tp = cm.ravel()

print("Accuracy :", accuracy_score(y_test, y_pred))
print("Precision:", precision_score(y_test, y_pred))
print("Recall   :", recall_score(y_test, y_pred))
print("F1-Score :", f1_score(y_test, y_pred))
print()
print(classification_report(y_test, y_pred))  # 한 번에 모두 출력
```

> 💡 **데이터 불균형** (예: 정상 99%, 이상 1%)이면 Accuracy는 쓸모없습니다.  
> 이럴 땐 F1-Score를 쓰세요.

#### 여러 임계값(Threshold) 기반 평가지표

확률값에 대한 임계값(Threshold)을 변화시키면서 모델의 성능을 평가하는 지표들이다.

| 지표 | 의미 | 축 / 구성 요소 | 중요한 상황 |
|--------|------|---------------|-------------|
| **ROC Curve** | 다양한 임계값에서 모델의 성능 변화를 시각화한 그래프 | X축: FPR = FP / (FP + TN)<br>Y축: TPR = TP / (TP + FN) | 임계값에 따른 성능 변화 확인 |
| **ROC-AUC** | ROC Curve 아래 면적(AUC)으로, 양성과 음성을 얼마나 잘 구분하는지를 나타내는 지표 | AUC 값 (0~1) | 전체적인 분류 성능 비교 |
| **Precision-Recall Curve** | 임계값 변화에 따른 Precision과 Recall의 관계를 시각화한 그래프 | X축: Recall<br>Y축: Precision | 클래스 불균형 데이터 |
| **Average Precision (AP)** | Precision-Recall Curve의 전체 성능을 하나의 값으로 요약한 지표 | Precision-Recall Curve 아래 면적 | 불균형 데이터에서 모델 비교 |

```python
from sklearn.metrics import (
    roc_curve,
    roc_auc_score,
    precision_recall_curve,
    average_precision_score
)

# 클래스 예측
y_pred = model.predict(X_test)

# 클래스 1일 확률값 추출
y_prob = model.predict_proba(X_test)[:, 1]

# ROC-AUC - 확률값을 입력으로 사용
roc_auc = roc_auc_score(y_test, y_prob)
print("ROC-AUC:", roc_auc)

# ROC Curve
fpr, tpr, roc_thresholds = roc_curve(y_test, y_prob)

print("FPR:", fpr)
print("TPR:", tpr)
print("Thresholds:", roc_thresholds)

# Precision-Recall Curve - 확률값을 입력으로 사용
precision, recall, pr_thresholds = precision_recall_curve(
    y_test,
    y_prob
)

print("Precision:", precision)
print("Recall:", recall)
print("Thresholds:", pr_thresholds)

# Average Precision (AP)
ap = average_precision_score(y_test, y_prob)
print("Average Precision:", ap)
```

---

### 7-2. 회귀 평가

| 지표 | 공식 | 특징 |
|------|------|------|
| **MAE** | (1/n) * Σ|실제값 - 예측값| | 단위가 원래 데이터와 동일, 직관적 |
| **MSE** | (1/n) * Σ(실제값-예측값)² | 큰 오차에 더 민감 |
| **RMSE** | √MSE | MSE를 원래 단위로 환원, 가장 많이 씀 |
| **R² Score** | 0~1, 1에 가까울수록 좋음 | 모델이 얼마나 데이터를 설명하는지 |

```python
from sklearn.metrics import mean_absolute_error, mean_squared_error, root_mean_squared_error, r2_score

y_pred = model.predict(X_test)

print("MAE :", mean_absolute_error(y_test, y_pred))
print("RMSE:", root_mean_squared_error(y_test, y_pred))
print("R²  :", r2_score(y_test, y_pred))
```

---

## CHAPTER 8. 과적합과 일반화

**일반화**: 학습하지 않은 새로운 데이터에 대해서도 좋은 성능을 내는 능력

### 8-1. 과적합이란?

모델이 학습 데이터를 지나치게 외운 상태

```
시험 공부 대신 작년 시험지를 외운 학생
→ 작년 시험: 100점 / 올해 새 시험: 30점
```

```
Train Accuracy: 99%  ← 너무 좋음
Test Accuracy : 65%  ← 실제 성능
```

**원인과 해결책**:

| 원인 | 해결책 |
|------|--------|
| 데이터가 너무 적음 | 데이터 수집, 데이터 증강(SMOTE) |
| 모델이 너무 복잡함 | 모델 단순화 |
| Feature가 너무 많음 | Feature Selection (Lasso 등) |
| 노이즈까지 학습 | 정규화(Regularization) |

---

### 8-2. 교차검증 (Cross Validation)

학습 데이터를 여러 번 나눠서 학습과 평가를 반복

```
5-Fold 교차검증:

fold1: [2 3 4 5]로 학습, [1]로 평가
fold2: [1 3 4 5]로 학습, [2]로 평가
fold3: [1 2 4 5]로 학습, [3]로 평가
fold4: [1 2 3 5]로 학습, [4]로 평가
fold5: [1 2 3 4]로 학습, [5]로 평가

→ 5번의 성능을 평균냄
```

```python
from sklearn.model_selection import cross_val_score

scores = cross_val_score(model, X, y, cv=5, scoring="accuracy") # cv: 폴드 수 (기본값: 5)

print("각 폴드 점수:", scores)
print("평균 점수   :", scores.mean())
print("표준편차    :", scores.std())   # 작을수록 안정적
```

---

### 8-3. 하이퍼파라미터 튜닝 (GridSearchCV)

**하이퍼파라미터**: 사람이 직접 정해주는 설정값 (max_depth, n_estimators 등)

```python
from sklearn.model_selection import GridSearchCV
from sklearn.ensemble import RandomForestClassifier

param_grid = {
    "n_estimators": [50, 100, 200],
    "max_depth"   : [3, 5, None]
}
# 위 조합: 3 × 3 = 9가지, cv=5면 총 45번 학습

# GridSearchCV: 여러 하이퍼파라미터 조합을 자동으로 비교
grid = GridSearchCV(
    RandomForestClassifier(random_state=42),
    param_grid=param_grid,
    scoring="f1",   # 평가 기준
    cv=5,
    n_jobs=-1       # 병렬 처리할 CPU 코어 개수 (-1: 모든 코어 사용)
)

# RandomizedSearchCV: 조합의 일부를 랜덤하게 선택해서 테스트
random_search = RandomizedSearchCV(
    estimator=RandomForestRegressor(random_state=42),
    param_distributions=param_distributions,
    n_iter=10, # 탐색할 조합의 개수
    cv=5,
    scoring="neg_mean_absolute_error", # MAE는 낮을수록 좋음
    random_state=42,
    n_jobs=-1
)

grid.fit(X_train, y_train)

print("최적 파라미터:", grid.best_params_)
print("최적 점수    :", grid.best_score_)

best_model = grid.best_estimator_
y_pred = best_model.predict(X_test)
```

---

## CHAPTER 9. 모델 저장과 재사용

```python
import joblib

# 저장
joblib.dump(best_model, "my_model.pkl")
print("모델 저장 완료!")

# 불러오기 (다음에 새로 열 때)
loaded_model = joblib.load("my_model.pkl")
y_pred = loaded_model.predict(X_test)
```

---

## CHAPTER 10. 실전 코드 — 처음부터 끝까지 한 번에

아래 코드는 지금까지 배운 모든 내용을 합친 **완전한 파이프라인**입니다.

```python
import pandas as pd
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split, GridSearchCV
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestClassifier
from sklearn.pipeline import Pipeline
from sklearn.metrics import classification_report
import joblib

# 데이터 준비
X, y = make_classification(
    n_samples=1000, n_features=8, n_classes=2, random_state=42
)

# 데이터 분리
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# 컬럼별 전처리
preprocessor = ColumnTransformer(
    transformers=[
        ("num", StandardScaler(), ["speed", "temperature"]),
        ("cat", OneHotEncoder(), ["mode"])
    ]
)

# 파이프라인 (전처리 + 모델)
pipeline = Pipeline([
    ("preprocessor", preprocessor),
    ("model", RandomForestClassifier(random_state=42))
])

# 하이퍼파라미터 튜닝 - 파이프라인 내부 파라미터 → 단계명__파라미터명
param_grid = {
    "model__n_estimators": [50, 100, 200],
    "model__max_depth"   : [3, 5, None]
}

grid = GridSearchCV(pipeline, param_grid, cv=5, scoring="f1", n_jobs=-1)
grid.fit(X_train, y_train)

# 최적 모델로 예측 & 평가
best_model = grid.best_estimator_
y_pred = best_model.predict(X_test)

print("최적 파라미터:", grid.best_params_)
print()
print(classification_report(y_test, y_pred))

# 모델 저장
joblib.dump(best_model, "best_model.pkl")
print("모델 저장 완료!")
```

---

## 마무리 — 알고리즘 선택 가이드

```
문제 파악
    ↓
분류? → Logistic Regression으로 시작
       → 성능 부족 → Random Forest
       → 더 필요 → Gradient Boosting

회귀? → Linear Regression으로 시작
       → 과적합 → Ridge / Lasso
       → 비선형 → Random Forest Regressor

데이터 부족? → KNN, SVM, GaussianNB
빠른 베이스라인? → Logistic Regression, GaussianNB
최고 성능 필요? → Gradient Boosting, XGBoost
```

---