# OpenCV

## 1번 예제

### 1. 전체 실행 흐름 (Flow)

```
[프로그램 시작]
        │
        ▼
[라이브러리 import] ─── os, cv2, numpy
        │
        ▼
[main() 호출]
        │
        ├─ 1. OpenCV / NumPy 버전 출력
        │
        ├─ 2. 저장 경로 설정
        │       ├─ /workspace/data/images  (입력 이미지 저장용)
        │       └─ /workspace/outputs      (결과 이미지 저장용)
        │
        ├─ 3. ensure_dir() 호출 → 폴더 없으면 생성
        │
        ├─ 4. np.zeros() → 300×500 검은색 이미지 생성 (메모리 상)
        │
        ├─ 5. 도형/텍스트 그리기
        │       ├─ cv2.rectangle() → 파란색 사각형
        │       ├─ cv2.circle()    → 초록색 원 (꽉 채움)
        │       └─ cv2.putText()   → 흰색 텍스트
        │
        ├─ 6. cv2.imwrite() → 테스트 이미지를 파일로 저장
        │
        ├─ 7. cv2.imread() → 저장한 파일을 다시 읽기
        │
        ├─ 8. cv2.cvtColor() → 컬러 → 흑백 변환
        │
        └─ 9. cv2.imwrite() → 흑백 이미지를 outputs 폴더에 저장

[프로그램 종료]
```

---

### 2. 예제 코드

```python
import os           # 파일 시스템 조작 (경로 생성, 경로 결합 등)
import cv2          # OpenCV 라이브러리. 이미지 읽기/쓰기/변환/도형 그리기 담당
import numpy as np  # 수치 배열 처리. OpenCV는 이미지를 NumPy 배열로 다룸


def ensure_dir(path: str) -> None:
    """
    폴더가 없으면 생성하는 함수입니다.
    Docker 볼륨 연결 상태에서도 안전하게 사용할 수 있습니다.
    """
    os.makedirs(path, exist_ok=True)

> `os.mkdir`과의 차이: `makedirs`는 중간 경로도 한번에 생성합니다.

def main():
    print("=== OpenCV Docker Lab 실행 테스트 ===")

    # 1. 버전 확인
    print(f"OpenCV version: {cv2.__version__}")
    print(f"NumPy version: {np.__version__}")

    # 2. 컨테이너 내부 기준 경로 설정
    image_dir = "/workspace/data/images"
    output_dir = "/workspace/outputs"

    ## `image_dir`와 `output_dir` 경로에 폴더가 없으면 생성합니다.
    ensure_dir(image_dir)
    ensure_dir(output_dir)

    # 3. 테스트 이미지 생성
    height = 300
    width = 500
    image = np.zeros((height, width, 3), dtype=np.uint8)    # 300x500 크기의 검은색 이미지 생성
```

| 함수 | 기능 |
|---|---|
| `np.zeros(shape, dtype)` | 모든 값을 0으로 초기화한 배열 생성 (픽셀값 0 = 검정색) |

- `shape`: (높이, 너비, 채널 수)
- `dtype`: 데이터 타입. 0~255 범위의 8비트 정수 (픽셀 값에 사용)

---

```python
    # 4. 이미지 위에 도형과 글자 그리기
    # OpenCV 색상 순서는 RGB가 아니라 BGR입니다.
    cv2.rectangle(
        image,
        pt1=(50, 50),       # (x, y) 왼쪽 상단 꼭짓점 좌표
        pt2=(450, 250),     # (x, y) 오른쪽 하단 꼭짓점 좌표
        color=(255, 0, 0),
        thickness=3,        # 선의 두께(픽셀 단위). `-1'이면 내부를 채움
    )

    cv2.circle(
        image,
        center=(250, 150),  # 원의 중심 좌표
        radius=60,          # 원의 반지름
        color=(0, 255, 0),
        thickness=-1,       # 원 내부를 꽉 채움.
    )

> `thickness=-1`은 OpenCV에서 내부 채우기를 의미하는 특수값

    cv2.putText(
        image,
        text="OpenCV Docker Test",          # 출력할 문자열
        org=(80, 285),                      # 텍스트 시작점 (왼쪽 하단 기준)
        fontFace=cv2.FONT_HERSHEY_SIMPLEX,  # 폰트 종류
        fontScale=0.8,                      # 폰트 크기 배율 (1.0 기본)
        color=(255, 255, 255),
        thickness=2,
    )

    # 5. 원본 테스트 이미지 저장
    input_image_path = os.path.join(image_dir, "opencv_test_input.png")
    cv2.imwrite(input_image_path, image)
    print(f"테스트 이미지 저장 완료: {input_image_path}")
```

| 함수 | 기능 |
|---|---|
| `os.path.join(path, *paths)` | OS에 맞는 구분자로 경로 연결 |
| `cv2.imwrite(filename, img)` | 이미지 파일로 저장 |

- `filename`: 저장할 파일 경로
- `img`: 저장할 이미지 (NumPy 배열)
- 반환값: 성공하면 `True`, 실패하면 `False`

---

```python
    # 6. OpenCV로 이미지 다시 읽기
    loaded_image = cv2.imread(input_image_path)

    if loaded_image is None:
        raise FileNotFoundError(f"이미지를 읽을 수 없습니다: {input_image_path}")

    print(f"이미지 읽기 성공: shape={loaded_image.shape}")  # `.shape` : 이미지의 크기 (높이, 너비, 채널 수)를 반환
```

| 함수 | 기능 |
|---|---|
| `cv2.imread(filename, flags=cv2.IMREAD_COLOR)` | 이미지 파일 읽기 |

- `filename`: 읽을 파일 경로
- `flags`: 읽기 방식 플래그
    - `cv2.IMREAD_COLOR` : `1` :  컬러 이미지 (기본값)
    - `cv2.IMREAD_GRAYSCALE` : `0` : 흑백 이미지
    - `cv2.IMREAD_UNCHANGED` : `-1` :알파 채널 포함 (PNG 등)
- 반환값: 실패하면 `None`

---

```python
    # 7. 흑백 변환
    gray_image = cv2.cvtColor(loaded_image, cv2.COLOR_BGR2GRAY)
```

| 함수 | 기능 |
|---|---|
| `cv2.cvtColor(src, code)` | 이미지 색상 공간 변환 |

- `src` : 변환할 원본 이미지 (NumPy 배열)
- `code` : 변환 코드
  - `cv2.COLOR_BGR2GRAY` : BGR → 흑백(채널 없음)
  - `cv2.COLOR_BGR2RGB` : BGR → RGB
  - `cv2.COLOR_BGR2HSV` : BGR → HSV
  - `cv2.COLOR_GRAY2BGR` : 흑백 → 컬러(3채널로 확장)

---

```python
    # 8. 결과 저장
    output_image_path = os.path.join(output_dir, "opencv_test_gray.png")
    cv2.imwrite(output_image_path, gray_image)

    print(f"흑백 변환 결과 저장 완료: {output_image_path}")
    print("=== 테스트 완료 ===")


if __name__ == "__main__":
    main()
```
