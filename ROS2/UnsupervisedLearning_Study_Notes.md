# Unsupervised Learning

정답(label)이 없는 데이터 안에서 숨겨진 패턴, 구조, 그룹, 관계를 찾아내는 방법

---

## 문제 유형

### 군집화 (Clustering)

유사한 데이터끼리 그룹으로 묶는 방법

예시:

- 고객 그룹 분류
- 비슷한 색/특징 픽셀끼리 그룹화

### 차원 축소 (Dimensionality Reduction)

데이터의 정보는 유지하면서 특징(feature)을 줄여서 표 현하는 방법

예시:

- 이미지 압축
- 고차원 데이터를 2차원으로 시각화
- 중요하지 않은 특징 제거

### 이상치 탐지 (Anomaly Detection)

정상 데이터와 다른 패턴을 가진 데이터를 찾아내는 방법

예시:

- 신용카드 사기 탐지
- 서버 이상 트래픽 탐지
- 제조 불량품 검출

### 연관 규칙 학습 (Association Rule Learning)

데이터 항목 간의 흥미로운 관계나 규칙을 찾는 방법

예시:

- 장바구니 분석: "기저귀를 산 사람이 맥주도 함께 산다"
- 웹사이트 추천: "이 페이지를 본 사람이 저 페이지도 본다"
- 추천 시스템: "이 상품을 구매한 사람이 함께 구매한 상품"

---

## 데이터

### 생성

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

### 전처리

#### 스케일링

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
```

---

## PCA (차원축소)

```python
from sklearn.decomposition import PCA

pca = PCA(n_components=2) # 2차원으로 축소 (기본값: None)
X_pca = pca.fit_transform(X_scaled)
```

---

## 군집화

### K-Means (K-평균)

데이터를 K개의 그룹으로 나누되, 각 클러스터 중심(centroid)과의 거리를 최소화하도록 묶는 알고리즘

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

장점

- 빠름 (대용량 가능)
- 구현 쉬움
- 직관적

단점

- K를 미리 지정해야 함
- 데이터가 원형이 아닐 때 성능 저하
- 이상치에 민감
- 스케일 영향 매우 큼

#### K 정하는 방법

1. Elbow Method (엘보우 방법): K를 늘리면서 “오차 감소”를 보는 방법

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

### DBSCAN

밀도 기반 군집화 알고리즘.
밀도가 높은 곳은 군집, 희박한 곳은 이상치로 판단.

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

장점

- K를 지정할 필요 없음
- 이상치 자동 탐지
- 다양한 모양의 군집 가능

단점

- eps와 min_samples를 지정해야 함
- 고차원 데이터에서 성능 저하
- 밀도가 다른 데이터에서는 성능 저하

### Hierarchical Clustering (계층적 군집)

데이터를 단계적으로 합치거나 나누면서 군집 생성

1. Agglomerative: 각 데이터를 하나의 클러스터로 시작 → 가까운 클러스터끼리 계속 합침
2. Divisive: 전체 데이터를 하나의 클러스터로 시작 → 점점 나눔

```python
from sklearn.cluster import AgglomerativeClustering

# n_clusters: 최종 군집 개수, linkage: 합치는 기준
agg = AgglomerativeClustering(n_clusters=3, linkage="ward")
labels = agg.fit_predict(X)
```

결과: Dendrogram (덴드로그램, 트리 구조)

장점

- 군집 개수를 나중에 결정 가능
- 데이터 구조 이해에 좋음

단점

- 느림
- 노이즈에 민감

---

## 고객 세분화

```text
데이터 → 전처리 → 스케일링 → PCA → K-Means → 그룹 해석
```

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

## 이상탐지

### Isolation Forest

```python
from sklearn.ensemble import IsolationForest

model = IsolationForest(contamination=0.1)
df["anomaly"] = model.fit_predict(X_scaled)

print(df) # 1: 정상, -1: 이상치
```

## 추천 시스템 전처리

```text
행동 데이터 → 벡터화 → 군집화 → 같은 그룹 추천
```

> 임베딩(Embedding): 데이터(텍스트, 이미지 등)를 숫자 벡터로 바꿔서 의미를 담아 표현하는 것

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

---

## Python 기반 고객 세분화

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