# OpenCV

## 0. 설치

```bash
pip install opencv-python

# 설치 확인
python -c "import cv2; print(cv2.__version__)"
```

---

## 1. 이미지 읽기/쓰기

OpenCV는 컬러 이미지를 **BGR** 순서로 다룬다. (일반적인 RGB와 채널 순서가 반대)

```python
import cv2

# 이미지 읽기 — NumPy 배열 반환, 실패 시 None 반환
img = cv2.imread(image_path)

# 그레이스케일로 읽기
img_gray = cv2.imread(image_path, cv2.IMREAD_GRAYSCALE)

# 이미지 표시
cv2.imshow("window", img)
cv2.waitKey(0)       # 0: 키 입력까지 무한 대기
cv2.waitKey(1000)    # 1000ms(1초) 대기 후 자동으로 넘어감
cv2.destroyAllWindows()

# 이미지 저장
cv2.imwrite(save_path, img)
```

---

## 2. 이미지 속성 확인

### 2-1. 좌표계

OpenCV의 좌표 원점은 이미지 **좌상단**이다.

```
(0,0) ───────────→ x (width)
  │
  │
  │
  ↓
  y (height)
```

### 2-2. shape / size / dtype

```python
import cv2

img = cv2.imread(image_path)

# shape — 배열 차원
# 컬러:       (height, width, channel)  예: (480, 640, 3)
# 그레이스케일: (height, width)          예: (480, 640)
print(img.shape)

# size — 전체 원소 수
# 컬러:       height × width × channel  예: 921600
# 그레이스케일: height × width           예: 307200
print(img.size)

# dtype — 픽셀 데이터 타입 (보통 uint8: 0~255)
print(img.dtype)

# 예시 — 이미지 크기 추출
height, width = img.shape[:2]
print(f"높이: {height}, 너비: {width}")
```

---

## 3. 픽셀 읽기/쓰기

인덱스는 `[y, x]` 순서이다. (행, 열 순서)

```python
import cv2

img = cv2.imread(image_path)

# 픽셀 읽기 (흑백은 밝기 값 1개, 컬러는 BGR 3개)
pixel = img[100, 200]   # y=100, x=200 위치의 픽셀
blue  = pixel[0]
green = pixel[1]
red   = pixel[2]

# 픽셀 쓰기 — 단일 픽셀
img[100, 200] = (0, 255, 0)           # 초록색으로 변경

# 픽셀 쓰기 — 사각형 영역 (슬라이싱)
img[50:150, 100:300] = (0, 0, 255)    # 사각형 영역을 빨간색으로 변경
```

---

## 4. 색상 변환

```python
import cv2

img = cv2.imread(image_path)

# BGR → RGB  (matplotlib 등 외부 라이브러리와 함께 쓸 때)
img_rgb  = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)

# BGR → 그레이스케일
img_gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

# BGR → HSV  (색상 검출에 사용)
img_hsv  = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
```

### 4-1. HSV 색상 공간

HSV는 색상(Hue), 채도(Saturation), 명도(Value)로 구성된 색상 공간이다.
HSV 이미지는 사람이 직접 보기 위한 것이 아니라, **색상 검출 계산**을 위해 사용한다.
조명 환경에 따라 같은 파란색도 HSV 값이 달라질 수 있다.

| 채널 | 범위 | 의미 |
|---|---|---|
| H | 0 ~ 179 | 색상 (빨강, 초록, 파랑 등) |
| S | 0 ~ 255 | 채도 (색의 진하기) |
| V | 0 ~ 255 | 명도 (색의 밝기) |

```python
import cv2
import numpy as np

img = cv2.imread(image_path)
hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)

# 특정 색상 범위 마스크 생성
# H: 100~130 (파란색 계열) / S: 100~255 / V: 100~255
lower_blue = np.array([100, 100, 100])
upper_blue = np.array([130, 255, 255])

# 범위 안의 픽셀은 255, 나머지는 0
mask   = cv2.inRange(hsv, lower_blue, upper_blue)

# 마스크 적용 — 파란색 영역만 남기고 나머지는 검정
result = cv2.bitwise_and(img, img, mask=mask)
```

---

## 5. 기본 전처리

### 5-1. 밝기·대비 조절

```
새 픽셀 값 = 원래 픽셀 값 × alpha + beta

alpha: 대비(contrast) 조절 — 1.0 기준, 높을수록 대비 강해짐
beta:  밝기(brightness) 조절 — 0 기준, 양수면 밝아짐
```

```python
import cv2

img = cv2.imread(image_path)

# alpha=1.0, beta=0 이면 원본과 동일
result = cv2.convertScaleAbs(img, alpha=1.5, beta=30)
```

### 5-2. 이미지 반전

```
새 픽셀 값 = 255 - 원래 픽셀 값
```

```python
import cv2

img = cv2.imread(image_path)

inverted = cv2.bitwise_not(img)
```

### 5-3. 이진화 (Thresholding)

픽셀 값을 기준으로 흑/백 두 가지로 분류한다.

```python
import cv2

img  = cv2.imread("image.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

# 기본 이진화
# THRESH_BINARY:     pixel > threshold → 255 / pixel <= threshold → 0
# THRESH_BINARY_INV: pixel > threshold → 0   / pixel <= threshold → 255
# ret: 실제로 사용된 임계값
ret, binary = cv2.threshold(gray, 127, 255, cv2.THRESH_BINARY)

# Adaptive Threshold — 픽셀마다 주변 영역을 보고 임계값을 다르게 적용
# blockSize: 픽셀을 판단할 주변 영역 크기 (홀수여야 함)
# C: 계산된 임계값에서 뺄 보정값
adaptive = cv2.adaptiveThreshold(
    gray, 255,
    cv2.ADAPTIVE_THRESH_GAUSSIAN_C,
    cv2.THRESH_BINARY,
    blockSize=11, C=2
)

# Otsu Threshold — 이미지 밝기 분포를 분석해 임계값을 자동으로 결정
# threshold 인자에 0을 넣고 THRESH_OTSU 플래그를 추가
ret, otsu = cv2.threshold(gray, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)
print(f"Otsu 임계값: {ret}")
```

| 방법 | 임계값 | 특징 |
|---|---|---|
| 기본 이진화 | 수동 지정 | 단순, 조명 변화에 취약 |
| Adaptive Threshold | 픽셀별 자동 계산 | 조명이 고르지 않은 이미지에 적합 |
| Otsu Threshold | 전체 이미지 기반 자동 계산 | 밝기가 두 그룹으로 뚜렷이 나뉠 때 적합 |

---

## 6. 이미지 크기 및 기하 변환

```python
import cv2

img = cv2.imread(image_path)

# 이미지 크기 변경 (image.shape와 순서 다름)
resized = cv2.resize(img, (new_width, new_height))

# 이미지 회전
# 1. 회전 행렬 생성
rotation_matrix = cv2.getRotationMatrix2D(
    center, # 회전 중심
    angle,  # 회전 각도
    scale   # 확대/축소 비율
)
# 2. 회전 변환 적용
rotated_image = cv2.warpAffine(
    image,
    rotation_matrix,
    (width, height) # 결과 이미지 크기
)

# 이미지 이동 (이동 후 빈 영역은 검은색으로 채워짐)
# 1. 이동 변환 행렬 생성
translation_matrix = np.float32([
    [1, 0, move_x],
    [0, 1, move_y]
])
# 2. 이동 변환 적용
moved_image = cv2.warpAffine(image, translation_matrix, (width, height))

# 이미지 뒤집기
flipped_horizontal = cv2.flip(image, 1)  # 좌우 반전
flipped_vertical   = cv2.flip(image, 0)  # 상하 반전
flipped_both       = cv2.flip(image, -1) # 상하좌우 반전
```