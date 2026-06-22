# NumPy

## 1. 기본 배열 생성

```python
import numpy as np

# 리스트 → 배열
a = np.array([2, 3])
b = np.array([1.0, 2.0, 0.5])

print(a)  # [2 3]
print(b)  # [1.  2.  0.5]
```

---

## 2. 벡터 연산

### 2-1. 기본 연산

```python
import numpy as np

v1 = np.array([1, 2])
v2 = np.array([3, 4])

# 인덱스
print(v1[0])  # 1
print(v1[1])  # 2

# 덧셈
print(v1 + v2)  # [4 6]

# 뺄셈
print(v2 - v1)  # [2 2]

# 스칼라 곱셈
print(v1 * 2)   # [2 4]

# 원소별 곱셈 (element-wise)
print(v1 * v2)  # [3 8]
```

### 2-2. 내적 (Dot Product)

두 벡터가 얼마나 같은 방향인지 나타내는 값

```python
import numpy as np

v1 = np.array([1, 2])
v2 = np.array([3, 4])

# 두 가지 방법, 결과 동일
print(v1 @ v2)         # 11
print(np.dot(v1, v2))  # 11
```

### 2-3. 외적 (Cross Product)

두 벡터에 수직인 벡터 (또는 2D에서는 스칼라) 반환

```python
import numpy as np

v1 = np.array([1, 2])
v2 = np.array([3, 4])
v3 = np.array([1, 0, 0])
v4 = np.array([0, 1, 0])

# 2D: 스칼라 반환 (z 성분)
print(np.cross(v1, v2))  # -2

# 3D: 벡터 반환
print(np.cross(v3, v4))  # [0 0 1]
```

### 2-4. 크기와 정규화

```python
import numpy as np

v2 = np.array([3, 4])

# 크기 (L2 norm)
print(np.linalg.norm(v2))  # 5.0

# 정규화 — 크기를 1로 만든 단위벡터
print(v2 / np.linalg.norm(v2))  # [0.6 0.8]
```

---

## 3. 각도 변환

NumPy의 삼각함수는 항상 라디안을 사용한다.

```python
import numpy as np

# 라디안 상수
angle_rad = np.pi / 4  # 45도

# cos 역함수 (rad) — 입력값은 반드시 -1 ~ 1 사이여야 함
dot_value = 0.707
angle_rad = np.arccos(dot_value)

# rad → deg
angle_deg = np.rad2deg(angle_rad)
angle_deg = np.degrees(angle_rad)
print(angle_deg)  # 45.0

# deg → rad
angle_rad = np.deg2rad(90)
print(angle_rad)  # 1.5707963267948966
```

---

## 4. 값 제한 (Clamp)

값을 지정한 범위 안으로 제한한다.

```python
import numpy as np

# np.clip(value, min, max)
print(np.clip(3.5, -2.0, 2.0))   # 2.0
print(np.clip(-5.0, -2.0, 2.0))  # -2.0
print(np.clip(1.2, -2.0, 2.0))   # 1.2

# 배열에도 동일하게 적용
arr = np.array([-5.0, 1.2, 3.8])
print(np.clip(arr, -2.0, 2.0))  # [-2.  1.2  2. ]
```

---

## 5. 행렬 연산

### 5-1. 행렬 생성과 인덱싱

```python
import numpy as np

# 단위행렬
identity_matrix = np.eye(3)

# 행렬 생성
A = np.array([
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
])

# 인덱싱
print(A[1, 2])  # 6        — 2행 3열 단일 원소
print(A[1])     # [4 5 6]  — 2행 전체
print(A[:, 1])  # [2 5 8]  — 2열 전체

# 행렬 크기
print(A.shape)  # (3, 3)
```

### 5-2. 기본 연산

```python
import numpy as np

A = np.array([
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
])
vector = np.array([1, 1, 1])

# 행렬 덧셈
print(A + A)

# 행렬 뺄셈
print(A - A)

# 원소별 곱셈 (element-wise)
print(A * A)

# 행렬 곱셈
print(A @ A)

# 행렬-벡터 곱셈
print(np.eye(3) @ vector)  # [1. 1. 1.]
```

### 5-3. 전치, 역행렬, 수치 비교

```python
import numpy as np

A = np.array([
    [1.0, 2.0],
    [3.0, 4.0]
])

# 전치 (Transpose)
print(A.T)

# 역행렬
print(np.linalg.inv(A))

# 두 배열의 값이 거의 같은지 비교 — 소수점 계산 오차가 있을 때 유용
B = np.linalg.inv(A) @ A   # 단위행렬이 되어야 함
print(np.allclose(B, np.eye(2)))  # True
```

---

## 6. 난수

```python
# 정수 랜덤값
np.random.randint(low, high, size)

# 0 ~ 1 사이 실수 랜덤값
np.random.rand(size)

# 평균이 loc, 표준편차가 scale인 정규분포를 따르는 실수
np.random.normal(loc, scale, size)
```

---

**flatten()**

```python
import numpy as np

# 2차원 배열
arr = np.array([[1, 2, 3], [4, 5, 6]])

# 1차원으로 변환
flat_arr = arr.flatten()
print(flat_arr)  # [1 2 3 4 5 6]
```

**reshape()**

```python
import numpy as np

# 1차원 배열
arr = np.arange(6)  # [0 1 2 3 4 5]

# 2차원으로 변환 (-1: 자동으로 계산)
reshaped_arr = arr.reshape(2, -1)
print(reshaped_arr)
# [[0 1 2]
#  [3 4 5]]
```

**NaN**

`np.nan`