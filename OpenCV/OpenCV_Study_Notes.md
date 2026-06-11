# OpenCV 학습 정리

---

## 1. 설치

```bash
pip install opencv-python

# 설치 확인
python -c "import cv2; print(cv2.__version__)"
```

---

## 2. 기본 개념

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

| 항목 | 순서 |
|---|---|
| 배열 인덱싱 | `img[y, x]` (행, 열) |
| `img.shape` 반환 | `(height, width, channel)` |
| `cv2.resize()` 인자 | `(width, height)` ← shape와 **반대** |

### 2-2. BGR 채널 순서

OpenCV는 컬러 이미지를 **BGR** 순서로 다룬다. (일반적인 RGB와 채널 순서가 반대)

```python
pixel = img[y, x]   # [blue, green, red]
blue  = pixel[0]
green = pixel[1]
red   = pixel[2]
```

---

## 3. 이미지 I/O

```python
import cv2

# 이미지 읽기 — NumPy 배열 반환, 실패 시 None 반환
img      = cv2.imread(image_path)
img_gray = cv2.imread(image_path, cv2.IMREAD_GRAYSCALE)

# 이미지 저장
cv2.imwrite(save_path, img)

# 이미지 표시
cv2.imshow("window", img)
cv2.waitKey(0)       # 0: 키 입력까지 무한 대기
cv2.waitKey(1000)    # 1000ms(1초) 대기 후 자동으로 넘어감
cv2.destroyAllWindows()
```

---

## 4. 이미지 속성

```python
import cv2

img = cv2.imread(image_path)

# shape — 배열의 차원
# 컬러:        (height, width, channel)  예: (480, 640, 3)
# 그레이스케일: (height, width)           예: (480, 640)
print(img.shape)

# size — 전체 원소 수
# 컬러:        height × width × channel  예: 921600
# 그레이스케일: height × width            예: 307200
print(img.size)

# dtype — 픽셀 데이터 타입 (보통 uint8: 0~255)
print(img.dtype)

# 이미지 크기 추출
height, width = img.shape[:2]
```

---

## 5. 픽셀 접근

```python
import cv2

img = cv2.imread(image_path)

# 픽셀 읽기 (흑백은 밝기 값 1개, 컬러는 BGR 3개)
pixel = img[100, 200]         # y=100, x=200 위치의 픽셀

# 픽셀 쓰기 — 단일 픽셀
img[100, 200] = (0, 255, 0)   # 초록색으로 변경

# 픽셀 쓰기 — 사각형 영역 (슬라이싱)
img[50:150, 100:300] = (0, 0, 255)    # 사각형 영역을 빨간색으로 변경
```

---

## 6. 색상 변환

```python
import cv2

img = cv2.imread(image_path)

img_rgb  = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)   # BGR → RGB  (matplotlib 등과 함께 쓸 때)
img_gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)  # BGR → 그레이스케일
img_hsv  = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)   # BGR → HSV  (색상 검출에 사용)
```

### 6-1. HSV 색상 공간

HSV는 색상(Hue), 채도(Saturation), 명도(Value)로 구성된 색상 공간이다.
HSV 이미지는 사람이 직접 보기 위한 것이 아니라, **색상 검출 계산**을 위해 사용한다.
조명 환경에 따라 같은 파란색도 HSV 값이 달라질 수 있다.

| 채널 | 범위 | 의미 |
|---|---|---|
| H | 0 ~ 179 | 색상 (빨강, 초록, 파랑 등) |
| S | 0 ~ 255 | 채도 (색의 진하기) |
| V | 0 ~ 255 | 명도 (색의 밝기) |

> OpenCV는 HSV를 저장할 때 uint8 (0~255)를 사용하기 때문에 H 채널의 범위를 0~179로 압축하여 사용한다.

### 6-2. 색상 검출

#### 파란색 / 초록색

```python
import cv2
import numpy as np

img = cv2.imread(image_path)
hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)

# 파란색
# H: 100~130 (파란색 계열) / S: 100~255 / V: 100~255
lower_blue = np.array([100, 100, 100])
upper_blue = np.array([130, 255, 255])

blue_mask   = cv2.inRange(hsv, lower_blue, upper_blue)   # 범위 안 → 255, 나머지 → 0
blue_result = cv2.bitwise_and(img, img, mask=blue_mask)  # 파란색 영역만 남기고 나머지는 검정

# 초록색
lower_green = np.array([40, 80, 80])
upper_green = np.array([80, 255, 255])

green_mask   = cv2.inRange(hsv, lower_green, upper_green)
green_result = cv2.bitwise_and(img, img, mask=green_mask)
```

#### 빨간색

빨간색은 HSV에서 H 값이 0 근처와 179 근처로 **두 범위에 걸쳐 존재**하므로 두 마스크를 합산한다.

```python
hsv = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)

lower_red1 = np.array([0, 100, 100])
upper_red1 = np.array([10, 255, 255])

lower_red2 = np.array([170, 100, 100])
upper_red2 = np.array([179, 255, 255])

mask1 = cv2.inRange(hsv, lower_red1, upper_red1)
mask2 = cv2.inRange(hsv, lower_red2, upper_red2)

red_mask   = cv2.bitwise_or(mask1, mask2)
red_result = cv2.bitwise_and(image, image, mask=red_mask)
```

---

## 7. 전처리

### 7-1. 밝기·대비 조절

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

### 7-2. 이미지 반전

```
새 픽셀 값 = 255 - 원래 픽셀 값
```

```python
import cv2

img      = cv2.imread(image_path)
inverted = cv2.bitwise_not(img)
```

### 7-3. 이진화 (Thresholding)

픽셀 값을 기준으로 흑/백 두 가지로 분류한다.

| 방법 | 임계값 | 특징 |
|---|---|---|
| 기본 이진화 | 수동 지정 | 단순, 조명 변화에 취약 |
| Adaptive Threshold | 픽셀별 자동 계산 | 조명이 고르지 않은 이미지에 적합 |
| Otsu Threshold | 전체 이미지 기반 | 밝기가 두 그룹으로 뚜렷이 나뉠 때 적합 |

```python
import cv2

img  = cv2.imread(image_path)
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

---

## 8. 기하 변환

```python
import cv2
import numpy as np

img = cv2.imread(image_path)
height, width = img.shape[:2]

# 크기 변경 (인자 순서: width, height — shape와 반대)
resized = cv2.resize(img, (new_width, new_height))

# 뒤집기
flipped_horizontal = cv2.flip(image, 1)   # 좌우 반전
flipped_vertical   = cv2.flip(image, 0)   # 상하 반전
flipped_both       = cv2.flip(image, -1)  # 상하좌우 반전

# 회전
rotation_matrix = cv2.getRotationMatrix2D(center, angle, scale)
rotated_image   = cv2.warpAffine(image, rotation_matrix, (width, height))

# 이동 (이동 후 빈 영역은 검은색으로 채워짐)
translation_matrix = np.float32([
    [1, 0, move_x],
    [0, 1, move_y]
])
moved_image = cv2.warpAffine(image, translation_matrix, (width, height))

# Affine 변환 — 이동, 회전, 확대/축소, 기울이기 (점 3개 기반)
affine_matrix = cv2.getAffineTransform(src_points, dst_points)  # np.float32 좌표
affine_image  = cv2.warpAffine(image, affine_matrix, (width, height))

# Perspective 변환 — 원근 왜곡 제거, 시점 변환 (점 4개 기반)
perspective_matrix = cv2.getPerspectiveTransform(src_points, dst_points)  # np.float32 좌표
perspective_image  = cv2.warpPerspective(image, perspective_matrix, (width, height))

# 패딩
bordered = cv2.copyMakeBorder(
    image,
    top, bottom, left, right,       # 각 방향에 추가할 여백 크기 (픽셀)
    borderType=cv2.BORDER_CONSTANT, # 고정된 색상으로 여백 채우기
    value=(0, 0, 0)                 # 여백 색상 (BGR)
)

# 피라미드
down_image = cv2.pyrDown(image)      # 가우시안 블러 → 절반 크기로 축소
up_image   = cv2.pyrUp(down_image)   # 픽셀 사이를 보간하여 2배 크기로 확대
```

> **Affine vs Perspective**
> - Affine: 점 3개 기반, 평행선 유지 (이동·회전·기울이기)
> - Perspective: 점 4개 기반, 평행선 불유지 (원근 왜곡 제거)

---

## 9. 필터링과 노이즈 제거

```python
import cv2
import numpy as np

# 평균 블러: 주변 픽셀들의 평균값으로 현재 픽셀 값을 바꿈
# 커널 크기가 클수록 블러 효과가 강해짐
blur_image = cv2.blur(image, (ksize, ksize))

# 가우시안 블러: 중심 픽셀과 가까운 주변 픽셀에 더 큰 가중치를 주는 블러
# 커널 크기가 클수록 블러 효과가 강해짐, 커널 크기는 홀수여야 함
gaussian_blur = cv2.GaussianBlur(image, (ksize, ksize), 0)

# 중앙값 블러: 주변 픽셀들의 중앙값으로 현재 픽셀 값을 바꿈
# salt-and-pepper 노이즈 제거에 효과적, 커널 크기는 홀수여야 함
median_blur = cv2.medianBlur(image, ksize)

# Bilateral 필터: 경계선을 정확하게 보존하면서 노이즈 제거
bilateral_filter = cv2.bilateralFilter(
    image,
    d,          # 필터링할 픽셀 주변 영역의 지름
    sigmaColor, # 색상 차이 허용 범위
    sigmaSpace  # 거리 차이 허용 범위
)

# Edge Preserving 필터: 경계를 유지하면서 표면의 노이즈를 부드럽게 다듬기
edge_preserved = cv2.edgePreservingFilter(
    image,
    flags=1,    # 필터 종류 선택
    sigma_s=60, # 필터가 고려할 주변 픽셀 영역의 크기
    sigma_r=0.4 # 색상 차이에 대한 민감도
)

# 샤프닝 필터: 이미지를 더 선명하게 만드는 필터
sharpening_kernel = np.array([
    [0, -1, 0],
    [-1, 5, -1],
    [0, -1, 0]
])
sharpened_image = cv2.filter2D(
    image,
    -1, # -1: 원본의 자료형을 동일하게 사용
    sharpening_kernel
)

# 노이즈 생성: 정규분포(가우시안 분포)를 따르는 랜덤 값을 생성
noise = np.random.normal(loc=0, scale=25, size=image.shape).astype(np.int16)
noisy_image = np.clip(image.astype(np.int16) + noise, 0, 255).astype(np.uint8)
```

| 필터 | 특징 | 주요 용도 |
|---|---|---|
| 평균 블러 | 단순 평균 | 기본 노이즈 제거 |
| 가우시안 블러 | 중심 가중 평균 | 부드러운 노이즈 제거, Edge 검출 전처리 |
| 중앙값 블러 | 중앙값 사용 | Salt-and-pepper 노이즈 제거 |
| Bilateral | 경계 정밀 보존 | 정밀한 노이즈 제거 |
| Edge Preserving | 경계 유지 + 다듬기 | 미적 처리 |
| 샤프닝 | 선명도 강화 | 이미지 강조 |

---

## 10. Edge · Contour · Shape 분석

### 10-1. Edge 검출

```python
import cv2

# Sobel Edge: x/y 방향 밝기 변화를 각각 검출
sobel_x     = cv2.Sobel(gray_img, cv2.CV_64F, 1, 0, ksize=3)
sobel_y     = cv2.Sobel(gray_img, cv2.CV_64F, 0, 1, ksize=3)
sobel_x_abs = cv2.convertScaleAbs(sobel_x)  # 절대값 변환 → uint8
sobel_y_abs = cv2.convertScaleAbs(sobel_y)
sobel_combined = cv2.addWeighted(sobel_x_abs, 0.5, sobel_y_abs, 0.5, 0)  # 0.5x + 0.5y + 0

# Laplacian Edge: x/y 방향 변화량을 한 번에 계산 (노이즈에 민감 → 블러 먼저)
blurred       = cv2.GaussianBlur(gray_img, (5, 5), 0)
laplacian     = cv2.Laplacian(blurred, cv2.CV_64F)
laplacian_abs = cv2.convertScaleAbs(laplacian)

# Canny Edge: threshold 두 개로 강/약 Edge를 구분하여 검출
# threshold2 이상 → Edge
# threshold1 이하 → Edge 아님
# 그 사이   → threshold2 이상인 Edge와 연결된 경우에만 Edge로 인정
blurred = cv2.GaussianBlur(gray, (5, 5), 0)
edges   = cv2.Canny(blurred, threshold1, threshold2)
```

| 방법 | 특징 |
|---|---|
| Sobel | x/y 방향 별도 계산, 방향성 있음 |
| Laplacian | x/y 한 번에 계산, 노이즈에 민감 |
| Canny | 이중 임계값으로 정교한 검출, 가장 많이 사용 |

### 10-2. Contour 검출

```python
import cv2

ret, binary = cv2.threshold(gray, 127, 255, cv2.THRESH_BINARY)

# contours: 검출된 외곽선 목록 / hierarchy: 외곽선들의 계층 구조
contours, hierarchy = cv2.findContours(
    binary,
    mode=cv2.RETR_EXTERNAL,         # 가장 바깥쪽 외곽선만 검출
    method=cv2.CHAIN_APPROX_SIMPLE  # 외곽선을 단순화하여 저장
)

# 외곽선 그리기
cv2.drawContours(
    result,
    contours,
    -1,          # -1: 모두 그리기
    (0, 0, 255), # 색상 (BGR)
    2            # 선 두께
)

# 면적 필터링
for contour in contours:
    area = cv2.contourArea(contour)
    if area > min_area:
        cv2.drawContours(result, [contour], -1, (0, 255, 0), 2)
```

### 10-3. Shape 분석

```python
import cv2

# Bounding Box: Contour를 감싸는 가장 작은 직사각형
x, y, w, h = cv2.boundingRect(contour)
cv2.rectangle(
    result,
    (x, y), (x + w, y + h), # (사각형의 왼쪽 위 좌표), (사각형의 오른쪽 아래 좌표)
    (0, 255, 0), 2
)

# 최소 외접 원: Contour를 포함하는 가장 작은 원
center, radius = cv2.minEnclosingCircle(contour)
cv2.circle(result, center, int(radius), (0, 255, 255), 2)

# 다각형 근사: 복잡한 Contour를 단순한 꼭짓점 집합으로 줄이기
perimeter = cv2.arcLength(contour, True)  # 외곽선 둘레 길이 계산
approx = cv2.approxPolyDP(
    contour,
    0.02 * perimeter,  # epsilon: 근사 오차의 최대 허용치
    True               # True: 닫힌 도형
)
cv2.drawContours(result, [approx], -1, (0, 255, 0), 2)

# 객체 중심점 계산 (모멘트 활용)
# m00: 면적 / m10: x 방향 1차 모멘트 / m01: y 방향 1차 모멘트
moments = cv2.moments(contour)
if moments["m00"] != 0:
    center_x = int(moments["m10"] / moments["m00"])
    center_y = int(moments["m01"] / moments["m00"])
```

---

## 11. 그리기

```python
import cv2

# 텍스트
cv2.putText(
    img, text,
    org,            # 텍스트 시작점 좌표 (기준: 왼쪽 아래)
    fontFace, fontScale,
    color,          # BGR
    thickness
)

# 직선
cv2.line(img, pt1, pt2, color, thickness)

# 사각형
cv2.rectangle(
    img,
    pt1,  # 왼쪽 위 꼭짓점
    pt2,  # 오른쪽 아래 꼭짓점
    color, thickness
)

# 원
cv2.circle(
    img, center, radius,
    color,
    thickness  # -1: 원 내부 채우기
)
```

---

## 12. 카메라 및 비디오

### 12-1. 카메라 기본

```python
import cv2

cap = cv2.VideoCapture(0)      # 0: 웹캠 열기

cap.isOpened()                 # 웹캠 열기 성공 여부 확인

ret, frame = cap.read()        # 한 프레임 읽기 — ret: 성공 여부, frame: 프레임

cap.release()                  # 카메라 해제
```

### 12-2. 해상도 설정

```python
# 해상도 설정 (카메라가 요청한 해상도를 지원하지 않으면 다른 값이 적용될 수 있음)
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)

# 실제 적용된 값 확인
cap.get(cv2.CAP_PROP_FRAME_WIDTH)
cap.get(cv2.CAP_PROP_FRAME_HEIGHT)
```

### 12-3. FPS 계산

```python
prev_time = time.time()

while True:
    ret, frame = cap.read()

    current_time = time.time()
    elapsed_time = current_time - prev_time
    prev_time = current_time

    if elapsed_time > 0:
        fps = 1.0 / elapsed_time
    else:
        fps = 0
```

### 12-4. 비디오 파일 읽기

```python
cap = cv2.VideoCapture(video_path)

while True:
    ret, frame = cap.read()

    if not ret:
        print("비디오가 끝났거나 프레임을 읽을 수 없습니다.")
        break

    cv2.imshow("Video File", frame)
```

### 12-5. 비디오 저장

```python
cap = cv2.VideoCapture(0)

width  = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
height = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
fps    = 20.0

fourcc = cv2.VideoWriter_fourcc(*'mp4v')    # 코덱 설정

out = cv2.VideoWriter(  # 비디오 저장 객체 생성
    "output_camera.mp4",
    fourcc,
    fps,
    (width, height)
)

while True:
    ret, frame = cap.read()

    out.write(frame)    # 현재 프레임 저장

    cv2.imshow("Recording Camera", frame)

    if cv2.waitKey(1) == ord('q'):
        break

cap.release()
out.release()   # 비디오 저장 객체 해제
cv2.destroyAllWindows()
```

### 12-6. 실시간 변환

```python
cap   = cv2.VideoCapture(0)
count = 0

while True:
    ret, frame = cap.read()

    if not ret:
        print("프레임을 읽을 수 없습니다.")
        break

    gray    = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)   # 흑백 프레임 변환
    blurred = cv2.GaussianBlur(gray, (5, 5), 0)         # 가우시안 블러 적용
    edges   = cv2.Canny(blurred, 50, 150)               # Canny 엣지 검출

    if key == ord('s'):
        file_path = os.path.join(save_dir, f"capture_{count:04d}.jpg")
        cv2.imwrite(file_path, frame)
        print("이미지 저장:", file_path)
        count += 1
```

---

## 13. 특징점 · 매칭 · 템플릿 매칭

### 13-1. 이미지 히스토그램

이미지의 픽셀 밝기 값이 얼마나 분포되어 있는지 보여주는 그래프.
흑백 이미지 기준으로 픽셀 값은 보통 0~255.

```python
import cv2
import matplotlib.pyplot as plt  # 그래프 출력

image = cv2.imread(image_path)
gray  = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)

# 히스토그램 평활화
# (이미지 전체) 어둡거나 대비가 낮은 이미지를 더 선명하게 보이도록 밝기 분포를 넓게 펴는 기법
# 노이즈도 함께 강조할 수 있음
equalized = cv2.equalizeHist(gray)

# CLAHE(Contrast Limited Adaptive Histogram Equalization)
# 이미지를 작은 타일(tile)로 나누어 각 타일마다 히스토그램 평활화를 적용
clahe = cv2.createCLAHE(
    clipLimit=2.0,      # 대비 제한 값
    tileGridSize=(8, 8) # 타일 크기
)
clahe_equalized = clahe.apply(gray)

hist_original = cv2.calcHist(
    [gray],
    [0],        # 사용할 채널 인덱스 (흑백은 채널이 하나뿐)
    None,       # 마스크 (없으면 None)
    [256],      # 히스토그램 구간(bin) 개수
    [0, 256]    # 범위 (0~255)
)
hist_equalized = cv2.calcHist([equalized], [0], None, [256], [0, 256])

cv2.imshow("Original Gray", gray)
cv2.imshow("Equalized Gray", equalized)

plt.figure()
plt.title("Histogram Comparison")
plt.plot(hist_original, label="Original")
plt.plot(hist_equalized, label="Equalized")
plt.xlim([0, 256])
plt.legend()
plt.show()

cv2.waitKey(0)
cv2.destroyAllWindows()
```

> **equalizeHist vs CLAHE**
> - `equalizeHist`: 전체 이미지 기준, 빠르지만 노이즈 강조 가능
> - `CLAHE`: 타일 단위 적용, 지역적 대비 개선, 노이즈 억제

### 13-2. Template Matching

작은 템플릿 이미지를 큰 이미지 안에서 찾는 방법

```python
import cv2

scene    = cv2.imread(scene_path)       # 전체 이미지
template = cv2.imread(template_path)    # 찾을 이미지

scene_gray    = cv2.cvtColor(scene, cv2.COLOR_BGR2GRAY)
template_gray = cv2.cvtColor(template, cv2.COLOR_BGR2GRAY)

# 템플릿 매칭 - 결과값: 유사도 맵
result = cv2.matchTemplate(
    scene_gray,
    template_gray,
    cv2.TM_CCOEFF_NORMED    # 비교 방법
)

min_val, max_val, min_loc, max_loc = cv2.minMaxLoc(result)  # _loc는 좌표

template_height, template_width = template_gray.shape[:2]

top_left     = max_loc
bottom_right = (
    top_left[0] + template_width,
    top_left[1] + template_height
)

output = scene.copy()

cv2.rectangle(
    output,
    top_left,
    bottom_right,
    (0, 0, 255),
    2
)
```

### 13-3. ORB 특징점 검출 및 매칭

이미지에서 특징적인 점을 찾아내는 알고리즘

```python
import cv2

reference  = cv2.imread(reference_path)  # 찾고 싶은 물체 이미지
scene      = cv2.imread(scene_path)      # 전체 이미지
ref_gray   = cv2.cvtColor(reference, cv2.COLOR_BGR2GRAY)
scene_gray = cv2.cvtColor(scene, cv2.COLOR_BGR2GRAY)

# ORB 특징점 검출기 생성
# nfeatures: 검출할 특징점의 최대 개수
orb = cv2.ORB_create(nfeatures=1500)

# 특징점 계산
# keypoints   : 특징점 위치, 크기, 방향 정보
# descriptors : 특징점을 숫자 벡터로 표현한 정보 (이진)
ref_keypoints,   ref_descriptors   = orb.detectAndCompute(ref_gray, None)
scene_keypoints, scene_descriptors = orb.detectAndCompute(scene_gray, None)

# Brute Force Matcher 생성
matcher = cv2.BFMatcher(cv2.NORM_HAMMING, crossCheck=True)

# 두 이미지의 descriptor를 매칭
matches = matcher.match(ref_descriptors, scene_descriptors)

# 매칭 선택 — 특정 거리 이하만 선택
good_matches = [
    match for match in matches
    if match.distance < 50
]

output = scene.copy()

if len(good_matches) > 30:
    result_text = "Object Detected"
    color = (0, 255, 0)
else:
    result_text = "Object Not Detected"
    color = (0, 0, 255)

cv2.putText(
    output,
    result_text,
    (30, 50),
    cv2.FONT_HERSHEY_SIMPLEX,
    1.0,
    color,
    2
)

# 특징점 시각화
match_view = cv2.drawMatches(
    reference,
    ref_keypoints,
    scene,
    scene_keypoints,
    sorted(good_matches, key=lambda x: x.distance)[:50],
    None,
    flags=cv2.DrawMatchesFlags_NOT_DRAW_SINGLE_POINTS
)
```

> **Template Matching vs ORB**
> - Template Matching: 템플릿과 동일한 크기/각도일 때 적합, 단순·빠름
> - ORB: 크기·회전 변화에 강인, 복잡한 환경에서 사용

---

## 14. ROS2 연계

```
/camera/image_raw Subscribe                           [ROS2 메시지 형식]
→ cv_bridge로 ROS2 Image 메시지를 OpenCV frame으로 변환  [NumPy 배열]
→ OpenCV 처리
→ 결과 이미지 또는 좌표 Topic Publish
```

### 14-1. cv_bridge

```python
from cv_bridge import CvBridge

# 변환 도구 객체 생성
bridge = CvBridge()

# ROS2 Image 메시지 → OpenCV 이미지
cv_image = bridge.imgmsg_to_cv2(msg, desired_encoding="bgr8")

# OpenCV 이미지 → ROS2 Image 메시지
ros_image_msg = bridge.cv2_to_imgmsg(cv_image, encoding="bgr8")
```

### 14-2. ROS2 Image 메시지

| 주요 필드 | 설명 |
|---|---|
| header | 메시지의 시간, 좌표계 정보 |
| height, width | 이미지 높이, 너비 |
| encoding | 픽셀 포맷 (bgr8, rgb8, mono8 등) |
| step | 한 줄의 바이트 수 (step = width × 채널 수) |
| data | 픽셀 데이터 (1차원 배열) |

```python
def image_callback(msg):
    print("height:",   msg.height)
    print("width:",    msg.width)
    print("encoding:", msg.encoding)
    print("step:",     msg.step)
```

### 14-3. 기본 Publisher / Subscriber

#### Publisher — OpenCV 이미지를 Topic으로 발행

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from cv_bridge import CvBridge
import cv2

class OpenCVImagePublisher(Node):
    def __init__(self):
        super().__init__("opencv_image_publisher")

        self.publisher = self.create_publisher(Image, "/opencv/image", 10)  # Publisher 생성
        self.bridge    = CvBridge()
        self.timer     = self.create_timer(0.1, self.timer_callback)  # 0.1초마다 timer_callback 함수 실행
        self.image     = cv2.imread("practice_images/sample.jpg")

    def timer_callback(self):
        if self.image is None:
            self.get_logger().error("이미지를 읽을 수 없습니다.")
            return

        msg = self.bridge.cv2_to_imgmsg(self.image, encoding="bgr8")     # OpenCV 이미지 → ROS2 Image 메시지 변환
        self.publisher.publish(msg)                                      # Topic 발행
        self.get_logger().info("OpenCV 이미지를 ROS2 Image로 발행했습니다.") # 발행 로그 출력


def main(args=None):
    rclpy.init(args=args)           # ROS2 Python 시스템 초기화
    node = OpenCVImagePublisher()   # 노드 생성
    rclpy.spin(node)                # 노드 실행
    node.destroy_node()             # 노드 소멸
    rclpy.shutdown()                # ROS2 Python 시스템 종료


if __name__ == "__main__":
    main()
```

#### Subscriber — Topic을 구독해 OpenCV로 표시

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from cv_bridge import CvBridge
import cv2

class ImageSubscriber(Node):
    def __init__(self):
        super().__init__("image_subscriber")

        self.bridge       = CvBridge()
        self.subscription = self.create_subscription(
            Image, "/camera/image_raw", self.image_callback, 10
        )

    def image_callback(self, msg):
        frame = self.bridge.imgmsg_to_cv2(msg, desired_encoding="bgr8")
        cv2.imshow("ROS2 Camera Image", frame)
        cv2.waitKey(1)


def main(args=None):
    rclpy.init(args=args)
    node = ImageSubscriber()
    rclpy.spin(node)
    node.destroy_node()
    cv2.destroyAllWindows()
    rclpy.shutdown()


if __name__ == "__main__":
    main()
```

---

### 14-4. 노드 구조 설계

구독 → 처리 → 발행 흐름을 메서드로 분리하는 기본 패턴.

#### 스켈레톤 (구조 파악용)

```python
class VisionNode(Node):
    def __init__(self):
        self.bridge = CvBridge()
        self.image_sub = ...
        self.image_pub = ...
        self.result_pub = ...

    def image_callback(self, msg):
        frame = self.bridge.imgmsg_to_cv2(msg, "bgr8")
        processed_frame, result = self.process_frame(frame)
        self.publish_image(processed_frame)
        self.publish_result(result)

    def process_frame(self, frame ):
        # OpenCV 처리
        return processed_frame, result

    def publish_image(self, processed_frame):
        # 처리 이미지 발행
        pass

    def publish_result(self, result):
        # 좌표, 상태 등 발행
        pass
```

#### 실제 구현 예시 (흑백 변환)

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from cv_bridge import CvBridge
import cv2

class BasicVisionSubscriber(Node):
    def __init__(self):
        super().__init__("basic_vision_subscriber")

        self.bridge    = CvBridge()
        self.image_sub = self.create_subscription(
            Image, "/camera/image_raw", self.image_callback, 10
        )

    def image_callback(self, msg):
        try:
            frame           = self.bridge.imgmsg_to_cv2(msg, desired_encoding="bgr8")
            processed_frame = self.process_frame(frame)
            cv2.imshow("Processed Frame", processed_frame)
            cv2.waitKey(1)

        except Exception as e:
            self.get_logger().error(f"이미지 처리 오류: {e}")

    def process_frame(self, frame):
        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        return cv2.cvtColor(gray, cv2.COLOR_GRAY2BGR)


def main(args=None):
    rclpy.init(args=args)
    node = BasicVisionSubscriber()
    rclpy.spin(node)
    node.destroy_node()
    cv2.destroyAllWindows()
    rclpy.shutdown()


if __name__ == "__main__":
    main()
```

---

### 14-5. 실시간 Edge Publisher

구독한 카메라 프레임에 Canny Edge를 적용해 결과를 Topic으로 발행한다.

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from cv_bridge import CvBridge
import cv2


class EdgePublisher(Node):
    def __init__(self):
        super().__init__("edge_publisher")

        self.bridge    = CvBridge()
        self.image_sub = self.create_subscription(
            Image, "/camera/image_raw", self.image_callback, 10
        )
        self.edge_pub  = self.create_publisher(
            Image, "/vision/edge_image", 10
        )

    def image_callback(self, msg):
        try:
            frame      = self.bridge.imgmsg_to_cv2(msg, desired_encoding="bgr8")
            edge_image = self.process_frame(frame)

            edge_msg        = self.bridge.cv2_to_imgmsg(edge_image, encoding="mono8")  # Canny 결과는 1채널 이미지
            edge_msg.header = msg.header    # 시간 정보와 frame_id를 유지

            self.edge_pub.publish(edge_msg)

        except Exception as e:
            self.get_logger().error(f"Edge 처리 오류: {e}")

    def process_frame(self, frame):
        gray    = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        blurred = cv2.GaussianBlur(gray, (5, 5), 0)
        edges   = cv2.Canny(blurred, 50, 150)
        return edges


def main(args=None):
    rclpy.init(args=args)
    node = EdgePublisher()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()


if __name__ == "__main__":
    main()
```

---

### 14-6. 객체 검출 Publisher

파란색 물체의 중심 좌표와 면적을 `Point` 메시지로 발행한다.
14-6-2는 여기에 **화면 중앙 기준 오차(error_x)** 와 **디버그 이미지 발행**을 추가한 확장판이다.

#### 14-6-1. 객체 중심 좌표 Publisher

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from geometry_msgs.msg import Point
from cv_bridge import CvBridge
import cv2
import numpy as np


class TargetCenterPublisher(Node):
    def __init__(self):
        super().__init__("target_center_publisher")

        self.bridge     = CvBridge()
        self.image_sub  = self.create_subscription(
            Image, "/camera/image_raw", self.image_callback, 10
        )
        self.center_pub = self.create_publisher(
            Point, "/vision/target_center", 10
        )

    def image_callback(self, msg):
        try:
            frame  = self.bridge.imgmsg_to_cv2(msg, desired_encoding="bgr8")
            target = self.detect_blue_target(frame)

            if target is not None:
                center_x, center_y, area = target

                point_msg   = Point()
                point_msg.x = float(center_x)
                point_msg.y = float(center_y)
                point_msg.z = float(area)

                self.center_pub.publish(point_msg)
                self.get_logger().info(
                    f"target center: x={center_x}, y={center_y}, area={area}"
                )

        except Exception as e:
            self.get_logger().error(f"객체 중심 처리 오류: {e}")

    def detect_blue_target(self, frame):
        hsv        = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
        lower_blue = np.array([100, 100, 100])
        upper_blue = np.array([130, 255, 255])
        mask       = cv2.inRange(hsv, lower_blue, upper_blue)

        kernel = np.ones((5, 5), np.uint8)
        # cv2.MORPH_OPEN:  Erode → Dilate (작은 노이즈 제거)
        # cv2.MORPH_CLOSE: Dilate → Erode (구멍 메우기)
        mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN,  kernel)
        mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, kernel)

        contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

        if len(contours) == 0:
            return None

        largest_contour = max(contours, key=cv2.contourArea)
        area            = cv2.contourArea(largest_contour)

        if area < 500:
            return None

        moments = cv2.moments(largest_contour)

        if moments["m00"] == 0:
            return None

        center_x = int(moments["m10"] / moments["m00"])
        center_y = int(moments["m01"] / moments["m00"])

        return center_x, center_y, area


def main(args=None):
    rclpy.init(args=args)
    node = TargetCenterPublisher()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()


if __name__ == "__main__":
    main()
```

#### 14-6-2. 로봇 추종용 비전 노드 (확장)

`error_x = center_x - image_center_x` 를 추가로 발행하고, 디버그 이미지도 함께 발행한다.

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from geometry_msgs.msg import Point
from cv_bridge import CvBridge
import cv2
import numpy as np


class FollowVisionNode(Node):
    def __init__(self):
        super().__init__("follow_vision_node")

        self.bridge    = CvBridge()
        self.image_sub = self.create_subscription(
            Image, "/camera/image_raw", self.image_callback, 10
        )
        self.error_pub = self.create_publisher(
            Point, "/vision/target_error", 10   # x: error_x / y: center_y / z: area
        )
        self.debug_image_pub = self.create_publisher(
            Image, "/vision/debug_image", 10
        )

    def image_callback(self, msg):
        try:
            frame               = self.bridge.imgmsg_to_cv2(msg, desired_encoding="bgr8")
            debug_frame, result = self.process_frame(frame)

            if result is not None:
                error_msg   = Point()
                error_msg.x = float(result["error_x"])
                error_msg.y = float(result["center_y"])
                error_msg.z = float(result["area"])
                self.error_pub.publish(error_msg)

            debug_msg        = self.bridge.cv2_to_imgmsg(debug_frame, encoding="bgr8")
            debug_msg.header = msg.header
            self.debug_image_pub.publish(debug_msg)

        except Exception as e:
            self.get_logger().error(f"추종 비전 처리 오류: {e}")

    def process_frame(self, frame):
        height, width  = frame.shape[:2]
        image_center_x = width // 2

        hsv        = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
        lower_blue = np.array([100, 100, 100])
        upper_blue = np.array([130, 255, 255])
        mask       = cv2.inRange(hsv, lower_blue, upper_blue)

        kernel = np.ones((5, 5), np.uint8)
        mask   = cv2.morphologyEx(mask, cv2.MORPH_OPEN,  kernel)
        mask   = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, kernel)

        contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

        debug_frame = frame.copy()
        cv2.line(debug_frame, (image_center_x, 0), (image_center_x, height), (255, 0, 0), 2)

        if len(contours) == 0:
            cv2.putText(debug_frame, "Target Not Found", (20, 40),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.8, (0, 0, 255), 2)
            return debug_frame, None

        largest_contour = max(contours, key=cv2.contourArea)
        area            = cv2.contourArea(largest_contour)

        if area < 500:
            return debug_frame, None

        moments = cv2.moments(largest_contour)

        if moments["m00"] == 0:
            return debug_frame, None

        center_x = int(moments["m10"] / moments["m00"])
        center_y = int(moments["m01"] / moments["m00"])
        error_x  = center_x - image_center_x   # 화면 중앙 기준 오차

        cv2.drawContours(debug_frame, [largest_contour], -1, (0, 255, 0), 2)
        cv2.circle(debug_frame, (center_x, center_y), 8, (0, 0, 255), -1)
        cv2.putText(debug_frame, f"error_x: {error_x}", (20, 40),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.8, (0, 255, 255), 2)

        result = {
            "center_x": center_x,
            "center_y": center_y,
            "area":     area,
            "error_x":  error_x
        }

        return debug_frame, result


def main(args=None):
    rclpy.init(args=args)
    node = FollowVisionNode()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()


if __name__ == "__main__":
    main()
```
