# 파이썬(Python) 강의안

## 0. 들어가며 — 파이썬을 왜, 어떻게 배워야 할까?

파이썬은 "사람이 읽기 쉬운 언어"로 설계되었습니다. 다른 언어들이 세미콜론(`;`), 중괄호(`{}`)로 코드 블록을 구분한다면, 파이썬은 **들여쓰기(indentation)** 만으로 블록을 구분합니다. 그래서 처음엔 어색해도 익숙해지면 코드가 글처럼 읽힙니다.

> 💡 **20년차 실무자의 한마디**: 파이썬 문법 자체는 어렵지 않습니다. 진짜 실력 차이는 "자료구조를 상황에 맞게 고르는 감각"과 "예외 상황을 미리 대비하는 습관"에서 납니다.

---

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

> 💡 f-string(`f"..."`)은 실무에서 `+` 문자열 연결보다 훨씬 많이 쓰입니다. 깔끔하게 쓸 수 있고, 숫자를 문자열로 직접 변환할 필요도 없습니다.

### 1.2 주석

```python
# 한 줄 주석

"""
여러 줄 주석
"""
```

> ⚠️ 좋은 주석은 **"왜"** 를 설명합니다. (`x = x + 1  # 배열 인덱스를 1부터 시작하도록 보정`)

### 1.3 자료형 확인

```python
type(변수)
```

> 디버깅할 때 가장 먼저 치는 명령어 중 하나입니다. "이 변수가 지금 문자열인지 숫자인지" 헷갈릴 때 바로 확인하세요.

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

> 💡 삼항 연산자는 "한 줄짜리 if-else"가 필요할 때만 씁니다. 조건이 복잡해지면 일반 if문으로 풀어쓰는 것이 가독성에 좋습니다.

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

> 💡 **for vs while 선택 기준**: "반복할 대상(리스트, 범위 등)이 이미 정해져 있다" → `for`. "조건이 충족될 때까지 계속 반복해야 한다(언제 끝날지 모름)" → `while`. 예를 들어 센서 값이 임계치 이하로 떨어질 때까지 반복하는 로직은 `while`이 자연스럽습니다.

### 2.4 range() 함수

```python
range(start, stop, step)
# start 기본값: 0 / step 기본값: 1
# stop 값은 결과에 포함되지 않음
```

> ⚠️ `range(5)`는 `0, 1, 2, 3, 4`를 반환합니다(5는 포함 안 됨). "stop 값 미포함"은 정말 많이 헷갈리는 부분이니 꼭 기억하세요.

### 2.5 break / continue / pass

| 키워드 | 설명 |
|--------|------|
| `break` | 반복문 완전 탈출 |
| `continue` | 현재 반복 건너뛰기 |
| `pass` | 아무것도 하지 않음 (뼈대 작성 시 사용) |

> 💡 `pass`는 "나중에 구현할 함수/클래스의 틀만 먼저 잡아둘 때" 자주 씁니다. 함수 본문을 비워두면 문법 에러가 나기 때문에, 자리표시(placeholder)로 `pass`를 넣어둡니다.

---

## 3. 자료구조

자료구조는 "데이터를 어떤 그릇에 담을 것인가"를 결정합니다. 같은 데이터라도 그릇 선택을 잘못하면 코드가 비효율적이거나 버그가 생기기 쉽습니다.

### 3.1 리스트 (list)

```python
lst = [10, 20, 30]

len(lst)            # 길이
lst[index]          # 접근
lst.append(value)   # 추가
lst[index] = value  # 변경
lst.remove(value)   # 삭제
```

> 💡 리스트는 "순서가 있고, 값이 바뀔 수 있는(mutable)" 그릇입니다. 순서대로 쌓이는 로그, 계속 늘어나는 센서 측정값 목록 등에 적합합니다.

### 3.2 튜플 (tuple)

```python
pos = (10, 20)      # 불변 (값 변경 불가)
single = (9,)       # 요소 1개일 때 쉼표 필수

len(pos)
pos[index]
pos + (30, 40)      # (10, 20, 30, 40)
```

> 💡 튜플은 "한번 정해지면 바뀌면 안 되는 값"에 씁니다. 예를 들어 (x, y) 좌표는 중간에 누가 실수로 값을 바꿔버리면 안 되므로 튜플로 표현하는 것이 안전합니다.

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

> ⚠️ `robot["없는키"]`는 `KeyError`로 프로그램이 멈춥니다. 키가 있을지 확실하지 않다면 `robot.get("키", 기본값)`을 쓰는 습관을 들이세요. 실무에서 가장 흔한 런타임 에러 원인 중 하나입니다.

### 3.4 집합 (set)

```python
s = {1, 2, 3, 4}
# 중복 제거, 순서 없음 → 인덱스 접근 불가
```

> 💡 "중복된 값을 빠르게 제거하고 싶다" → `list(set(원본리스트))`가 가장 짧은 방법입니다. 단, 순서가 보장되지 않으니 순서가 중요한 데이터에는 쓰지 마세요.

### 3.5 자료구조 선택 가이드 (요약)

| 상황 | 추천 자료구조 |
|---|---|
| 순서가 있고 값이 바뀌는 목록 | list |
| 순서가 있고 값이 절대 안 바뀌어야 함 | tuple |
| key로 빠르게 값을 찾고 싶음 | dict |
| 중복 제거, 순서 무관 | set |

---

## 4. 함수

```python
def 함수명(매개변수1, 매개변수2):
    statement
    return 반환값
```

> 함수 안 변수는 밖에서 접근 불가능

> 💡 함수는 "반복되는 코드를 한 군데로 모으는 것"이 1차 목적입니다. 같은 로직을 3번 이상 복사-붙여넣기 하고 있다면, 함수로 뽑아낼 신호입니다.

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

> 💡 `*args`, `**kwargs`는 "인자 개수가 몇 개일지 미리 정할 수 없을 때" 씁니다. 예를 들어 로그 함수 `log(*messages)`는 메시지를 1개든 5개든 자유롭게 받을 수 있습니다.

### 4.2 반환값

```python
return value
```

```python
return v1, v2
# 튜플로 반환
```

> 💡 함수에서 값을 여러 개 돌려줘야 할 때, 굳이 리스트나 딕셔너리를 만들 필요 없이 `return v1, v2`처럼 쉼표로 나열하면 됩니다. 받는 쪽에서는 `a, b = 함수()`처럼 바로 풀어서 받을 수 있습니다.

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

> ⚠️ 타입 힌트는 강제 규칙이 아니라 "약속/문서화"입니다. `name: str`이라고 적어도 실제로 숫자를 넘기면 에러 없이 실행은 됩니다. 다만 협업 시 함수 사용법을 한눈에 알 수 있고, IDE가 자동완성/에러 검출을 더 잘 해주기 때문에 실무에서는 거의 필수로 씁니다.

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

> 💡 `strip()`은 파일이나 사용자 입력을 받을 때 거의 항상 같이 씁니다. `input()`으로 받은 값 끝에 의도치 않은 줄바꿈/공백이 섞여 들어오는 경우가 많기 때문입니다.

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

> 💡 CSV 파일, 센서 로그, API 응답 등 "텍스트 한 줄에 여러 값이 구분자로 합쳐진 데이터"를 다룰 때 `split`/`join`은 정말 자주 등장합니다. 이 둘은 서로 정반대 동작이라는 것만 기억하면 헷갈리지 않습니다.

---

## 6. 파일 입출력

### 파일 열기 모드

| 모드 | 설명 |
|------|------|
| `r` | 읽기 |
| `w` | 쓰기 (덮어쓰기) |
| `a` | 추가 (append) |

> ⚠️ `w` 모드로 열면 기존 파일 내용이 통째로 사라집니다. 로그처럼 계속 누적해야 하는 파일은 반드시 `a` 모드를 써야 합니다.

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

> 💡 `with`를 안 쓰고 `file = open(...)`만 쓰면, 작업이 끝난 뒤 직접 `file.close()`를 호출해야 합니다. 까먹으면 파일이 계속 열린 상태로 남아 다른 프로그램이 그 파일에 접근하지 못하는 문제가 생길 수 있습니다. `with`는 코드 블록이 끝나면 에러가 나도 자동으로 닫아주기 때문에 실무에서는 거의 항상 `with`를 씁니다.

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

> 💡 에러 이름은 단순히 "외울 것"이 아니라, 에러 메시지를 읽고 "내 코드 어디가 문제인지" 빠르게 짚어내기 위한 지도입니다. 에러 메시지를 무시하지 말고 항상 끝까지 읽는 습관을 들이세요.

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

> 💡 각 블록의 역할을 정리하면:
> - `try`: 에러가 날 수도 있는 코드
> - `except`: 특정 에러가 났을 때 실행
> - `else`: 에러가 **안** 났을 때만 실행
> - `finally`: 에러가 나든 안 나든 **무조건** 실행 (자원 정리에 자주 사용)

> ⚠️ `except:`처럼 에러 종류를 명시하지 않고 모든 에러를 다 잡아버리는 습관은 피하세요. 어떤 에러가 났는지 모르게 되어 디버깅이 훨씬 어려워집니다.

### 7.3 파일 예외 처리

```python
try:
    with open("sensor.txt", "r") as file:
        data = file.read()
except FileNotFoundError:
    print("파일이 존재하지 않습니다.")
```

> 💡 파일/네트워크처럼 "내 코드 밖의 외부 요인"에 의존하는 작업은 항상 실패할 가능성이 있다고 가정하고 예외 처리를 같이 작성하는 것이 좋은 습관입니다.

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

> 💡 클래스는 "데이터(속성)와 그 데이터를 다루는 동작(메서드)을 한 묶음으로 만드는 설계도"입니다. `Robot`이라는 설계도 하나로, `turtlebot`, `pinky` 같은 실제 로봇 인스턴스를 여러 개 찍어낼 수 있습니다.

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

> 💡 예를 들어 `Robot`이라는 공통 부모 클래스를 만들고, `DeliveryRobot(Robot)`, `CleaningRobot(Robot)`처럼 자식 클래스를 만들면, 공통 기능(`move`, `battery`)은 부모에서 그대로 물려받고 각 로봇만의 고유 기능만 자식 클래스에 추가하면 됩니다. 중복 코드를 줄이는 대표적인 방법입니다.

---

## 9. 추가 문법

### 9.1 리스트 컴프리헨션

반복문으로 리스트를 만드는 코드를 한 줄로 작성하는 문법

```python
# [표현식 for 요소 in 순회가능객체 if 조건]

[x**2 for x in range(10) if x % 2 == 0] # [0, 4, 16, 36, 64]
```

> 💡 위 코드는 풀어서 쓰면 다음과 같습니다.
> ```python
> result = []
> for x in range(10):
>     if x % 2 == 0:
>         result.append(x**2)
> ```
> 같은 결과를 4줄 → 1줄로 줄여줍니다. 단, 조건이 너무 복잡해지면 가독성이 떨어지니 그럴 땐 다시 일반 for문으로 풀어쓰는 것이 낫습니다.

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

> 💡 4.1에서 배운 `*args`, `**kwargs`와 정반대 상황입니다. 함수를 "정의"할 때는 여러 인자를 모아 받는 용도(`*args`)였지만, 함수를 "호출"할 때는 이미 있는 리스트/딕셔너리를 풀어서 넘기는 용도(`*lst`)로 씁니다.

### 9.3 zip()

```python
for a, b in zip([1, 2, 3], [4, 5, 6]):
    print(a, b)  # 1 4, 2 5, 3 6
```

> 💡 길이가 같은 두 리스트를 "짝지어서" 동시에 순회하고 싶을 때 씁니다. 예를 들어 이름 리스트와 점수 리스트를 따로 가지고 있을 때, `zip(names, scores)`로 묶어서 한 번에 순회할 수 있습니다.

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

> 💡 `key`에 들어가는 `lambda`는 "이름 없는 한 줄 함수"입니다. `lambda robot: robot["battery"]`는 `robot`을 받아서 `robot["battery"]`를 반환하는 함수와 똑같습니다. `max`, `min`, `sorted` 등 "기준이 필요한" 함수에서 자주 짝지어 등장합니다.

### 9.5 iterable

for문으로 순회 가능한 객체

- list
- tuple
- str
- dict
- set
- range

> 💡 "for문에 넣을 수 있는 모든 것"을 통틀어 iterable이라고 부릅니다. 위 6가지만 우선 기억해두면 대부분의 코드를 읽고 쓰는 데 충분합니다.

### 9.6 items()

딕셔너리의 키와 값을 함께 꺼내는 메서드

```python
for key, value in my_dict.items():
    print(key, value)
```

---