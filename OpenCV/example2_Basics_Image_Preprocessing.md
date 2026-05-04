# OpenCV

## 2번 예제

### 1. 전체 실행 흐름 (Flow)

```
[프로그램 시작]
        │
        ▼
[경로 설정]
        ├─ input_path  = "/workspace/data/images/sample.jpg"
        └─ output_dir  = "/workspace/outputs"
        │
        ▼
[ensure_dir(output_dir)] → 출력 폴더 없으면 생성
        │
        ▼
[cv2.imread(input_path)] → 이미지 파일을 NumPy 배열로 읽기
        │
        ├─ image == None? → FileNotFoundError 발생 후 종료
        │
        ▼ (읽기 성공)
[image.shape 언패킹] → height, width, channels 추출 및 출력
        │
        ▼
[cv2.resize()] → 640×480으로 리사이즈
[cv2.imwrite()] → sample_resized_640x480.jpg 저장
        │
        ▼
[중앙 좌표 계산] → center_x, center_y
[경계값 처리] → max(), min()으로 이미지 밖으로 나가지 않게 보정
[NumPy 슬라이싱] → image[y1:y2, x1:x2]로 크롭
[cv2.imwrite()] → sample_center_crop.jpg 저장
        │
        ▼
[cv2.cvtColor()] → BGR → 흑백 변환
[cv2.imwrite()] → sample_gray.jpg 저장
        │
        ▼
[프로그램 종료]
```

---

### 2. 예제 코드

```python
import os
import cv2


def ensure_dir(path: str) -> None:
    os.makedirs(path, exist_ok=True)


def main():
    print("=== 실제 이미지 기본 처리 실습 시작 ===")

    input_path = "/workspace/data/images/sample.jpg"
    output_dir = "/workspace/outputs"

    ensure_dir(output_dir)

    # 1. 이미지 읽기
    image = cv2.imread(input_path)

    # 2. 이미지 읽기 실패 확인
    if image is None:
        raise FileNotFoundError(
            f"이미지를 읽을 수 없습니다: {input_path}\n"
            "확인 사항:\n"
            "1) data/images 폴더에 sample.jpg가 있는지 확인하세요.\n"
            "2) 파일명이 sample.jpg와 정확히 일치하는지 확인하세요.\n"
            "3) 확장자가 .jpeg, .png가 아닌지 확인하세요."
        )

> `cv2.imread()`는 파일이 없거나 경로가 잘못되어도 **예외(Exception)를 발생시키지 않고** 그냥 `None`을 반환합니다. 따라서 이후 코드에서 `image.shape`처럼 사용하면 `AttributeError`가 발생하므로, **읽은 직후 반드시 None 체크**를 해야 합니다.

    # 3. 이미지 크기 확인
    height, width, channels = image.shape

    print(f"원본 이미지 크기: width={width}, height={height}, channels={channels}")

    # 4. 리사이즈
    resized_width = 640
    resized_height = 480

    resized = cv2.resize(image, (resized_width, resized_height))

    resized_path = os.path.join(output_dir, "sample_resized_640x480.jpg")
    cv2.imwrite(resized_path, resized)

    print(f"리사이즈 결과 저장: {resized_path}")
```

| 함수 | 기능 |
| --- | --- |
| `cv2.resize(src, dsize)` | 이미지 리사이즈 |

- `src`: 원본 이미지
- `dsize`: 변경할 크기 `(width, height)` (튜플 형태)

> [!Warning]
> image.shape  →  (높이, 너비, 채널)  
> cv2.resize() →  (너비, 높이)

```python
    # 5. 중앙 영역 자르기
    # OpenCV/NumPy 배열 인덱싱은 [y1:y2, x1:x2] 순서입니다.
    crop_size = 300

    # 중앙 좌표 계산
    center_x = width // 2
    center_y = height // 2

    # 경계값 보정
    x1 = max(center_x - crop_size // 2, 0)
    y1 = max(center_y - crop_size // 2, 0)
    x2 = min(center_x + crop_size // 2, width)
    y2 = min(center_y + crop_size // 2, height)

    # 배열 슬라이싱
    cropped = image[y1:y2, x1:x2]

> y1: y 시작 좌표 / y2: y 끝 좌표  
> x1: x 시작 좌표 / x2: x 끝 좌표

> [!Warning]
> 슬라이싱은 `[행(y), 열(x)]` 순서** — `(x, y)` 좌표 순서와 반대입니다.

    cropped_path = os.path.join(output_dir, "sample_center_crop.jpg")
    cv2.imwrite(cropped_path, cropped)

    print(f"중앙 자르기 결과 저장: {cropped_path}")
    print(f"자른 영역 좌표: x1={x1}, y1={y1}, x2={x2}, y2={y2}")

    # 6. 흑백(1채널) 변환
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)

    gray_path = os.path.join(output_dir, "sample_gray.jpg")
    cv2.imwrite(gray_path, gray)

    print(f"흑백 변환 결과 저장: {gray_path}")

    print("=== 실제 이미지 기본 처리 실습 완료 ===")


if __name__ == "__main__":
    main()
```