# Python

## 1. Python 코드 기초

### 1.1 출력문

| 형식 | 코드 |
|------|------|
| 기본형 | `print(value)` |
| 구분자 변경 | `print(v1, v2, sep="문자")` |
| 끝 문자 변경 | `print(v, end="문자")` |
| 문자열 포맷팅 (version 3.6+) | `print(f"...{변수명}...")` |

**정렬 옵션**

```python
f"{변수:<자릿수}"   # 왼쪽 정렬
f"{변수:^자릿수}"   # 가운데 정렬
f"{변수:>자릿수}"   # 오른쪽 정렬
```

### 1.2 주석

```python
# 한 줄 주석

"""
여러 줄 주석
"""
```

### 1.3 자료형 확인

`type(변수)`

---

## 2. 조건문과 반복문

### 2.1 조건문

#### if 문

```python
if 조건:
    statement
elif 조건:
    statement
else:
    statement
```

#### 삼항 연산자

```python
A if 조건 else B    # 조건이 참이면 A 거짓이면 B
```

### 2.2 반복문

#### for 문

```python
for 변수 in 반복대상:
    statement
```

#### while 문

```python
while 조건:
    statement
```

### 2.3 break / continue / pass

| 키워드 | 설명 |
|--------|------|
| `break` | 반복문 완전 탈출 |
| `continue` | 현재 반복 건너뛰기 |
| `pass` | 아무것도 하지 않음 (뼈대 작성 시 사용) |

### 2.4 range() 함수

```python
range(start, stop, step)
# start 기본값: 0 / step 기본값: 1
# stop 값은 결과에 포함되지 않음
```

### 2.5 논리 연산자

| 연산자 | 설명 |
|---|---|
| `x and y` | x와 y 모두 참이어야 참 |
| `x or y` | x와 y 둘 중 하나만 참이어도 참 |
| `not x` | x가 거짓이면 참 |

---

## 3. 리스트 / 튜플 / 딕셔너리 / set

### 3.1 리스트

```python
lst = [10, 20, 30]

len(lst)            # 길이
lst[index]          # 접근
lst.append(value)   # 추가
lst[index] = value  # 변경
lst.remove(value)   # 삭제
```

### 3.2 튜플

```python
pos = (10, 20)      # 불변 (값 변경 불가)
single = (9,)       # 요소 1개일 때 쉼표 필수

len(pos)            # 길이
pos[index]          # 접근
pos + (30, 40)      # 이어붙이기
```

### 3.3 딕셔너리

```python
robot = {"name": "turtlebot", "battery": 90}

# 길이
len(robot)

# 접근
robot["name"]               # KeyError 오류 : key가 없을 때
robot.get("name")           # 안전 접근 : 없으면 None 반환
robot.get("name", "기본값")  # key가 없을 때 기본값 반환

# 키·값 순회
robot.keys()    # 키 목록 (반환타입: dict_keys)
robot.values()  # 값 목록 (반환타입: dict_values)
robot.items()   # (key, value) 튜플 쌍 (반환타입: dict_items)

# 추가 / 변경
robot["speed"] = 1.2

# 삭제
del robot["speed"]      # 반환값 없음
robot.pop("speed")      # 삭제된 value 반환
robot.popitem()         # 삭제된 마지막 (key, value) 반환
```

### 3.4 set

```python
s = {1, 2, 3, 4}
# 중복 제거, 순서 없음 → 인덱스 접근 불가
```

---

## 4. 함수

```python
def 함수명(매개변수1, 매개변수2):
    statement
    return 반환값
```

> 함수 안 변수는 밖에서 접근 불가능

### 4.1 매개변수

```python
def f(a=1, b=2):      # 기본값 지정
    pass

''' 가변인자: 함수에 들어오는 인자의 개수가 정해져 있지 않은 것 '''
def f(*args):          # 가변인자 → 튜플로 묶임
    pass

def f(**kwargs):       # 키워드 가변인자 → 딕셔너리로 묶임
    pass
```

### 4.2 반환값

```python
return value        # 단일 반환
return v1, v2       # 다중 반환 → 튜플로 반환
```

### 4.3 타입 힌트

1. **반환타입**

"리턴값을 특정 자료형으로 반환하겠다"는 의미

```python
def main() -> int:
    return 0
```

2. **매개변수**

해당 변수의 타입을 표시

```python
def print_info(name: str, age: int) -> None:
    print(f"Name: {name}, Age: {age}")
```

---

## 5. 문자열 처리와 파일 기초

### 5.1 문자열 메서드

```python
s.lower()           # 소문자 변환
s.upper()           # 대문자 변환
s.replace(old, new) # 문자열 대체
s.strip()           # 앞뒤 공백 제거
s.find(sub)         # 부분 문자열 위치 탐색
```

### 5.2 문자열 전송

네트워크/파일 전송은 문자열(byte) 기반이라서 리스트 같은 구조를 그대로 못 보냄

| 함수 | 기능 | 설명 |
|---|---|---|
| `separator.join(string_list)` | 리스트 → 문자열 | 리스트의 요소들을 separator로 이어서 하나의 문자열로 만든다 |
| `string.split(separator)` | 문자열 → 리스트 | 문자열을 separator 기준으로 분리하여 리스트로 만든다 |

```python
lst = ["apple", "banana", "cherry"]
s = ",".join(lst)   # apple,banana,cherry

s = "apple,banana,cherry"
lst = s.split(",")  # ["apple", "banana", "cherry"]
```

### 5.3 파일

| 모드 | 설명 |
|------|------|
| `r` | 읽기 |
| `w` | 쓰기 (덮어쓰기) |
| `a` | 추가 (append) |

```python
# 파일 닫기 : file.close()
# with 블록: 자동으로 close 처리
with open("log.txt", "w") as file:
    file.write("로봇 시작\n")   # 파일 쓰기

with open("log.txt", "r") as file:
    data = file.read()         # 파일 읽기
    print(data)

with open("log.txt", "a") as file:
    file.write("새 로그 추가\n")
```
 
---

## 6. 예외 처리와 디버깅 기초

### 6.1 주요 에러

| 에러 | 발생 상황 |
|------|-----------|
| `ValueError` | 형변환 실패 |
| `NameError` | 미선언 변수 사용 |
| `TypeError` | 자료형 불일치 |
| `IndexError` | 리스트 범위 초과 |
| `KeyError` | 딕셔너리에 key 없음 |
| `FileNotFoundError` | 파일이 존재하지 않음 |

### 6.2 try-except

```python
try:
    위험한 코드
except ValueError:
    print("정수를 입력해야 합니다.")
except ZeroDivisionError:
    print("0으로 나눌 수 없음")
except ValueError as e:
    print(e)          # 예외 객체 출력
else:
    print("에러 없음") # 에러가 없을 때만 실행
finally:              # 에러 여부와 상관없이 항상 실행
    print("항상 실행") # 파일 닫기 등 마무리 작업
```

### 6.3 파일 처리와 예외 처리

```python
try:
    with open("sensor.txt", "r") as file:
        data = file.read()
except FileNotFoundError:
    print("파일이 존재하지 않습니다.")
```

---

## 7. 클래스

```python
class Robot:
    # 초기화 함수: 객체가 생성될 때 자동 실행
    # self: 자기 자신을 가리키는 변수
    def __init__(self, name, battery):
        self.name = name        # 인스턴스 변수
        self.battery = battery

    def move(self):
        print(f"{self.name} 이동 중")
```

### 7.1 객체

```python
r = Robot("turtlebot", 90)   # 객체 생성
r.name                       # 변수 접근
r.move()                     # 메서드 호출
```

> **객체** vs **인스턴스**: `r`은 객체이자 Robot의 인스턴스 ("어떤 클래스로부터 만들어졌는지" 강조)

### 7.2 상속

부모 클래스의 속성과 메서드를 자식 클래스가 물려받는 것
- 기존 클래스를 수정하지 않고 기능을 추가하거나 변경할 때 사용

```python
class 자식(부모):
    def __init__(self, ...):
        super().__init__(...)  # 부모 클래스의 초기화 호출
        self.추가변수 = ...
```

---

## 8. ROS2 연결 맛보기

### 8.1 데이터 통신 방식
- **토픽(Topic)**: 1:N, N:N 통신 / 비동기
    - **publisher**: 메시지 발행
    - **subscriber**: 메시지 수신
- **서비스(Service)**: 1:1 통신 / 동기
    - **client**: 요청
    - **server**: 요청 받아 처리 후 응답
- **액션(Action)**: 토픽 + 서비스 같이 사용하는 장시간 작업 통신

---

## 9. 추가 문법

### 언패킹 연산자

| 연산자 | 설명 |
|---|---|---|
| `*` | 리스트, 튜플 언패킹 |
| `**` | 딕셔너리 언패킹 |

```python
lst = [1, 2, 3]
dic = {"a": 1, "b": 2}

func(*lst)   # func(1, 2, 3)
func(**dic)  # func(a=1, b=2)
```

### `zip()` 함수

```python
for a, b in zip([1, 2, 3], [4, 5, 6]):
    print(a, b)
# 1 4 / 2 5 / 3 6
```

### iterable

for문으로 순회 가능한 객체 → `list`, `tuple`, `str`, `dict`, `set`, `range`

---

## 10. 주요 라이브러리

| 라이브러리 | 설명 |
|-----------|------|
| `faker` | 가짜 데이터 생성 |
| `pandas` | 데이터 읽기·가공·분석 (엑셀처럼) |
| `matplotlib` | 데이터 시각화 (그래프) |