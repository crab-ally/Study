# Python 정리

## 1. Python 코드 기초

### 1.1 출력문

| 형식 | 코드 |
|------|------|
| 기본형 | `print(value)` |
| 구분자 변경 | `print(v1, v2, sep="문자")` |
| 끝 문자 변경 | `print(v, end="문자")` |
| 문자열 포맷팅 (Python 3.6+) | `print(f"...{변수명}...")` |

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

```python
type(변수)
```

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
A if 조건 else B
# 조건이 참이면 A, 거짓이면 B
```

### 2.2 논리 연산자

| 연산자 | 설명 |
|---|---|
| `x and y` | x와 y 모두 참이어야 참 |
| `x or y` | x와 y 중 하나만 참이어도 참 |
| `not x` | x가 거짓이면 참 |

### 2.3 반복문

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

### 2.4 range() 함수

```python
range(start, stop, step)
# start 기본값: 0 / step 기본값: 1
# stop 값은 결과에 포함되지 않음
```

### 2.5 break / continue / pass

| 키워드 | 설명 |
|--------|------|
| `break` | 반복문 완전 탈출 |
| `continue` | 현재 반복 건너뛰기 |
| `pass` | 아무것도 하지 않음 (뼈대 작성 시 사용) |

---

## 3. 자료구조

### 3.1 리스트 (list)

```python
lst = [10, 20, 30]

len(lst)            # 길이
lst[index]          # 접근
lst.append(value)   # 추가
lst[index] = value  # 변경
lst.remove(value)   # 삭제
```

### 3.2 튜플 (tuple)

```python
pos = (10, 20)      # 불변 (값 변경 불가)
single = (9,)       # 요소 1개일 때 쉼표 필수

len(pos)
pos[index]
pos + (30, 40)      # (10, 20, 30, 40)
```

### 3.3 딕셔너리 (dict)

```python
robot = {"name": "turtlebot", "battery": 90}

# 길이
len(robot)

# 접근
robot["name"]                 # KeyError 발생 가능
robot.get("name")             # 없으면 None
robot.get("name", "기본값")    # 없으면 기본값 반환

# 키·값 순회
robot.keys()
robot.values()
robot.items()   # (key, value) 튜플로 반환

# 추가 / 변경
robot["speed"] = 1.2

# 삭제
del robot["speed"]
robot.pop("speed")  # 삭제된 value 반환
robot.popitem()     # 마지막에 추가된 key:value 삭제 후 튜플로 반환
```

### 3.4 집합 (set)

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
def f(a=1, b=2):    # 기본값
    pass
```

```python
def f(*args):       # 가변인자 → 튜플로 받음
    pass
```

```python
def f(**kwargs):    # 키워드 가변인자 → 딕셔너리로 받음
    pass
```

### 4.2 반환값

```python
return value
```

```python
return v1, v2
# 튜플로 반환
```

### 4.3 타입 힌트

#### 반환 타입

```python
def main() -> int:
    return 0
```

#### 매개변수 타입

```python
def print_info(name: str, age: int) -> None:
    print(f"Name: {name}, Age: {age}")
```

---

## 5. 문자열 처리

### 5.1 문자열 메서드

```python
s.lower()           # 소문자로 변환
s.upper()           # 대문자로 변환
s.replace(old, new) # 문자열 치환
s.strip()           # 앞뒤 공백 제거
s.find(sub)         # 부분 문자열 위치 탐색
```

### 5.2 join / split

네트워크·파일 전송은 문자열(byte) 기반이라서 리스트 같은 구조를 그대로 보낼 수 없음

```python
# 리스트 → 문자열 (separator 기준 연결)
lst = ["apple", "banana", "cherry"]

s = ",".join(lst)   # "apple,banana,cherry"
```

```python
# 문자열 → 리스트 (separator 기준 분리)
s = "apple,banana,cherry"

lst = s.split(",")  # ["apple", "banana", "cherry"]
```

---

## 6. 파일 입출력

### 파일 열기 모드

| 모드 | 설명 |
|------|------|
| `r` | 읽기 |
| `w` | 쓰기 (덮어쓰기) |
| `a` | 추가 (append) |

### 파일 쓰기

```python
with open("log.txt", "w") as file:
    file.write("로봇 시작\n")
```

### 파일 읽기

```python
with open("log.txt", "r") as file:
    data = file.read()
```

### 파일 추가

```python
with open("log.txt", "a") as file:
    file.write("새 로그 추가\n")
```

> `with` 블록을 사용하면 자동으로 `close()` 처리

---

## 7. 예외 처리

### 7.1 주요 에러

| 에러 | 발생 상황 |
|------|-----------|
| `ValueError` | 형변환 실패 |
| `NameError` | 미선언 변수 사용 |
| `TypeError` | 자료형 불일치 |
| `IndexError` | 리스트 범위 초과 |
| `KeyError` | 딕셔너리에 key 없음 |
| `FileNotFoundError` | 파일이 존재하지 않음 |

### 7.2 try-except

```python
try:
    위험한 코드
except ValueError:
    print("정수를 입력해야 합니다.")
except ZeroDivisionError:
    print("0으로 나눌 수 없음")
except ValueError as e:
    print(e)
else:
    print("에러 없음")
finally:
    print("항상 실행")
```

### 7.3 파일 예외 처리

```python
try:
    with open("sensor.txt", "r") as file:
        data = file.read()
except FileNotFoundError:
    print("파일이 존재하지 않습니다.")
```

---

## 8. 클래스

### 클래스 정의

```python
class Robot:
    # 초기화 함수 (객체 생성 시 자동 실행)
    def __init__(self, name, battery):
        self.name = name
        self.battery = battery

    def move(self):
        print(f"{self.name} 이동 중")
```

### 객체 생성

```python
r = Robot("turtlebot", 90)

r.name
r.move()
```

> `r`은 객체이자 `Robot` 클래스의 인스턴스

### 상속

부모 클래스의 속성과 메서드를 자식 클래스가 물려받는 것

```python
class 자식(부모):
    def __init__(self, ...):
        # 부모 클래스의 초기화 함수 호출
        super().__init__(...)
        self.추가변수 = ...
```

---

## 9. 추가 문법

### 9.1 리스트 컴프리헨션

반복문으로 리스트를 만드는 코드를 한 줄로 작성하는 문법

```python
# [표현식 for 요소 in 순회가능객체 if 조건]

[x**2 for x in range(10) if x % 2 == 0] # [0, 4, 16, 36, 64]
```

### 9.2 언패킹 연산자

| 연산자 | 설명 |
|---|---|
| `*` | 리스트, 튜플 언패킹 |
| `**` | 딕셔너리 언패킹 |

```python
lst = [1, 2, 3]
dic = {"a": 1, "b": 2}

func(*lst)  # func(1, 2, 3)
func(**dic) # func(a=1, b=2)
```

### 9.3 zip()

```python
for a, b in zip([1, 2, 3], [4, 5, 6]):
    print(a, b)  # 1 4, 2 5, 3 6
```

### 9.4 max()

```python
max(objects, key=기준함수)
```

```python
# 가장 긴 문자열 찾기
max(names, key=len)

# 딕셔너리에서 battery 기준 가장 큰 값 찾기
max(robots, key=lambda robot: robot["battery"])

# 객체에서 battery 기준 가장 큰 값 찾기
max(robots, key=lambda robot: robot.battery)
```

### 9.5 iterable

for문으로 순회 가능한 객체

- list
- tuple
- str
- dict
- set
- range

---

**items()**

딕셔너리의 키와 값을 함께 꺼내는 메서드

```python
for key, value in my_dict.items():
    print(key, value)
```