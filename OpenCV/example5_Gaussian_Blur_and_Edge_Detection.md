# OpenCV

## 5번 예제

### 1. 전체 실행 흐름 (Flow)

```
[시작]
    │
    ▼
[create_edge_test_image()]
    ├─ np.full()       → 흰색 배경 (480×640)
    ├─ cv2.rectangle() → 검정 사각형 테두리
    ├─ cv2.circle()    → 파란 채워진 원
    ├─ cv2.polylines() → 빨간 삼각형 (새 함수!)
    ├─ cv2.line() × 3  → 초록 주행선 2개 + 회색 약한 선
    ├─ np.random + 반복문 → 랜덤 노이즈 200개 픽셀
    └─ cv2.putText()   → 안내 텍스트
    │
    ▼
[save_image()] → 05_original_edge_test.png
    │
    ▼
[cv2.cvtColor(BGR→GRAY)] → gray
[save_image()] → 05_gray.png
    │
    ▼
[cv2.GaussianBlur(gray, (5,5), 0)] → blurred
[save_image()] → 05_blurred.png
    │
    ▼
[cv2.Canny(blurred, 50, 150)] → edges_50_150  ← 기준값
[save_image()] → 05_canny_50_150.png
    │
    ▼
[threshold 비교 실험]
    ├─ cv2.Canny(blurred, 30, 100)  → 더 민감 (엣지 많이 검출)
    ├─ cv2.Canny(blurred, 100, 200) → 표준
    └─ cv2.Canny(blurred, 150, 250) → 덜 민감 (엣지 적게 검출)
[save_image() × 3]
    │
    ▼
[cv2.Canny(gray, 50, 150)] → blur 없이 Canny
[save_image()] → 05_canny_without_blur.png
    │
    ▼
[edge_overlay = image.copy()]
[edge_overlay[edges_50_150 > 0] = (0, 0, 255)] → 엣지 위치를 빨간색으로 덮어쓰기
[save_image()] → 05_edge_overlay_red.png
    │
    ▼
[종료]
```

---

### 2. 예제 코드

```python
import os
import cv2
import numpy as np


def ensure_dir(path: str) -> None:
    os.makedirs(path, exist_ok=True)


def save_image(path: str, image) -> None:
    success = cv2.imwrite(path, image)
    if not success:
        raise IOError(f"이미지 저장 실패: {path}")


def create_edge_test_image() -> np.ndarray:
    """
    에지 검출 실습용 테스트 이미지를 생성합니다.
    실제 카메라 이미지 대신 도형과 선이 포함된 이미지를 만듭니다.
    """

    height = 480
    width = 640

    # 흰색 배경
    image = np.full((height, width, 3), 255, dtype=np.uint8)

    black = (0, 0, 0)
    red = (0, 0, 255)
    blue = (255, 0, 0)
    green = (0, 255, 0)
    gray = (180, 180, 180)

    # 큰 사각형
    cv2.rectangle(image, (60, 80), (240, 240), black, 3)

    # 채워진 원
    cv2.circle(image, (430, 160), 80, blue, -1)

    # 삼각형
    points = np.array([[320, 330], [460, 430], [200, 430]], np.int32)
    points = points.reshape((-1, 1, 2))
    cv2.polylines(image, [points], isClosed=True, color=red, thickness=4)
```

#### 삼각형

- 꼭짓점 배열 만들기 `np.array`

```
[[320, 330],   ← 위 꼭짓점
 [460, 430],   ← 오른쪽 아래
 [200, 430]]   ← 왼쪽 아래
```

- 형태 변환 `reshape((-1, 1, 2))`

```python
변환 전: shape (3, 2)
[[320, 330],
 [460, 430],
 [200, 430]]

# OpenCV 내부구조 (3차원 구조 필요)
# "3행 1열짜리 표인데, 각 칸에 숫자가 2개씩"
변환 후: shape (3, 1, 2)
[[[320, 330]],
 [[460, 430]],
 [[200, 430]]]

reshape(-1, 1, 2)에서 -1은 "자동으로 계산" 의미
→ 꼭짓점이 몇 개든 알아서 맞춰줌
```

- 선으로 연결 `cv2.polylines(img, pts, isClosed, color, thickness)`

| 인자 | 값 | 설명 |
|------|----|------|
| `img` | `image` | 그릴 대상 이미지 |
| `pts` | `[points]` | 꼭짓점 배열의 **리스트** (여러 다각형 가능) |
| `isClosed` | `True` / `False` | 닫힌 도형 / 열린 선 |
| `color` | `red` | 선 색상 (BGR) |
| `thickness` | `4` | 선 두께 |

```
isClosed=True:  점1→점2→점3→점1  (삼각형 완성)
isClosed=False: 점1→점2→점3      (열린 선)
```

---

```python
    # 로봇 주행 라인처럼 보이는 두 선
    cv2.line(image, (90, 460), (280, 280), green, 5)
    cv2.line(image, (550, 460), (360, 280), green, 5)

    # 약한 회색 선
    cv2.line(image, (20, 40), (620, 40), gray, 2)

    # 작은 노이즈 점 생성
    rng = np.random.default_rng(seed=42)

    for _ in range(200):
        x = int(rng.integers(0, width))
        y = int(rng.integers(0, height))
        color_value = int(rng.integers(0, 255))
        image[y, x] = (color_value, color_value, color_value)
```

| 함수 | 기능 |
|------|------|
| `np.random.default_rng(seed)` | 재현 가능한 난수 생성기 (seed 고정하면 매번 같은 순서로 나옴) |
| `rng.integers(low, high)` | low 이상 high 미만의 랜덤 정수 생성 |
| `image[y, x] = (r, g, b)` | 이미지 특정 좌표에 색상 설정 |

---

```python
    cv2.putText(
        image,
        "Canny Edge Test",
        (190, 40),
        cv2.FONT_HERSHEY_SIMPLEX,
        0.9,
        black,
        2,
    )

    return image

def main():
    print("=== Canny Edge Detection 실습 시작 ===")

    output_dir = "/workspace/outputs"
    ensure_dir(output_dir)

    # 1. 테스트 이미지 생성
    image = create_edge_test_image()

    original_path = os.path.join(output_dir, "05_original_edge_test.png")
    save_image(original_path, image)
    print(f"원본 이미지 저장: {original_path}")

    # 2. 흑백 변환
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)

    gray_path = os.path.join(output_dir, "05_gray.png")
    save_image(gray_path, gray)
    print(f"흑백 이미지 저장: {gray_path}")

    # 3. Gaussian Blur 적용
    # 커널 크기는 홀수여야 합니다. 예: 3, 5, 7
    blurred = cv2.GaussianBlur(gray, (5, 5), 0)

    blurred_path = os.path.join(output_dir, "05_blurred.png")
    save_image(blurred_path, blurred)
    print(f"Gaussian Blur 이미지 저장: {blurred_path}")
```

| 함수 | 기능 |
|---|---|
| `cv2.GaussianBlur(src, ksize, sigmaX)` | 이미지 색 공간 변환 |

- src: 입력 이미지 (흑백 또는 컬러 가능)
- ksize: 커널 크기 (홀수)
    - (3, 3) → 약한 블러  (노이즈 조금 제거)
    - (5, 5) → 중간 블러  (일반적으로 많이 사용)
    - (7, 7) → 강한 블러  (노이즈 많이 제거, 엣지도 뭉개짐)
- sigmaX: X 방향 표준 편차 (`0`이면 커널 크기로 자동 계산)

> [!Warning]
> 커널이 **반드시 홀수**여야 하는 이유: 커널의 중심 픽셀이 있어야 함  

> 왜 Canny 전에 Blur를 적용하는가?  

```
Blur 없이 Canny:
  노이즈 픽셀들도 픽셀값 차이가 크면 → 엣지로 오인식
  → 실제 엣지 외에 수많은 노이즈 엣지 검출

Blur 후 Canny:
  노이즈 픽셀들이 주변과 평균화되어 흐릿해짐
  → 노이즈에 의한 오인식 감소
  → 실제 경계선만 깔끔하게 검출
```

---

```python
    # 4. Canny Edge Detection 적용
    edges_50_150 = cv2.Canny(blurred, 50, 150)

    edges_50_150_path = os.path.join(output_dir, "05_canny_50_150.png")
    save_image(edges_50_150_path, edges_50_150)
    print(f"Canny 결과 저장 threshold 50/150: {edges_50_150_path}")

    # 5. threshold 값 변화 비교
    edges_30_100 = cv2.Canny(blurred, 30, 100)
    edges_100_200 = cv2.Canny(blurred, 100, 200)
    edges_150_250 = cv2.Canny(blurred, 150, 250)

    save_image(os.path.join(output_dir, "05_canny_30_100_more_sensitive.png"), edges_30_100)
    save_image(os.path.join(output_dir, "05_canny_100_200_standard.png"), edges_100_200)
    save_image(os.path.join(output_dir, "05_canny_150_250_less_sensitive.png"), edges_150_250)

    print("threshold 비교 결과 저장 완료")
```

| 함수 | 기능 |
|---|---|
| `cv2.Canny(image, threshold1, threshold2)` | 엣지 검출 |

- image: 입력 이미지 (흑백)
- threshold1: 약한 엣지 임계값 (이 값보다 낮은 픽셀은 무시)
- threshold2: 강한 엣지 임계값 (이 값보다 높은 픽셀은 무조건 엣지로 채택)
    - threshold1 < 픽셀 기울기 강도 < threshold2 → 인접한 확실한 엣지와 연결된 경우만 엣지
- 반환값: 엣지가 검출된 위치가 255, 나머지가 0인 흑백 이미지

---

```python
    # 6. Blur 없이 Canny 적용해서 비교
    edges_without_blur = cv2.Canny(gray, 50, 150)

    edges_without_blur_path = os.path.join(output_dir, "05_canny_without_blur.png")
    save_image(edges_without_blur_path, edges_without_blur)
    print(f"Blur 없이 Canny 적용 결과 저장: {edges_without_blur_path}")

    # 7. 컬러 원본 위에 에지 표시
    # edges는 흑백 이미지이므로 컬러 마스크처럼 활용합니다.
    edge_overlay = image.copy()
    edge_overlay[edges_50_150 > 0] = (0, 0, 255)    # 빨간색

    overlay_path = os.path.join(output_dir, "05_edge_overlay_red.png")
    save_image(overlay_path, edge_overlay)
    print(f"에지 오버레이 이미지 저장: {overlay_path}")

    print("=== Canny Edge Detection 실습 완료 ===")


if __name__ == "__main__":
    main()
```

---

### 3. Canny 동작 원리

Canny 알고리즘은 내부적으로 4단계로 동작합니다.

```
[1단계] Gaussian Blur (노이즈 제거)
    └─ 보통 Canny 호출 전에 미리 적용

[2단계] Gradient 계산 (픽셀값 변화율)
    └─ 픽셀값이 급격히 변하는 곳 = 경계선 후보
    └─ 각 픽셀마다 "변화 강도"와 "변화 방향" 계산

[3단계] Non-Maximum Suppression (엣지 얇게 만들기)
    └─ 변화 방향으로 가장 강한 픽셀만 남기고 나머지 제거
    └─ 두꺼운 엣지 → 1픽셀 두께의 날카로운 엣지

[4단계] Double Threshold + Hysteresis (이중 임계값 필터링)
    └─ threshold2 이상 → 확실한 엣지 (Strong Edge)
    └─ threshold1~2 사이 → 연결된 경우만 엣지 (Weak Edge)
    └─ threshold1 미만 → 제거
```