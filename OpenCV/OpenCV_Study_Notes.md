# OpenCV

## 0. 설치

```bash
pip install opencv-python

# 설치 확인
python -c "import cv2; print(cv2.__version__)"
```

---

## 1. 기본 개념

### 1-1. 좌표계

OpenCV의 좌표 원점은 이미지 **좌상단**이다.

```
(0,0) ───────────→ x (width)
  │
  │
  │
  ↓
  y (height)
```

- 배열 인덱싱 순서: `img[y, x]` (행, 열 순서)
- `img.shape` 반환 순서: `(height, width, channel)`
- `cv2.resize()` 인자 순서: `(width, height)` ← shape와 반대

### 1-2. BGR 채널 순서

OpenCV는 컬러 이미지를 **BGR** 순서로 다룬다. (일반적인 RGB와 채널 순서가 반대)

```python
pixel = img[y, x]   # [blue, green, red]
blue  = pixel[0]
green = pixel[1]
red   = pixel[2]
```

---

## 2. 이미지 I/O

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

## 3. 이미지 속성

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

## 4. 픽셀 접근

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

## 5. 색상 변환

```python
import cv2

img = cv2.imread(image_path)

img_rgb  = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)   # BGR → RGB  (matplotlib 등과 함께 쓸 때)
img_gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)  # BGR → 그레이스케일
img_hsv  = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)   # BGR → HSV  (색상 검출에 사용)
```

### 5-1. HSV 색상 공간

HSV는 색상(Hue), 채도(Saturation), 명도(Value)로 구성된 색상 공간이다.
HSV 이미지는 사람이 직접 보기 위한 것이 아니라, **색상 검출 계산**을 위해 사용한다.
조명 환경에 따라 같은 파란색도 HSV 값이 달라질 수 있다.

| 채널 | 범위 | 의미 |
|---|---|---|
| H | 0 ~ 179 | 색상 (빨강, 초록, 파랑 등) |
| S | 0 ~ 255 | 채도 (색의 진하기) |
| V | 0 ~ 255 | 명도 (색의 밝기) |

> OpenCV는 HSV를 저장할 때 uint8 (0~255)를 사용하기 때문에 H 채널의 범위를 0~179로 압축하여 사용한다.

---

### 5-2. 색상 검출

#### 파란색/초록색

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

green_mask = cv2.inRange(hsv, lower_green, upper_green)
green_result = cv2.bitwise_and(image, image, mask=green_mask)
```

#### 빨간색

```python
hsv = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)

lower_red1 = np.array([0, 100, 100])
upper_red1 = np.array([10, 255, 255])

lower_red2 = np.array([170, 100, 100])
upper_red2 = np.array([179, 255, 255])

mask1 = cv2.inRange(hsv, lower_red1, upper_red1)
mask2 = cv2.inRange(hsv, lower_red2, upper_red2)

red_mask = cv2.bitwise_or(mask1, mask2)
red_result = cv2.bitwise_and(image, image, mask=red_mask)
```

---

## 6. 전처리

### 6-1. 밝기·대비 조절

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

### 6-2. 이미지 반전

```
새 픽셀 값 = 255 - 원래 픽셀 값
```

```python
import cv2

img      = cv2.imread(image_path)
inverted = cv2.bitwise_not(img)
```

### 6-3. 이진화 (Thresholding)

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

## 7. 기하 변환

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

---

## 8. 필터링과 노이즈 제거

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

> **Bilateral vs Edge Preserving**
> - Bilateral Filter: 경계를 정확하게 보존하면서 노이즈 제거 (정밀한 작업)
> - Edge Preserving Filter: 경계를 유지하면서 사진을 예쁘게 다듬기 (미적 처리)

---

## 9. Edge · Contour · Shape 분석

### 9-1. Edge 검출

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

### 9-2. Contour 검출

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

### 9-3. Shape 분석

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

## 10. 그리기

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

## 11. 카메라

```python
import cv2

cap = cv2.VideoCapture(0)      # 0: 웹캠 열기

cap.isOpened()                 # 웹캠 열기 성공 여부 확인

ret, frame = cap.read()        # 한 프레임 읽기 — ret: 성공 여부, frame: 프레임

cap.release()                  # 카메라 해제
```

---

## 11-1. 해상도 설정

```python
# 해상도 설정 (카메라가 요청한 해상도를 지원하지 않으면 다른 값이 적용될 수 있음)
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)

# 실제 적용된 값 확인
cap.get(cv2.CAP_PROP_FRAME_WIDTH)
cap.get(cv2.CAP_PROP_FRAME_HEIGHT)
```

---

## 11-2. fps

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

---

## 11-3. 비디오 파일 읽기

```python
cap = cv2.VideoCapture(video_path)

while True:
    ret, frame = cap.read()

    if not ret:
        print("비디오가 끝났거나 프레임을 읽을 수 없습니다.")
        break

    cv2.imshow("Video File", frame)
```

---

## 11-4. 비디오 저장

```python
cap = cv2.VideoCapture(0)

width = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
height = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
fps = 20.0

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

---

## 11-5. 실시간 변환

```python
cap = cv2.VideoCapture(0)

count = 0

while True:
    ret, frame = cap.read()

    if not ret:
        print("프레임을 읽을 수 없습니다.")
        break

    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)  # 흑백 프레임 변환
    blurred = cv2.GaussianBlur(gray, (5, 5), 0)     # 가우시안 블러 적용
    edges = cv2.Canny(blurred, 50, 150)             # Canny 엣지 검출

    if key == ord('s'):
        file_path = os.path.join(save_dir, f"capture_{count:04d}.jpg")
        cv2.imwrite(file_path, frame)
        print("이미지 저장:", file_path)
        count += 1
```

---

