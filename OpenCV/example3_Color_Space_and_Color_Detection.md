# OpenCV

## 3번 예제

### 1. 전체 실행 흐름 (Flow)

```
[시작]
    │
    ▼
[np.full()] → 흰색 배경 이미지 생성 (400×600)
    │
    ▼
[cv2.rectangle() × 2] → 파란 사각형, 빨간 사각형
[cv2.circle()    × 2] → 초록 원, 노란 원
[cv2.putText()   × 4] → 색상 라벨 텍스트
    │
    ▼
[save_image()] → 03_original_bgr.png
    │
    ▼
[cv2.cvtColor BGR→RGB] → rgb_image 생성
[cv2.cvtColor RGB→BGR] → 저장을 위해 BGR로 재변환
[save_image()] → 03_rgb_converted_back_to_bgr.png
    │
    ▼
[cv2.cvtColor BGR→HSV] → hsv_image  ← 이하 모든 마스크 연산에 사용
    │
    ├─ [cv2.inRange()] → green_mask (초록 마스크)
    │  [cv2.bitwise_and()] → green_result
    │  [save_image()] → 03_green_mask.png / 03_green_result.png
    │
    ├─ [cv2.inRange()] → blue_mask (파란 마스크)
    │  [cv2.bitwise_and()] → blue_result
    │  [save_image()] → 03_blue_mask.png / 03_blue_result.png
    │
    └─ [cv2.inRange()] × 2 → red_mask_1, red_mask_2 (빨간 마스크 두 구간)
       [cv2.bitwise_or()] → red_mask (두 범위 합치기)
       [cv2.bitwise_and()] → red_result
       [save_image()] → 03_red_mask.png / 03_red_result.png
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


def main():
    print("=== 색상 공간 변환 및 색상 검출 실습 시작 ===")

    output_dir = "/workspace/outputs"
    ensure_dir(output_dir)

    # 1. 테스트 이미지 생성
    height = 400
    width = 600

    # 흰색 배경 이미지 생성
    image = np.full((height, width, 3), 255, dtype=np.uint8)
```

| 함수 | 기능 |
|---|---|
| `np.full(shape, fill_value, dtype)` | 주어진 shape과 dtype으로 모든 요소를 `fill_value`로 채운 배열 생성 |

- `shape`: (높이, 너비, 채널 수)
- `fill_value`: 채울 값. (255 = 흰색, 0 = 검은색)
- `dtype`: 데이터 타입. `np.uint8`는 0~255 범위의 8비트 부호 없는 정수

```python
    # OpenCV 색상은 BGR 순서입니다.
    blue_bgr = (255, 0, 0)
    green_bgr = (0, 255, 0)
    red_bgr = (0, 0, 255)
    yellow_bgr = (0, 255, 255)
    black_bgr = (0, 0, 0)

    # 파란 사각형
    cv2.rectangle(
        image,
        pt1=(50, 60),       # (x, y) 왼쪽 상단 꼭짓점 좌표
        pt2=(220, 180),     # (x, y) 오른쪽 하단 꼭짓점 좌표
        color=blue_bgr,
        thickness=-1,       # 내부를 채움
    )

    # 초록 원
    cv2.circle(
        image,
        center=(420, 120),
        radius=70,
        color=green_bgr,
        thickness=-1,
    )

    # 빨간 사각형
    cv2.rectangle(
        image,
        pt1=(70, 250),
        pt2=(250, 350),
        color=red_bgr,
        thickness=-1,
    )

    # 노란 원
    cv2.circle(
        image,
        center=(430, 300),
        radius=60,
        color=yellow_bgr,
        thickness=-1,
    )

    # 라벨 표시
    cv2.putText(image, "BLUE", (95, 130), cv2.FONT_HERSHEY_SIMPLEX, 0.8, black_bgr, 2)
    cv2.putText(image, "GREEN", (365, 130), cv2.FONT_HERSHEY_SIMPLEX, 0.8, black_bgr, 2)
    cv2.putText(image, "RED", (125, 315), cv2.FONT_HERSHEY_SIMPLEX, 0.8, black_bgr, 2)
    cv2.putText(image, "YELLOW", (370, 310), cv2.FONT_HERSHEY_SIMPLEX, 0.8, black_bgr, 2)

    original_path = os.path.join(output_dir, "03_original_bgr.png")
    save_image(original_path, image)
    print(f"원본 BGR 테스트 이미지 저장: {original_path}")

    # 2. BGR → RGB 변환
    rgb_image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)

    # 주의:
    # cv2.imwrite()는 BGR 기준으로 저장하는 것이 자연스럽습니다.
    # RGB 이미지를 그대로 cv2.imwrite()로 저장하면 색상이 뒤틀려 보일 수 있습니다.
    # 따라서 비교용으로만 사용하고, 저장할 때는 다시 BGR로 변환합니다.
    rgb_saved_as_bgr = cv2.cvtColor(rgb_image, cv2.COLOR_RGB2BGR)
    rgb_path = os.path.join(output_dir, "03_rgb_converted_back_to_bgr.png")
    save_image(rgb_path, rgb_saved_as_bgr)
    print(f"RGB 변환 후 다시 BGR로 저장: {rgb_path}")

    # 3. BGR → HSV 변환
    hsv_image = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)

    # HSV 이미지는 사람이 바로 보기 어렵습니다.
    # 실무에서는 HSV 자체보다 마스크 결과를 주로 확인합니다.
    print("BGR 이미지를 HSV 이미지로 변환 완료")
```

#### HSV vs BGR

BGR: 색상을 빛의 삼원색으로 표현
- (B, G, R) = (255, 0, 0) → 파랑
- 조명이 달라지면 같은 색도 B/G/R 값이 모두 바뀜 → 색상 검출 어려움

HSV: 색상을 사람의 인식 방식으로 표현
- H (Hue)        : 색조 (어떤 색?) → 0~179
    - H = 0 ~ 10 and H = 170 ~ 179 : 빨강
    - H = 20 ~ 35 : 노랑
    - H = 40 ~ 85 : 초록
    - H = 90 ~ 130 : 파랑
    - H = 130 ~ 160 : 보라
- S (Saturation) : 채도 (얼마나 선명?) → 0~255
- V (Value)      : 명도 (얼마나 밝?) → 0~255
- 조명이 달라져도 H(색조)는 거의 변하지 않음 → 색상 검출 용이

> OpenCV의 HSV에서 H 범위가 **0~179**인 이유:  
> 원래 Hue는 0°~360° 원형인데, `uint8` 최대값이 255이므로 360을 2로 나눠 0~179로 압축했습니다.

> H(Hue) 값이 0~10과 170~179 두 구간으로 나뉘는 이유는 H가 원형이기 때문입니다.
> H=0은 빨강이고, H=179도 빨강에 가깝습니다.
> 따라서 H를 검출할 때는 두 구간을 모두 고려해야 합니다.

```python
    # 4. 초록색 영역 검출
    # OpenCV HSV 범위:
    # H: 0~179, S: 0~255, V: 0~255
    lower_green = np.array([40, 80, 80])
    upper_green = np.array([85, 255, 255])

    green_mask = cv2.inRange(hsv_image, lower_green, upper_green)

    green_mask_path = os.path.join(output_dir, "03_green_mask.png")
    save_image(green_mask_path, green_mask)
    print(f"초록색 마스크 저장: {green_mask_path}")
```

| 함수 | 기능 |
|---|---|
| `cv2.inRange(src, lower_bound, upper_bound)` | 범위 내 픽셀만 추출 |

- `src`: 검사할 이미지 (HSV)
- `lower_bound`: HSV 하한값 `[H_min, S_min, V_min]`
- `upper_bound`: HSV 상한값 `[H_max, S_max, V_max]`
- 반환값: 마스크 이미지 (lower_bound ≤ src ≤ upper_bound 인 픽셀은 255(흰색), 아니면 0(검은색))

```python
    green_result = cv2.bitwise_and(image, image, mask=green_mask)

    green_result_path = os.path.join(output_dir, "03_green_result.png")
    save_image(green_result_path, green_result)
    print(f"초록색 검출 결과 저장: {green_result_path}")
```

| 함수 | 기능 |
|---|---|
| `cv2.bitwise_and(src1, src2, mask)` | AND 비트 연산 |

```python
# 내부 연산
for each pixel:
    if mask == 255:
        result = src1 AND src2
    else:
        result = 0
```

- `src1`, `src2`: 입력 이미지 (HSV) 또는 원본 이미지
- `mask`: 마스크 이미지 (0~255, 0이면 해당 픽셀은 연산에서 제외)
- 반환값: 결과 이미지 (마스크가 0인 부분은 원본 이미지의 해당 픽셀 유지, 255인 부분은 AND 연산 수행)

```python
    # 5. 파란색 영역 검출
    lower_blue = np.array([90, 80, 80])
    upper_blue = np.array([130, 255, 255])

    blue_mask = cv2.inRange(hsv_image, lower_blue, upper_blue)

    blue_mask_path = os.path.join(output_dir, "03_blue_mask.png")
    save_image(blue_mask_path, blue_mask)
    print(f"파란색 마스크 저장: {blue_mask_path}")

    blue_result = cv2.bitwise_and(image, image, mask=blue_mask)

    blue_result_path = os.path.join(output_dir, "03_blue_result.png")
    save_image(blue_result_path, blue_result)
    print(f"파란색 검출 결과 저장: {blue_result_path}")

    # 6. 빨간색 영역 검출
    # 빨강은 Hue 범위가 0 근처와 179 근처로 나뉩니다.
    lower_red_1 = np.array([0, 80, 80])
    upper_red_1 = np.array([10, 255, 255])

    lower_red_2 = np.array([170, 80, 80])
    upper_red_2 = np.array([179, 255, 255])

    red_mask_1 = cv2.inRange(hsv_image, lower_red_1, upper_red_1)
    red_mask_2 = cv2.inRange(hsv_image, lower_red_2, upper_red_2)

    red_mask = cv2.bitwise_or(red_mask_1, red_mask_2)

    red_mask_path = os.path.join(output_dir, "03_red_mask.png")
    save_image(red_mask_path, red_mask)
    print(f"빨간색 마스크 저장: {red_mask_path}")
```

| 함수 | 기능 |
|---|---|
| `cv2.bitwise_or(src1, src2, mask)` | OR 비트 연산 |

```python
# 내부 연산
for each pixel:
    if mask == 255:
        result = src1 OR src2
    else:
        result = 0
```

- `src1`, `src2`: 입력 이미지 (HSV) 또는 원본 이미지
- `mask`: 마스크 이미지 (0~255, 0이면 해당 픽셀은 연산에서 제외)
- 반환값: 결과 이미지 (마스크가 0인 부분은 원본 이미지의 해당 픽셀 유지, 255인 부분은 OR 연산 수행)

```python
    red_result = cv2.bitwise_and(image, image, mask=red_mask)

    red_result_path = os.path.join(output_dir, "03_red_result.png")
    save_image(red_result_path, red_result)
    print(f"빨간색 검출 결과 저장: {red_result_path}")

    print("=== 색상 공간 변환 및 색상 검출 실습 완료 ===")


if __name__ == "__main__":
    main()
```