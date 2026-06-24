# 비지도학습(Unsupervised Learning)

## 0. 들어가며 — 왜 비지도학습을 배워야 할까?

정답(label)이 없는 데이터 안에서 숨겨진 패턴, 구조, 그룹, 관계를 찾아내는 방법

| 구분 | 지도학습 | 비지도학습 |
|---|---|---|
| 데이터 | X + y (정답 있음) | X만 있음 (정답 없음) |
| 목표 | 정답을 예측 | 숨겨진 구조/패턴 발견 |
| 예시 | 스팸 메일 분류, 집값 예측 | 고객 그룹화, 이상 거래 탐지 |
| 평가 | 정답과 비교해서 정확도 측정 | "그럴듯한가"를 사람이 해석 |

> 💡 실무에서는 "정답 데이터가 없어서 일단 비지도학습으로 그룹을 나눠보고, 그 그룹에 사람이 의미를 붙인 뒤, 나중에 지도학습으로 전환"하는 흐름이 정말 많습니다. 비지도학습은 데이터를 처음 들여다볼 때 쓰는 "탐색 도구"이기도 합니다.

---

## 1. 비지도학습의 4가지 문제 유형

```text
비지도학습
 ├─ 군집화 (Clustering)                       → "비슷한 것끼리 묶기"
 ├─ 차원 축소 (Dimensionality Reduction)      → "정보는 유지하고 압축하기"
 ├─ 이상치 탐지 (Anomaly Detection)           → "다른 것 찾아내기"
 └─ 연관 규칙 학습 (Association Rule Learning) → "같이 다니는 패턴 찾기"
```

### 1-1. 군집화 (Clustering)

유사한 데이터끼리 그룹으로 묶는 방법

**예시**
- 고객 그룹 분류
- 비슷한 색/특징 픽셀끼리 그룹화

> 💡 비유: 마트에서 장바구니 데이터를 보고 "이 사람들은 비슷한 쇼핑 패턴을 가지고 있구나"라고 무리를 짓는 것. 누가 어떤 그룹인지 미리 정해주지 않아도, 데이터의 거리/밀도만 보고 자동으로 그룹을 나눕니다.

### 1-2. 차원 축소 (Dimensionality Reduction)

데이터의 정보는 유지하면서 특징(feature)을 줄여서 표현하는 방법

**예시**
- 이미지 압축
- 고차원 데이터를 2차원으로 시각화
- 중요하지 않은 특징 제거

> 💡 비유: 사람을 설명할 때 "키, 몸무게, 나이, 혈압, 시력, ... " 100개 항목 대신 "체형 점수 1개, 건강 점수 1개"처럼 핵심 축 2개로 압축하는 것. 정보는 최대한 보존하면서 차원(컬럼 수)만 줄입니다.

### 1-3. 이상치 탐지 (Anomaly Detection)

정상 데이터와 다른 패턴을 가진 데이터를 찾아내는 방법

**예시**
- 신용카드 사기 탐지
- 서버 이상 트래픽 탐지
- 제조 불량품 검출

> 💡 비유: 100명이 줄을 서 있는데 혼자 거꾸로 서 있는 사람을 찾는 것. "정상이 뭔지"는 정의하지 않아도, "다수와 동떨어진 패턴"을 통계적으로 잡아냅니다.

### 1-4. 연관 규칙 학습 (Association Rule Learning)

데이터 항목 간의 흥미로운 관계나 규칙을 찾는 방법

**예시**
- 장바구니 분석: "기저귀를 산 사람이 맥주도 함께 산다"
- 웹사이트 추천: "이 페이지를 본 사람이 저 페이지도 본다"
- 추천 시스템: "이 상품을 구매한 사람이 함께 구매한 상품"

> 💡 비유: 사람이 직접 생각하지 못했던 항목 간의 동행 패턴을 데이터에서 자동으로 찾아냅니다.

---

## 2. 실습 준비 — 데이터 만들고 전처리하기

### 2-1. 가짜 데이터 생성

```python
from sklearn.datasets import make_blobs

# 군집화용 가짜 데이터
X, _ = make_blobs(
    n_samples=100,      # 데이터 개수
    n_features=2,       # 특징 개수
    centers=3,          # 클러스터 개수
    cluster_std=1.0,    # 클러스터 표준편차
    random_state=42
)
```

### 2-2. 스케일링 (Scaling)

비지도학습 알고리즘은 대부분 **거리(distance)** 를 기준으로 동작합니다. 그런데 "나이(20~60)"와 "연간 소비액(100~5000)"처럼 단위가 다른 값들을 그대로 쓰면, 값이 큰 컬럼이 거리 계산을 지배해버립니다. 그래서 항상 스케일링을 먼저 합니다.

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
```

> ⚠️ **초보자가 가장 많이 하는 실수**: 스케일링 없이 바로 K-Means를 돌리는 것. 단위가 큰 컬럼 하나가 군집 결과를 좌우해버려서, 의도와 다른 그룹이 만들어집니다.

---

## 3. 차원 축소 — PCA

PCA(주성분분석, Principal Component Analysis)는 데이터를 잘 설명하는 "축"을 새로 찾아서, 그 축 위로 데이터를 투영(projection)하는 방법입니다. 여러 컬럼을 몇 개의 핵심 축으로 압축한다고 생각하면 됩니다.

```python
from sklearn.decomposition import PCA

pca = PCA(n_components=2) # 2차원으로 축소 (기본값: None)
X_pca = pca.fit_transform(X_scaled)
```

> 💡 PCA는 보통 두 가지 목적으로 씁니다.
> 1. **시각화**: 컬럼이 6개, 10개라서 눈으로 볼 수 없을 때 2차원으로 줄여서 그래프로 확인
> 2. **전처리**: 군집화 전에 노이즈가 적고 핵심 정보만 남은 축으로 줄여서 알고리즘 성능을 높임

---

## 4. 군집화 알고리즘 3종 비교

### 4-1. K-Means (K-평균)

데이터를 K개의 그룹으로 나누되, 각 클러스터 중심(centroid)과의 거리를 최소화하도록 묶는 알고리즘

**동작 원리**

```text
1. k개의 중심점(centroid)을 임의로 선택
2. 각 데이터를 가장 가까운 중심점에 할당
3. 각 클러스터의 평균으로 중심점 재계산
4. 중심점이 더 이상 변하지 않을 때까지 반복
```

```python
from sklearn.cluster import KMeans

# K-Means 모델
kmeans = KMeans(n_clusters=3) # 클러스터 개수 (기본값: 8)
kmeans.fit(X)

# 결과
labels = kmeans.labels_ # 각 데이터가 속한 클러스터 라벨
centers = kmeans.cluster_centers_ # 각 클러스터의 중심점 좌표
```

**장점**
- 빠름 (대용량 가능)
- 구현 쉬움
- 직관적

**단점**
- K를 미리 지정해야 함
- 데이터가 원형이 아닐 때 성능 저하
- 이상치에 민감
- 스케일 영향 매우 큼

#### K 정하는 방법 — Elbow Method (엘보우 방법)

K를 늘리면서 "오차(Inertia) 감소"를 보는 방법입니다. K가 커질수록 오차는 계속 줄어들지만, 어느 순간부터는 줄어드는 속도가 확 느려집니다. 그 꺾이는 지점(팔꿈치 모양)을 적절한 K로 선택합니다.

```python
inertia = []

for k in range(1, 10):
    kmeans = KMeans(n_clusters=k)
    kmeans.fit(X)
    inertia.append(kmeans.inertia_) # 각 데이터와 자신이 속한 클러스터 중심점 사이의 거리 제곱의 합

plt.plot(range(1, 10), inertia, marker='o')
plt.xlabel("K")
plt.ylabel("Inertia")
plt.show()
```

### 4-2. DBSCAN

밀도 기반 군집화 알고리즘입니다. 밀도가 높은 곳은 군집, 희박한 곳은 이상치로 판단합니다.

**동작 원리**

```text
1. 특정 반경(eps) 내에 최소 데이터 개수(min_samples)가 있으면 클러스터 형성
2. 경계점(border point)은 인접 클러스터에 포함
3. 그 외는 이상치(noise)로 분류
```

```python
from sklearn.cluster import DBSCAN

# eps: 얼마나 가까워야 이웃인가, min_samples: 몇 명 이상 모여야 군집인가
dbscan = DBSCAN(eps=0.1, min_samples=5)
labels = dbscan.fit_predict(X)
```

**장점**
- K를 지정할 필요 없음
- 이상치 자동 탐지
- 다양한 모양의 군집 가능

**단점**
- eps와 min_samples를 지정해야 함
- 고차원 데이터에서 성능 저하
- 밀도가 다른 데이터에서는 성능 저하

### 4-3. Hierarchical Clustering (계층적 군집)

데이터를 단계적으로 합치거나 나누면서 군집을 생성합니다.

1. **Agglomerative**: 각 데이터를 하나의 클러스터로 시작 → 가까운 클러스터끼리 계속 합침
2. **Divisive**: 전체 데이터를 하나의 클러스터로 시작 → 점점 나눔

> 💡 결과를 그리면 **덴드로그램(Dendrogram)** 이라는 나무 모양 그림이 나오는데, 어느 높이에서 "자를지"를 정해서 군집 개수를 나중에 결정할 수 있습니다.

```python
from sklearn.cluster import AgglomerativeClustering

# n_clusters: 최종 군집 개수, linkage: 합치는 기준
agg = AgglomerativeClustering(n_clusters=3, linkage="ward")
labels = agg.fit_predict(X)
```

**장점**
- 군집 개수를 나중에 결정 가능
- 데이터 구조 이해에 좋음

**단점**
- 느림
- 노이즈에 민감

### 4-4. 세 알고리즘 한눈에 비교

| 알고리즘 | K 지정 필요? | 군집 모양 | 이상치 처리 | 속도 | 추천 상황 |
|---|---|---|---|---|---|
| K-Means | 필요 | 원형/구형 위주 | 약함 (민감) | 빠름 | 데이터 양이 많고 군집이 둥글게 분포할 때 |
| DBSCAN | 불필요 | 자유로운 모양 | 강함 (자동 탐지) | 보통 | 이상치 탐지가 같이 필요할 때 |
| Hierarchical | 불필요 (나중에 결정) | 자유로운 모양 | 약함 | 느림 | 데이터 양이 적고 구조를 해석하고 싶을 때 |

---

## 5. 실전 미니 프로젝트 — 고객 세분화

```text
데이터 → 전처리 → 스케일링 → PCA → K-Means → 그룹 해석
```

### 5-1. 작은 예제로 흐름 익히기

```python
import pandas as pd
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA
from sklearn.cluster import KMeans

# 샘플 데이터
data = {
    "age": [25, 45, 23, 35, 52, 23, 40],
    "spending": [200, 800, 150, 400, 1000, 120, 700],
    "visit": [10, 30, 5, 15, 40, 3, 25]
}

df = pd.DataFrame(data)

# 스케일링
scaler = StandardScaler()
X_scaled = scaler.fit_transform(df)

# PCA
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X_scaled)

# K-Means
kmeans = KMeans(n_clusters=3)
df["cluster"] = kmeans.fit_predict(X_pca)

print(df)
```

### 5-2. 실전 규모로 확장하기 (300명 고객 데이터)

이번에는 더 많은 특징(feature)과 더 많은 고객 수로 실전과 비슷한 파이프라인을 구성합니다.

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

from sklearn.preprocessing import StandardScaler
from sklearn.cluster import KMeans
from sklearn.decomposition import PCA

# 1. 데이터 생성
np.random.seed(42)
n = 300

df = pd.DataFrame({
    "customer_id": range(1, n + 1),
    "age": np.random.randint(20, 60, n),
    "annual_spending": np.random.randint(100, 5000, n),
    "visit_count": np.random.randint(1, 50, n),
    "days_since_last_purchase": np.random.randint(1, 365, n),
    "avg_basket_amount": np.random.randint(10, 300, n),
    "category_count": np.random.randint(1, 10, n)
})

# 2. feature 선택
features = [
    "age",
    "annual_spending",
    "visit_count",
    "days_since_last_purchase",
    "avg_basket_amount",
    "category_count"
]

X = df[features]

# 3. 스케일링
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# 4. Elbow Method
inertia = []
for k in range(1, 11):
    model = KMeans(n_clusters=k, random_state=42, n_init=10)
    model.fit(X_scaled)
    inertia.append(model.inertia_)

plt.figure(figsize=(8, 5))
plt.plot(range(1, 11), inertia, marker='o')
plt.xlabel("K")
plt.ylabel("Inertia")
plt.title("Elbow Method")
plt.show()

# 5. K-Means 적용
kmeans = KMeans(n_clusters=4, random_state=42, n_init=10)
df["cluster"] = kmeans.fit_predict(X_scaled)

# 6. 군집 요약
cluster_summary = df.groupby("cluster")[features].mean()
print(cluster_summary)

# 7. PCA 시각화
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X_scaled)

plt.figure(figsize=(8, 6))
plt.scatter(X_pca[:, 0], X_pca[:, 1], c=df["cluster"])
plt.xlabel("PCA 1")
plt.ylabel("PCA 2")
plt.title("Customer Segmentation")
plt.show()
```

> 💡 **이 파이프라인에서 기억할 것**: K-Means 자체는 6개 feature를 그대로 써서 학습시키고, PCA는 "결과를 2차원 그림으로 보여주기 위한 시각화 용도"로만 따로 사용했습니다. PCA로 줄인 데이터로 군집화를 할 수도 있지만(5-1 예제처럼), 이 예제처럼 원본 스케일 데이터로 군집화하고 PCA는 시각화에만 쓰는 방식도 실무에서 흔합니다.

> 6번 단계의 `cluster_summary`를 보면 "0번 그룹은 나이가 많고 소비액이 큰 VIP 그룹", "2번 그룹은 방문은 적지만 객단가가 높은 그룹"처럼, **숫자 그룹에 사람이 의미를 붙이는 해석 단계**가 비지도학습의 마지막 핵심 단계입니다. 모델은 그룹만 나눠줄 뿐, "이게 무슨 의미인지"는 사람이 도메인 지식으로 채워야 합니다.

---

## 6. 이상치 탐지 — Isolation Forest

Isolation Forest는 "이상치는 정상 데이터보다 더 쉽게 고립(분리)된다"는 아이디어를 이용합니다. 트리를 랜덤하게 여러 번 쪼개봤을 때, 이상치는 적은 횟수만에 혼자 떨어져 나옵니다.

```python
from sklearn.ensemble import IsolationForest

model = IsolationForest(contamination=0.1)
df["anomaly"] = model.fit_predict(X_scaled)

print(df) # 1: 정상, -1: 이상치
```

> 💡 `contamination=0.1`은 "전체 데이터 중 약 10%는 이상치일 것"이라는 사전 가정입니다. 실제 이상치 비율을 모를 때는 도메인 지식이나 과거 통계를 참고해서 조정합니다.

---

## 7. 군집화로 추천 시스템 맛보기

연관 규칙 학습의 한 응용으로, 사용자 행동 데이터를 군집화해서 "같은 그룹 안에서 서로 추천"하는 방식이 있습니다.

```text
행동 데이터 → 벡터화 → 군집화 → 같은 그룹 추천
```

> **임베딩(Embedding)**: 데이터(텍스트, 이미지 등)를 숫자 벡터로 바꿔서 의미를 담아 표현하는 것

```python
user_data = [
    [1, 0, 1],  # A 사용자
    [1, 1, 0],  # B 사용자
    [0, 1, 1],  # C 사용자
]

kmeans = KMeans(n_clusters=2)
labels = kmeans.fit_predict(user_data)

print(labels)
```

> 💡 A, C 사용자처럼 "행동 패턴(어떤 상품을 봤는지/샀는지)"이 비슷하면 같은 군집으로 묶이고, 같은 군집에 속한 다른 사용자가 산 상품을 서로에게 추천하는 식으로 활용합니다. 이것이 협업 필터링(Collaborative Filtering) 추천 시스템의 아주 단순화된 원리입니다.

---

## 8. 전체 정리

| 목적 | 알고리즘 | 핵심 한 줄 |
|---|---|---|
| 군집화 | K-Means | 중심점과의 거리를 최소화하며 K개로 묶기 |
| 군집화 | DBSCAN | 밀도가 높은 곳을 군집, 낮은 곳을 이상치로 |
| 군집화 | Hierarchical | 가까운 것부터 단계적으로 합치기 |
| 차원 축소 | PCA | 정보는 유지, 핵심 축으로 압축 |
| 이상치 탐지 | Isolation Forest | 쉽게 고립되는 데이터 = 이상치 |
| 전처리(공통) | StandardScaler | 단위 차이로 거리 계산이 왜곡되는 것 방지 |

**전형적인 파이프라인**

```text
데이터 수집 → 결측치/이상값 처리 → 스케일링 → (선택) PCA → 군집화/이상탐지 → 결과 해석
```

---