# OpenCV

## 1번 예제

### 1. 전체 실행 흐름 (Flow)

```
[시작]
    │
    ▼
[create_test_image()]
    ├─ np.full() → 흰색 배경 (480×640)
    ├─ cv2.circle()    → 초록 큰 원 (검출 대상)
    ├─ cv2.rectangle() → 빨간 사각형 (검출 대상 아님)
    ├─ cv2.circle()    → 파란 원 (검출 대상 아님)
    ├─ cv2.circle() × 3 → 작은 초록 노이즈들
    ├─ cv2.line() × 2  → 화면 중앙 기준선
    └─ cv2.putText()   → 안내 텍스트
    │
    ▼
[save_image()] → 04_original_scene.png
    │
    ▼
[detect_green_object(image)]
    ├─ cv2.cvtColor(BGR→HSV)
    ├─ cv2.inRange() → 초록 마스크 (raw)
    ├─ cv2.morphologyEx(MORPH_OPEN)  → 작은 노이즈 제거
    ├─ cv2.morphologyEx(MORPH_CLOSE) → 객체 내부 채우기
    ├─ cv2.findContours() → 윤곽선 목록
    ├─ contourArea 필터 (min_area=1000) → 작은 윤곽선 제거
    ├─ max(contours, key=contourArea)  → 가장 큰 윤곽선 선택
    ├─ cv2.boundingRect() → x, y, w, h
    └─ center_x, center_y 계산 → object_info 딕셔너리 반환
    │
    ▼
[save_image()] → 04_green_mask_raw.png
[save_image()] → 04_green_mask_clean.png
    │
    ▼
[draw_detection_result(image, object_info)]
    ├─ image.copy() → 원본 보호
    ├─ cv2.rectangle() → 바운딩 박스
    ├─ cv2.circle()    → 객체 중심점
    ├─ cv2.line()      → 화면 중심 → 객체 중심 연결선
    └─ cv2.putText() × 3 → 좌표/오차/로봇 판단 텍스트
    │
    ▼
[save_image()] → 04_detection_result.png
    │
    ▼
[콘솔 출력] → 면적, 바운딩 박스, 중심 좌표, 로봇 판단
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


def create_test_image() -> np.ndarray:
    """
    실습용 테스트 이미지를 생성합니다.
    실제 카메라 영상 대신 코드에서 직접 이미지를 만들어 사용합니다.
    """

    height = 480
    width = 640

    # 흰색 배경 생성
    image = np.full((height, width, 3), 255, dtype=np.uint8)

    # OpenCV는 BGR 순서입니다.
    red_bgr = (0, 0, 255)
    green_bgr = (0, 255, 0)
    blue_bgr = (255, 0, 0)
    black_bgr = (0, 0, 0)

    # 검출 대상: 초록색 큰 원
    cv2.circle(
        image,
        center=(420, 260),
        radius=70,
        color=green_bgr,
        thickness=-1,
    )

    # 검출 대상이 아닌 빨간 사각형
    cv2.rectangle(
        image,
        pt1=(70, 80),   # (x, y) 왼쪽 상단 꼭짓점 좌표
        pt2=(200, 200), # (x, y) 오른쪽 하단 꼭짓점 좌표
        color=red_bgr,
        thickness=-1,
    )

    # 검출 대상이 아닌 파란 원
    cv2.circle(
        image,
        center=(180, 360),
        radius=55,
        color=blue_bgr,
        thickness=-1,
    )

    # 작은 초록색 노이즈들
    cv2.circle(image, center=(80, 420), radius=8, color=green_bgr, thickness=-1)
    cv2.circle(image, center=(580, 70), radius=6, color=green_bgr, thickness=-1)
    cv2.circle(image, center=(550, 390), radius=10, color=green_bgr, thickness=-1)

    # 화면 중앙 기준선 표시
    cv2.line(image, (width // 2, 0), (width // 2, height), black_bgr, 1)    # 수직선
    cv2.line(image, (0, height // 2), (width, height // 2), black_bgr, 1)   # 수평선
```

| 함수 | 기능 |
|------|------|
| `cv2.line` | 이미지에 선 추가 |

- image: 이미지
- pt1: 시작점 좌표 `(x, y)`
- pt2: 끝점 좌표 `(x, y)`
- color: 선 색상
- thickness: 선 두께

---

```python
    cv2.putText(
        image,
        "Target: GREEN Object",
        (20, 40),                   # 텍스트 시작점 (왼쪽 하단 기준)
        cv2.FONT_HERSHEY_SIMPLEX,
        0.9,
        black_bgr,
        2,
    )

    return image


def detect_green_object(image: np.ndarray):
    """
    초록색 객체를 검출하고 가장 큰 객체의 정보를 반환합니다.
    """

    # 1. BGR → HSV 변환
    hsv = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)

    # 2. 초록색 HSV 범위 설정
    lower_green = np.array([40, 80, 80])
    upper_green = np.array([85, 255, 255])

    # 3. 초록색 마스크 생성
    mask = cv2.inRange(hsv, lower_green, upper_green)

    # 4. 노이즈 완화를 위한 Morphology 연산
    # 작은 점을 줄이고 객체 영역을 더 안정적으로 만듭니다.
    kernel = np.ones((5, 5), np.uint8)

    mask_clean = cv2.morphologyEx(mask, cv2.MORPH_OPEN, kernel)
    mask_clean = cv2.morphologyEx(mask_clean, cv2.MORPH_CLOSE, kernel)
```

| 함수 | 기능 |
|---|---|
| `np.ones((rows, cols), dtype)` | 모든 값이 1로 채워진 배열(행렬) 생성 |
| `cv2.morphologyEx(src, op, kernel)` | 이미지에서 형태(모양)을 바꾸는 연산 (노이즈 제거) |

- src: 입력 마스크
- op: 연산 종류
    - Opening: cv2.MORPH_OPEN (침식 → 팽창, 작은 노이즈 제거에 효과적)
    - Closing: cv2.MORPH_CLOSE (팽창 → 침식, 객체 내부 구멍 채우기에 효과적)
- kernel: 연산에 사용할 커널

#### 커널(kernel)

이미지 위에 올려서 주변 픽셀을 보고 계산하는 작은 필터 행렬 (Morphology 연산의 "브러시" 역할)

- 커널이 클수록 → 연산 효과가 강해짐
- 커널이 작을수록 → 연산 효과가 약해짐

---

```python
    # 5. 윤곽선 찾기
    contours, _ = cv2.findContours(
        mask_clean,
        cv2.RETR_EXTERNAL,
        cv2.CHAIN_APPROX_SIMPLE,
    )

    if not contours:
        return mask, mask_clean, None
```

| 함수 | 기능 |
|---|---|
| `cv2.findContours(image, mode, method)` | 이미지에서 윤곽선 찾기 |

- image: 흑백 마스크 이미지 (컬러 이미지 불가)
- mode: 윤곽선 검색 모드
    - `cv2.RETR_EXTERNAL`: 가장 바깥쪽 윤곽선만 검출 (겹친 객체가 있을 때 유용)
    - `cv2.RETR_LIST`: 모든 윤곽선을 검출 (계층 구조 무시)
    - `cv2.RETR_TREE`: 모든 윤곽선을 계층 구조로 검출
- method: 윤곽선 근사 방법
    - `cv2.CHAIN_APPROX_SIMPLE`: 꼭짓점만 저장 (메모리 효율적)
    - `cv2.CHAIN_APPROX_NONE`: 모든 점을 저장
- 반환값: `contours, hierarchy`
    - contours: 윤곽선 좌표 목록 (리스트)
    - hierarchy: 윤곽선 계층 구조 정보 (필요 없으면 무시)

---

```python
    # 6. 작은 노이즈 제거
    min_area = 1000
    valid_contours = []

    for contour in contours:
        area = cv2.contourArea(contour)

        if area >= min_area:
            valid_contours.append(contour)

    if not valid_contours:
        return mask, mask_clean, None
```

| 함수 | 기능 |
|---|---|
| `cv2.contourArea(contour)` | 윤곽선이 감싸는 영역의 면적 계산 |

- contour: 윤곽선 좌표 목록

---

```python
    # 7. 가장 큰 객체 선택
    largest_contour = max(valid_contours, key=cv2.contourArea)  # `key=` 비교기준 설정

    # 8. 바운딩 박스 계산
    x, y, w, h = cv2.boundingRect(largest_contour)
```

| 함수 | 기능 |
|---|---|
| `cv2.boundingRect(contour)` | 윤곽선을 감싸는 최소 크기의 사각형 좌표 반환 |

- contour: 윤곽선 좌표 목록
- 반환값: `(x, y, w, h)`
    - x, y: 사각형의 왼쪽 상단 꼭짓점 좌표
    - w, h: 사각형의 너비와 높이

---

```python
    # 9. 중심 좌표 계산
    center_x = x + w // 2
    center_y = y + h // 2

    area = cv2.contourArea(largest_contour)

    object_info = {
        "contour": largest_contour,
        "area": area,
        "x": x,
        "y": y,
        "w": w,
        "h": h,
        "center_x": center_x,
        "center_y": center_y,
    }

    return mask, mask_clean, object_info


def draw_detection_result(image: np.ndarray, object_info):
    """
    검출 결과를 이미지 위에 시각화합니다.
    """

    # 원본 이미지 보호를 위해 복사
    result = image.copy()

    height, width = result.shape[:2]    # 채널 수 무시
    screen_center_x = width // 2
    screen_center_y = height // 2

    if object_info is None:
        cv2.putText(
            result,
            "No green object detected",
            (30, 80),
            cv2.FONT_HERSHEY_SIMPLEX,
            0.8,
            (0, 0, 255),
            2,
        )
        return result

    x = object_info["x"]
    y = object_info["y"]
    w = object_info["w"]
    h = object_info["h"]
    center_x = object_info["center_x"]
    center_y = object_info["center_y"]
    area = object_info["area"]

    # 바운딩 박스 그리기
    cv2.rectangle(
        result,
        (x, y),
        (x + w, y + h),
        (0, 0, 0),
        3,
    )

    # 객체 중심점 그리기
    cv2.circle(
        result,
        (center_x, center_y),
        8,
        (0, 0, 255),
        -1,
    )

    # 화면 중심에서 객체 중심까지 선 그리기
    cv2.line(
        result,
        (screen_center_x, screen_center_y),
        (center_x, center_y),
        (255, 0, 255),
        2,
    )

    # 중심 오차 계산
    error_x = center_x - screen_center_x
    error_y = center_y - screen_center_y

    # 정보 텍스트 출력
    info_text_1 = f"center=({center_x}, {center_y}) area={area:.1f}"
    info_text_2 = f"error_x={error_x}, error_y={error_y}"

    cv2.putText(
        result,
        info_text_1,
        (30, 420),
        cv2.FONT_HERSHEY_SIMPLEX,
        0.7,
        (0, 0, 0),
        2,
    )

    cv2.putText(
        result,
        info_text_2,
        (30, 455),
        cv2.FONT_HERSHEY_SIMPLEX,
        0.7,
        (0, 0, 0),
        2,
    )

    # 로봇 제어 관점의 간단한 판단
    if error_x < -50:
        command = "Object is LEFT -> turn left"
    elif error_x > 50:
        command = "Object is RIGHT -> turn right"
    else:
        command = "Object is CENTER -> go forward"

> ±50 픽셀의 여유(dead zone)를 두는 이유:  
> 오차가 매우 작을 때도 계속 좌우로 흔들리는 것을 방지

    cv2.putText(
        result,
        command,
        (30, 80),
        cv2.FONT_HERSHEY_SIMPLEX,
        0.8,
        (0, 0, 0),
        2,
    )

    return result


def main():
    print("=== 윤곽선 검출 및 중심 좌표 계산 실습 시작 ===")

    output_dir = "/workspace/outputs"
    ensure_dir(output_dir)

    # 1. 테스트 이미지 생성
    image = create_test_image()

    original_path = os.path.join(output_dir, "04_original_scene.png")
    save_image(original_path, image)
    print(f"원본 테스트 이미지 저장: {original_path}")

    # 2. 초록색 객체 검출
    mask, mask_clean, object_info = detect_green_object(image)

    mask_path = os.path.join(output_dir, "04_green_mask_raw.png")
    save_image(mask_path, mask)
    print(f"초록색 원본 마스크 저장: {mask_path}")

    mask_clean_path = os.path.join(output_dir, "04_green_mask_clean.png")
    save_image(mask_clean_path, mask_clean)
    print(f"초록색 정제 마스크 저장: {mask_clean_path}")

    # 3. 검출 결과 시각화
    result = draw_detection_result(image, object_info)

    result_path = os.path.join(output_dir, "04_detection_result.png")
    save_image(result_path, result)
    print(f"검출 결과 이미지 저장: {result_path}")

    # 4. 콘솔에 객체 정보 출력
    if object_info is None:
        print("초록색 객체를 찾지 못했습니다.")
    else:
        print("초록색 객체 검출 성공")
        print(f"면적: {object_info['area']:.1f}")
        print(f"바운딩 박스: x={object_info['x']}, y={object_info['y']}, w={object_info['w']}, h={object_info['h']}")
        print(f"중심 좌표: center_x={object_info['center_x']}, center_y={object_info['center_y']}")

        height, width = image.shape[:2]
        screen_center_x = width // 2
        error_x = object_info["center_x"] - screen_center_x

        print(f"화면 중심 기준 x 오차: {error_x}")

        if error_x < -50:
            print("판단: 객체가 화면 왼쪽에 있습니다. 로봇은 왼쪽으로 회전해야 합니다.")
        elif error_x > 50:
            print("판단: 객체가 화면 오른쪽에 있습니다. 로봇은 오른쪽으로 회전해야 합니다.")
        else:
            print("판단: 객체가 화면 중앙에 있습니다. 로봇은 전진 또는 집기 동작을 수행할 수 있습니다.")

    print("=== 윤곽선 검출 및 중심 좌표 계산 실습 완료 ===")


if __name__ == "__main__":
    main()
```