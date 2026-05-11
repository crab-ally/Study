# OpenCV

## 6번 예제

### 1. 전체 실행 흐름 (Flow)

```
[시작]
    │
    ▼
[create_lane_test_image()]
    ├─ np.full((480,640,3), 70) → 어두운 회색 배경
    ├─ cv2.line() × 2  → 흰색/노란색 주행선 (검출 대상)
    ├─ cv2.line() × 3  → 회색 바닥 노이즈선
    ├─ cv2.line() × 2  → 빨간 상단 불필요한 선
    └─ cv2.line()      → 화면 중심선
    │
    ▼
[save_image()] → 06_original_lane_scene.png
    │
    ▼
[draw_roi_polygon(image)] → 원본에 초록 ROI 테두리 표시
[save_image()] → 06_roi_visualization.png
    │
    ▼
[cv2.cvtColor BGR→GRAY] → gray
[cv2.GaussianBlur (5,5)] → blurred
[cv2.Canny(50, 150)]     → edges (전체 에지)
[save_image() × 3]
    │
    ▼
[apply_roi_mask(edges)]
    ├─ np.zeros_like()  → 검은 마스크 생성
    ├─ cv2.fillPoly()   → 사다리꼴 영역을 흰색으로 채움
    └─ cv2.bitwise_and(edges, mask) → ROI 내부 에지만 추출
[save_image() × 2] → 06_roi_mask.png / 06_edges_roi_only.png
    │
    ▼
[detect_lines(roi_edges)]
    └─ cv2.HoughLinesP() → 직선 목록 반환
    │
    ▼
[draw_lines(image, lines)]
    ├─ line[0] 언패킹 → x1, y1, x2, y2
    └─ cv2.line() 으로 빨간선 표시
[save_image()] → 06_line_detection_result.png
    │
    ▼
[analyze_lane_direction(detected_lines, width)]
    └─ 각 선의 중심 x 평균 → error_x 계산 → 주행 판단 문자열 반환
    │
    ▼
[콘솔 출력] → 검출 라인 수, 주행 판단, 좌표 목록
    │
    ▼
[종료]
```

---

### 2. ROI (Region of Interest)

이미지의 관심 영역을 지정하여 해당 영역의 데이터만 처리하도록 하는 기술

#### 2-1. 사용 이유

- 불필요한 영역 제외 → 불필요한 데이터 처리 시간 감소(오검출 줄임, 처리 속도 증가)

---

### 3. 예제 코드

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


def create_lane_test_image() -> np.ndarray:
    """
    간단한 주행선 테스트 이미지를 생성합니다.
    실제 카메라 이미지 대신 코드로 직접 차선/라인이 있는 장면을 만듭니다.
    """

    height = 480
    width = 640

    # 어두운 회색(70) 배경: 바닥 느낌
    image = np.full((height, width, 3), 70, dtype=np.uint8)

    white = (255, 255, 255)
    yellow = (0, 255, 255)
    gray = (130, 130, 130)
    red = (0, 0, 255)
    black = (0, 0, 0)

    # 원근감 있는 주행선 두 개
    cv2.line(image, (160, height), (290, 270), white, 8)
    cv2.line(image, (500, height), (350, 270), yellow, 8)

    # 바닥 노이즈성 선들
    cv2.line(image, (40, 120), (250, 160), gray, 2)
    cv2.line(image, (390, 130), (620, 180), gray, 2)
    cv2.rectangle(image, (40, 300), (120, 360), gray, 2)

    # 화면 상단에 불필요한 선 추가
    cv2.line(image, (20, 50), (620, 50), red, 4)
    cv2.line(image, (80, 90), (560, 110), red, 3)

    cv2.putText(
        image,
        "ROI + Line Detection",
        (170, 40),
        cv2.FONT_HERSHEY_SIMPLEX,
        0.9,
        white,
        2,
    )

    # 화면 중심선
    cv2.line(image, (width // 2, 0), (width // 2, height), black, 1)

    return image


def apply_roi_mask(edges: np.ndarray) -> tuple[np.ndarray, np.ndarray]:
    """
    에지 이미지에 사다리꼴 ROI 마스크를 적용합니다.

    반환:
    - roi_edges: ROI 영역만 남긴 에지 이미지
    - roi_mask: ROI 영역이 흰색인 마스크 이미지
    """

    height, width = edges.shape

    # 검은 마스크 생성
    mask = np.zeros_like(edges)
```

| 함수 | 기능 |
|---|---|
| `np.zeros_like(array)` | 입력 배열과 동일한 shape, dtype을 가진 배열을 생성하되, 모든 값을 0(검정)으로 채움 |

```python
    # 사다리꼴 ROI 좌표 설정
    # 좌표 순서: 왼쪽 아래, 왼쪽 위, 오른쪽 위, 오른쪽 아래
    polygon = np.array(
        [
            [
                (80, height),
                (260, int(height * 0.55)),
                (380, int(height * 0.55)),
                (580, height),
            ]
        ],
        dtype=np.int32,
    )

    # ROI 영역을 흰색으로 채움
    cv2.fillPoly(mask, polygon, 255)
```

| 함수 | 기능 |
|---|---|
| `cv2.fillPoly(img, pts, color)` | 마스크에서 다각형 영역을 특정 색상으로 채움 |

- img: 원본 이미지
- pts: 꼭짓점 배열
- color: 채울 색상

```python
    # 에지 이미지와 마스크를 AND 연산
    roi_edges = cv2.bitwise_and(edges, mask)

    return roi_edges, mask


def detect_lines(roi_edges: np.ndarray):
    """
    ROI 내부 에지 이미지에서 직선을 검출합니다.
    """

    lines = cv2.HoughLinesP(
        roi_edges,
        rho=1,
        theta=np.pi / 180,
        threshold=45,
        minLineLength=40,
        maxLineGap=35,
    )

    return lines
```

#### 허프 변환 

`cv2.HoughLinesP(image, rho, theta, threshold, minLineLength=None, maxLineGap=None)`
- 이미지에서 직선(line)을 검출하는 함수

| 매개변수 | 설명 |
|----------|----|
| `image` | 흑백 에지 이미지 (Canny 결과) |
| `rho` | 거리 해상도 (픽셀 단위). 보통 1 사용 |
| `theta` | 각도 해상도 (라디안). `np.pi/180` = 1도 단위 |
| `threshold` | 직선으로 인정하는 최소 투표 수. **클수록 엄격** |
| `minLineLength` | 직선으로 인정하는 **최소 길이** (픽셀). 짧은 선 무시 |
| `maxLineGap` | 같은 직선으로 합칠 **최대 간격** (픽셀). 끊긴 선 연결 |

> 투표: 픽셀들이 이 선이 맞다고 찍어준 개수

##### 반환값

```python
lines = cv2.HoughLinesP(...)

# lines의 구조:
lines = [
    [[x1, y1, x2, y2]],   ← 첫 번째 직선
    [[x1, y1, x2, y2]],   ← 두 번째 직선
    ...
]

# lines.shape = (N, 1, 4)
# N: 검출된 직선 수
# 1: HoughLinesP의 고정 구조
# 4: x1, y1, x2, y2

# 검출된 선 없으면 None 반환
```

```python
def draw_lines(image: np.ndarray, lines) -> tuple[np.ndarray, list[tuple[int, int, int, int]]]:
    """
    검출된 선을 이미지 위에 표시합니다.
    """

    result = image.copy()
    detected_lines = []

    if lines is None:
        cv2.putText(
            result,
            "No lines detected",
            (30, 440),
            cv2.FONT_HERSHEY_SIMPLEX,
            0.8,
            (0, 0, 255),
            2,
        )
        return result, detected_lines

    for line in lines:
        x1, y1, x2, y2 = line[0]
        detected_lines.append((x1, y1, x2, y2))

        cv2.line(
            result,
            (x1, y1),
            (x2, y2),
            (0, 0, 255),
            4,
        )

    cv2.putText(
        result,
        f"Detected lines: {len(detected_lines)}",
        (30, 440),
        cv2.FONT_HERSHEY_SIMPLEX,
        0.8,
        (255, 255, 255),
        2,
    )

    return result, detected_lines


def draw_roi_polygon(image: np.ndarray) -> np.ndarray:
    """
    원본 이미지 위에 ROI 영역을 표시합니다.
    """

    result = image.copy()
    height, width = result.shape[:2]

    # 사다리꼴 꼭짓점 좌표
    polygon = np.array(
        [
            [
                (80, height),           # 왼쪽 아래
                (260, int(height * 0.55)),  # 왼쪽 위
                (380, int(height * 0.55)),  # 오른쪽 위
                (580, height),          # 오른쪽 아래
            ]
        ],
        dtype=np.int32,
    )
```

> 카메라에서 도로를 찍으면 **원근감** 때문에 차선이 멀어질수록 좁아져 보입니다.  
> 그래서 ROI도 위쪽이 좁고 아래쪽이 넓은 사다리꼴로 잡아야 차선을 놓치지 않습니다.

```python
    cv2.polylines(
        result,
        polygon,
        isClosed=True,
        color=(0, 255, 0),
        thickness=3,
    )

    return result


def analyze_lane_direction(detected_lines, image_width: int) -> str:
    """
    검출된 선들의 대략적인 위치를 이용해 간단한 주행 판단을 수행합니다.

    실제 자율주행 알고리즘은 훨씬 복잡하지만,
    초급 과정에서는 선들의 평균 x 위치로 개념만 이해합니다.
    """

    if not detected_lines:
        return "라인 미검출: 정지 또는 저속 탐색 필요"

    x_centers = []

    for x1, y1, x2, y2 in detected_lines:
        x_centers.append((x1 + x2) / 2)

    avg_x = sum(x_centers) / len(x_centers)
    screen_center_x = image_width / 2
    error_x = avg_x - screen_center_x

    if error_x < -40:
        return f"라인 중심이 왼쪽: 좌측 보정 필요, error_x={error_x:.1f}"
    elif error_x > 40:
        return f"라인 중심이 오른쪽: 우측 보정 필요, error_x={error_x:.1f}"
    else:
        return f"라인이 중앙 근처: 직진 가능, error_x={error_x:.1f}"


def main():
    print("=== ROI 관심 영역 및 라인 검출 실습 시작 ===")

    output_dir = "/workspace/outputs"
    ensure_dir(output_dir)

    # 1. 테스트 이미지 생성
    image = create_lane_test_image()
    height, width = image.shape[:2]

    original_path = os.path.join(output_dir, "06_original_lane_scene.png")
    save_image(original_path, image)
    print(f"원본 주행선 이미지 저장: {original_path}")

    # 2. ROI 표시 이미지 저장
    roi_visual = draw_roi_polygon(image)
    roi_visual_path = os.path.join(output_dir, "06_roi_visualization.png")
    save_image(roi_visual_path, roi_visual)
    print(f"ROI 시각화 이미지 저장: {roi_visual_path}")

    # 3. Gray 변환
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    gray_path = os.path.join(output_dir, "06_gray.png")
    save_image(gray_path, gray)
    print(f"Gray 이미지 저장: {gray_path}")

    # 4. Gaussian Blur
    blurred = cv2.GaussianBlur(gray, (5, 5), 0)
    blurred_path = os.path.join(output_dir, "06_blurred.png")
    save_image(blurred_path, blurred)
    print(f"Blur 이미지 저장: {blurred_path}")

    # 5. Canny Edge
    edges = cv2.Canny(blurred, 50, 150)
    edges_path = os.path.join(output_dir, "06_edges_full.png")
    save_image(edges_path, edges)
    print(f"전체 에지 이미지 저장: {edges_path}")

    # 6. ROI 마스크 적용
    roi_edges, roi_mask = apply_roi_mask(edges)

    roi_mask_path = os.path.join(output_dir, "06_roi_mask.png")
    save_image(roi_mask_path, roi_mask)
    print(f"ROI 마스크 저장: {roi_mask_path}")

    roi_edges_path = os.path.join(output_dir, "06_edges_roi_only.png")
    save_image(roi_edges_path, roi_edges)
    print(f"ROI 내부 에지 저장: {roi_edges_path}")

    # 7. HoughLinesP로 라인 검출
    lines = detect_lines(roi_edges)

    # 8. 검출 라인 시각화
    result, detected_lines = draw_lines(image, lines)

    result_path = os.path.join(output_dir, "06_line_detection_result.png")
    save_image(result_path, result)
    print(f"라인 검출 결과 저장: {result_path}")

    # 9. 주행 판단
    decision = analyze_lane_direction(detected_lines, width)

    print(f"검출된 라인 수: {len(detected_lines)}")
    print(f"간단 주행 판단: {decision}")

    print("검출 라인 좌표 일부:")
    for idx, line in enumerate(detected_lines[:10], start=1):
        x1, y1, x2, y2 = line
        print(f"  line {idx}: x1={x1}, y1={y1}, x2={x2}, y2={y2}")
```

| 함수 | 기능 |
|---|---|---|
| `enumerate(iterable, start=0)` | 반복 가능한 객체를 돌 때 인덱스와 값을 같이 꺼내주는 함수 |

- start: 인덱스 시작값

```python
    print("=== ROI 관심 영역 및 라인 검출 실습 완료 ===")


if __name__ == "__main__":
    main()
```