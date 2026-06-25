# 파이썬(Python)

## 0. 들어가며

파이썬은 "사람이 읽기 쉬운 언어"로 설계되었습니다. C++이나 Java가 세미콜론(`;`)과 중괄호(`{}`)로 코드 블록을 구분한다면, 파이썬은 **들여쓰기(indentation)** 만으로 블록을 구분합니다.

파이썬은 또한 **동적 타이핑(dynamic typing)** 언어입니다. 변수를 선언할 때 자료형을 명시하지 않아도 되고, 실행하면서 자료형이 자동으로 결정됩니다.

> 파이썬 문법 자체는 어렵지 않습니다. 진짜 실력 차이는 다음 세 가지에서 납니다.
> 1. 상황에 맞는 **자료구조를 고르는 감각**
> 2. **예외 상황을 미리 대비**하는 습관
> 3. 같은 동작을 반복하지 않고 **함수·클래스로 추상화**하는 능력

---

## 1. 변수와 기본 자료형

### 1.1 변수

변수는 값을 저장하는 "이름이 붙은 상자"입니다.

```python
robot_name = "turtlebot"
battery = 90
```

> ⚠️ 변수명 규칙: 숫자로 시작 불가, 공백 불가, 예약어(`if`, `for`, `class` 등) 사용 불가. 관례적으로 `snake_case`(소문자+언더스코어)를 씁니다.

### 1.2 기본 자료형

| 자료형 | 설명 | 예시 |
|---|---|---|
| `int` | 정수 | `90`, `-3` |
| `float` | 실수 | `3.14`, `0.5` |
| `str` | 문자열 | `"turtlebot"` |
| `bool` | 참/거짓 | `True`, `False` |
| `NoneType` | 값이 없음을 의미 | `None` |

```python
type(변수)   # 변수의 현재 자료형 확인
```

> 💡 `type()`은 디버깅할 때 가장 먼저 치는 명령어 중 하나입니다. "이 변수가 지금 문자열인지 숫자인지" 헷갈릴 때 바로 확인하세요.

### 1.3 형변환

```python
int("90")     # 90       (문자열 → 정수)
float("3.14") # 3.14     (문자열 → 실수)
str(90)       # "90"     (정수 → 문자열)
bool(0)       # False    (0, "", [], None 은 False로 취급됨)
```

> ⚠️ `int("3.14")`는 에러가 납니다. 소수점이 포함된 문자열은 먼저 `float()`을 거친 뒤 `int()`로 변환해야 합니다. (`int(float("3.14"))`)

---

## 2. 출력과 입력

### 2.1 출력문

| 형식 | 코드 |
|---|---|
| 기본형 | `print(value)` |
| 구분자 변경 | `print(v1, v2, sep="문자")` |
| 끝 문자 변경 | `print(v, end="문자")` |
| 문자열 포맷팅 (Python 3.6+) | `print(f"...{변수명}...")` |

**f-string 정렬 옵션**

```python
f"{변수:<10}"   # 왼쪽 정렬, 10자리
f"{변수:^10}"   # 가운데 정렬, 10자리
f"{변수:>10}"   # 오른쪽 정렬, 10자리
f"{변수:.2f}"   # 소수점 둘째자리까지
```

> 💡 f-string은 실무에서 `+` 문자열 연결보다 훨씬 많이 쓰입니다. 가독성이 좋고, 숫자를 문자열로 직접 변환할 필요도 없습니다.

### 2.2 입력문

```python
name = input("이름을 입력하세요: ")
```

> ⚠️ `input()`은 항상 **문자열(str)**을 반환합니다. 숫자로 쓰려면 반드시 `int()` 또는 `float()`으로 변환해야 합니다.

### 2.3 주석

```python
# 한 줄 주석

"""
여러 줄 주석
"""
```

> ⚠️ 좋은 주석은 코드가 **"무엇을"** 하는지가 아니라 **"왜"** 그렇게 했는지를 설명합니다.
> `x = x + 1  # 배열 인덱스를 1부터 시작하도록 보정` 처럼요.

---

## 3. 연산자

| 종류 | 연산자 | 설명 |
|---|---|---|
| 산술 | `+ - * / // % **` | `//`는 정수 나눗셈(몫), `%`는 나머지, `**`는 거듭제곱 |
| 비교 | `== != > < >= <=` | 결과는 항상 `bool` |
| 논리 | `and / or / not` | 아래 3.1 참고 |
| 할당 | `= += -= *= /=` | `x += 1`은 `x = x + 1`과 동일 |

### 3.1 논리 연산자

| 연산자 | 설명 |
|---|---|
| `x and y` | x와 y 모두 참이어야 참 |
| `x or y` | x와 y 중 하나만 참이어도 참 |
| `not x` | x가 거짓이면 참 |

> 💡 `and`/`or`는 **단축 평가(short-circuit)**를 합니다. `x and y`에서 `x`가 거짓이면 `y`는 평가조차 하지 않고 바로 거짓을 반환합니다. 그래서 `if data and data[0] == "OK":` 처럼 안전 검사와 실제 조건을 한 줄에 같이 쓸 수 있습니다.

---

## 4. 조건문과 반복문

### 4.1 조건문

```python
if 조건:
    statement
elif 조건:
    statement
else:
    statement
```

**삼항 연산자**

```python
A if 조건 else B
# 조건이 참이면 A, 거짓이면 B
```

> 💡 삼항 연산자는 "한 줄짜리 if-else"가 필요할 때만 씁니다. 조건이 복잡해지면 일반 if문으로 풀어쓰는 것이 가독성에 좋습니다.

### 4.2 반복문

**for 문**

```python
for 변수 in 반복대상:
    statement
```

**while 문**

```python
while 조건:
    statement
```

> 💡 **for vs while 선택 기준**
> - 반복할 대상(리스트, 범위 등)이 이미 정해져 있다 → `for`
> - 조건이 충족될 때까지 계속 반복해야 한다(언제 끝날지 모름) → `while`
>
> 예를 들어 센서 값이 임계치 이하로 떨어질 때까지 반복하는 로직은 `while`이 자연스럽습니다.

**range() 함수**

```python
range(start, stop, step)
# start 기본값: 0 / step 기본값: 1
# stop 값은 결과에 포함되지 않음
```

> ⚠️ `range(5)`는 `0, 1, 2, 3, 4`를 반환합니다(5는 포함 안 됨). "stop 값 미포함"은 정말 많이 헷갈리는 부분이니 꼭 기억하세요.

**break / continue / pass**

| 키워드 | 설명 |
|---|---|
| `break` | 반복문 완전 탈출 |
| `continue` | 현재 반복만 건너뛰고 다음 반복으로 |
| `pass` | 아무것도 하지 않음 (뼈대 작성 시 사용) |

> 💡 `pass`는 "나중에 구현할 함수/클래스의 틀만 먼저 잡아둘 때" 자주 씁니다. 함수 본문을 비워두면 문법 에러가 나기 때문에, 자리표시(placeholder)로 `pass`를 넣어둡니다.

---

## 5. 자료구조

자료구조는 "데이터를 어떤 그릇에 담을 것인가"를 결정합니다. 같은 데이터라도 그릇 선택을 잘못하면 코드가 비효율적이거나 버그가 생기기 쉽습니다.

### 5.1 리스트 (list)

```python
lst = [10, 20, 30]

len(lst)               # 길이
lst[index]             # 접근
lst.append(value)      # 추가
lst[index] = value     # 변경
lst.remove(value)      # 값으로 삭제
lst.pop(index)         # 위치로 삭제 (삭제된 값 반환)
lst.sort()             # 오름차순 정렬 (원본 변경)
lst.sort(reverse=True) # 내림차순 정렬 (원본 변경)
lst.reverse()          # 순서 뒤집기 (원본 변경)
```

> 💡 리스트는 "순서가 있고, 값이 바뀔 수 있는(mutable)" 그릇입니다. 순서대로 쌓이는 로그, 계속 늘어나는 센서 측정값 목록 등에 적합합니다.

**인덱싱과 슬라이싱**

```python
lst = [10, 20, 30, 40, 50]

lst[0]      # 10  (첫 번째)
lst[-1]     # 50  (마지막) / 음수 인덱싱: 뒤에서부터 세는 방식
lst[1:3]    # [20, 30]
lst[:2]     # [10, 20]
lst[::2]    # [10, 30, 50]  (2칸씩 건너뛰기)
lst[::-1]   # [50, 40, 30, 20, 10]  (역순)
```

> ⚠️ 슬라이싱은 문자열에도 똑같이 적용됩니다. `s[1:3]`, `s[::-1]`(문자열 뒤집기) 등. "stop 인덱스는 포함되지 않는다"는 `range()`와 동일한 규칙입니다.

### 5.2 튜플 (tuple)

```python
pos = (10, 20)      # 불변 (값 변경 불가)
single = (9,)       # 요소 1개일 때 쉼표 필수

len(pos)
pos[index]
pos + (30, 40)      # (10, 20, 30, 40)
```

> 💡 튜플은 "한번 정해지면 바뀌면 안 되는 값"에 씁니다. 예를 들어 `(x, y)` 좌표는 중간에 누가 실수로 값을 바꿔버리면 안 되므로 튜플로 표현하는 것이 안전합니다.

### 5.3 딕셔너리 (dict)

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

### 5.4 집합 (set)

```python
s = {1, 2, 3, 4}
# 중복 제거, 순서 없음 → 인덱스 접근 불가
```

> 💡 "중복된 값을 빠르게 제거하고 싶다" → `list(set(원본리스트))`가 가장 짧은 방법입니다. 단, 순서가 보장되지 않으니 순서가 중요한 데이터에는 쓰지 마세요.

### 5.5 자료구조 선택 가이드 (요약)

| 상황 | 추천 자료구조 |
|---|---|
| 순서가 있고 값이 바뀌는 목록 | `list` |
| 순서가 있고 값이 절대 안 바뀌어야 함 | `tuple` |
| key로 빠르게 값을 찾고 싶음 | `dict` |
| 중복 제거, 순서 무관 | `set` |

---

## 6. 함수

```python
def 함수명(매개변수1, 매개변수2):
    statement
    return 반환값
```

> 함수 안에서 만든 변수는 함수 밖에서 접근할 수 없습니다 (지역 변수 / local scope).

> 💡 함수는 "반복되는 코드를 한 군데로 모으는 것"이 1차 목적입니다. 같은 로직을 3번 이상 복사-붙여넣기 하고 있다면, 함수로 뽑아낼 신호입니다.

### 6.1 매개변수

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

### 6.2 반환값

```python
return value
```

```python
return v1, v2
# 튜플로 반환
```

> 💡 함수에서 값을 여러 개 돌려줘야 할 때, 굳이 리스트나 딕셔너리를 만들 필요 없이 `return v1, v2`처럼 쉼표로 나열하면 됩니다. 받는 쪽에서는 `a, b = 함수()`처럼 바로 풀어서 받을 수 있습니다.

### 6.3 타입 힌트

```python
def main() -> int:
    return 0

def print_info(name: str, age: int) -> None:
    print(f"Name: {name}, Age: {age}")
```

> ⚠️ 타입 힌트는 강제 규칙이 아니라 "약속/문서화"입니다. `name: str`이라고 적어도 실제로 숫자를 넘기면 에러 없이 실행은 됩니다. 다만 협업 시 함수 사용법을 한눈에 알 수 있고, IDE가 자동완성/에러 검출을 더 잘 해주기 때문에 실무에서는 거의 필수로 씁니다.

### 6.4 람다(lambda) 함수

```python
lambda 매개변수: 반환식
```

```python
square = lambda x: x ** 2
square(4)   # 16
```

> 💡 람다는 "이름 없는 한 줄 함수"입니다. 따로 `def`로 정의하기 아까운 간단한 함수를, `sorted()`나 `max()` 같은 다른 함수의 인자로 바로 넘길 때 주로 씁니다. (13장에서 다시 다룹니다.)

---

## 7. 문자열 처리

### 7.1 문자열 메서드

```python
s.lower()           # 소문자로 변환
s.upper()           # 대문자로 변환
s.replace(old, new) # 문자열 치환
s.strip()           # 앞뒤 공백 제거
s.find(sub)         # 부분 문자열 위치 탐색 (없으면 -1)
s.startswith(sub)   # 특정 문자열로 시작하는지
s.endswith(sub)     # 특정 문자열로 끝나는지
```

> 💡 `strip()`은 파일이나 사용자 입력을 받을 때 거의 항상 같이 씁니다. `input()`으로 받은 값 끝에 의도치 않은 줄바꿈/공백이 섞여 들어오는 경우가 많기 때문입니다.

### 7.2 join / split

네트워크·파일 전송은 문자열(byte) 기반이라서 리스트 같은 구조를 그대로 보낼 수 없습니다.

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

## 8. 파일 입출력

### 8.1 파일 열기 모드

| 모드 | 설명 |
|---|---|
| `r` | 읽기 |
| `w` | 쓰기 (덮어쓰기) |
| `a` | 추가 (append) |

> ⚠️ `w` 모드로 열면 기존 파일 내용이 통째로 사라집니다. 로그처럼 계속 누적해야 하는 파일은 반드시 `a` 모드를 써야 합니다.

### 8.2 파일 쓰기 / 읽기 / 추가

```python
# 쓰기
with open("log.txt", "w") as file:
    file.write("로봇 시작\n")

# 읽기
with open("log.txt", "r") as file:
    data = file.read()

# 한 줄씩 읽기
with open("log.txt", "r") as file:
    for line in file:
        print(line.strip())

# 추가
with open("log.txt", "a") as file:
    file.write("새 로그 추가\n")
```

> `with` 블록을 사용하면 자동으로 `close()` 처리됩니다.

> 💡 `with`를 안 쓰고 `file = open(...)`만 쓰면, 작업이 끝난 뒤 직접 `file.close()`를 호출해야 합니다. 까먹으면 파일이 계속 열린 상태로 남아 다른 프로그램이 그 파일에 접근하지 못하는 문제가 생길 수 있습니다. `with`는 코드 블록이 끝나면 에러가 나도 자동으로 닫아주기 때문에 실무에서는 거의 항상 `with`를 씁니다.

---

## 9. 예외 처리

### 9.1 주요 에러

| 에러 | 발생 상황 |
|---|---|
| `ValueError` | 형변환 실패 |
| `NameError` | 미선언 변수 사용 |
| `TypeError` | 자료형 불일치 |
| `IndexError` | 리스트 범위 초과 |
| `KeyError` | 딕셔너리에 key 없음 |
| `ZeroDivisionError` | 0으로 나눔 |
| `FileNotFoundError` | 파일이 존재하지 않음 |

> 💡 에러 이름은 단순히 "외울 것"이 아니라, 에러 메시지를 읽고 "내 코드 어디가 문제인지" 빠르게 짚어내기 위한 지도입니다. 에러 메시지를 무시하지 말고 항상 끝까지 읽는 습관을 들이세요.

### 9.2 try-except

```python
try:
    위험한 코드
except ValueError:
    print("정수를 입력해야 합니다.")
except ZeroDivisionError:
    print("0으로 나눌 수 없음")
except Exception as e:
    print(e)
else:
    print("에러 없음")
finally:
    print("항상 실행")
```

> 💡 각 블록의 역할
> - `try`: 에러가 날 수도 있는 코드
> - `except`: 특정 에러가 났을 때 실행
> - `else`: 에러가 **안** 났을 때만 실행
> - `finally`: 에러가 나든 안 나든 **무조건** 실행 (자원 정리에 자주 사용)

> ⚠️ `except:`처럼 에러 종류를 명시하지 않고 모든 에러를 다 잡아버리는 습관은 피하세요. 어떤 에러가 났는지 모르게 되어 디버깅이 훨씬 어려워집니다.

### 9.3 파일 예외 처리

```python
try:
    with open("sensor.txt", "r") as file:
        data = file.read()
except FileNotFoundError:
    print("파일이 존재하지 않습니다.")
```

> 💡 파일/네트워크처럼 "내 코드 밖의 외부 요인"에 의존하는 작업은 항상 실패할 가능성이 있다고 가정하고 예외 처리를 같이 작성하는 것이 좋은 습관입니다.

### 9.4 직접 예외 발생시키기 (raise)

```python
def set_battery(value):
    if not (0 <= value <= 100):
        raise ValueError("배터리는 0~100 사이여야 합니다.")
    return value
```

> 💡 `raise`는 "이 상황은 정상이 아니다"라고 코드 스스로 알리는 방법입니다. 함수가 받은 값이 말이 안 될 때, 조용히 잘못된 값을 반환하기보다 즉시 에러를 발생시키는 것이 버그를 더 빨리 발견하게 해줍니다.

---

## 10. 클래스와 객체지향

### 10.1 클래스 정의

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

### 10.2 객체 생성

```python
r = Robot("turtlebot", 90)

r.name
r.move()
```

> `r`은 객체이자 `Robot` 클래스의 인스턴스입니다.

### 10.3 상속

부모 클래스의 속성과 메서드를 자식 클래스가 물려받는 것입니다.

```python
class 자식(부모):
    def __init__(self, ...):
        # 부모 클래스의 초기화 함수 호출
        super().__init__(...)
        self.추가변수 = ...
```

> 💡 예를 들어 `Robot`이라는 공통 부모 클래스를 만들고, `DeliveryRobot(Robot)`, `CleaningRobot(Robot)`처럼 자식 클래스를 만들면, 공통 기능(`move`, `battery`)은 부모에서 그대로 물려받고 각 로봇만의 고유 기능만 자식 클래스에 추가하면 됩니다. 중복 코드를 줄이는 대표적인 방법입니다.

예시:

```python
class DeliveryRobot(Robot):
    def __init__(self, name, battery, max_load):
        super().__init__(name, battery)
        self.max_load = max_load

    def deliver(self):
        print(f"{self.name}이 최대 {self.max_load}kg까지 배달합니다.")
```

### 10.4 캡슐화 살짝 맛보기

```python
class Robot:
    def __init__(self, name, battery):
        self.name = name
        self._battery = battery   # 관례상 "내부용" 표시 (밑줄 1개)

    def get_battery(self):
        return self._battery
```

> ⚠️ 파이썬은 진짜 "비공개(private)" 변수를 강제하지 않습니다. 변수명 앞에 밑줄 하나(`_battery`)를 붙이는 것은 "이건 외부에서 직접 건드리지 말아주세요"라는 **약속(convention)**일 뿐, 실제로 접근을 막지는 않습니다.

---

## 11. 파이썬다운 코드 작성법 (Pythonic Code)

이 장은 "문법적으로는 알지만 실무 코드에서는 잘 안 쓰게 되는" 패턴들을 다룹니다. 익숙해지면 코드 줄 수가 확 줄어듭니다.

### 11.1 리스트 컴프리헨션

반복문으로 리스트를 만드는 코드를 한 줄로 작성하는 문법입니다.

```python
# [표현식 for 요소 in 순회가능객체 if 조건]
[x**2 for x in range(10) if x % 2 == 0]   # [0, 4, 16, 36, 64]
```

> 💡 위 코드를 풀어서 쓰면 다음과 같습니다.
> ```python
> result = []
> for x in range(10):
>     if x % 2 == 0:
>         result.append(x**2)
> ```
> 같은 결과를 4줄 → 1줄로 줄여줍니다. 단, 조건이 너무 복잡해지면 가독성이 떨어지니 그럴 땐 다시 일반 for문으로 풀어쓰는 것이 낫습니다.

딕셔너리도 같은 방식으로 만들 수 있습니다.

```python
{k: v for k, v in zip(["a", "b"], [1, 2])}   # {'a': 1, 'b': 2}
```

### 11.2 언패킹 연산자

| 연산자 | 설명 |
|---|---|
| `*` | 리스트, 튜플 언패킹 |
| `**` | 딕셔너리 언패킹 |

```python
lst = [1, 2, 3]
dic = {"a": 1, "b": 2}

func(*lst)   # func(1, 2, 3)
func(**dic)  # func(a=1, b=2)
```

> 💡 6.1에서 배운 `*args`, `**kwargs`와 정반대 상황입니다. 함수를 "정의"할 때는 여러 인자를 모아 받는 용도(`*args`)였지만, 함수를 "호출"할 때는 이미 있는 리스트/딕셔너리를 풀어서 넘기는 용도(`*lst`)로 씁니다.

### 11.3 zip()

```python
for a, b in zip([1, 2, 3], [4, 5, 6]):
    print(a, b)  # 1 4 / 2 5 / 3 6
```

> 💡 길이가 같은 두 리스트를 "짝지어서" 동시에 순회하고 싶을 때 씁니다. 예를 들어 이름 리스트와 점수 리스트를 따로 가지고 있을 때, `zip(names, scores)`로 묶어서 한 번에 순회할 수 있습니다.

### 11.4 max() / min() / sorted()와 람다

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

# 정렬도 같은 방식 (원본은 그대로, 새 리스트 반환)
sorted(robots, key=lambda robot: robot["battery"], reverse=True)
```

> 💡 `key`에 들어가는 `lambda`는 "이름 없는 한 줄 함수"입니다. `max`, `min`, `sorted` 등 "기준이 필요한" 함수에서 자주 짝지어 등장합니다.

### 11.5 iterable과 items()

for문으로 순회 가능한 객체를 **iterable**이라고 부릅니다.

- `list`, `tuple`, `str`, `dict`, `set`, `range`

```python
for key, value in my_dict.items():
    print(key, value)
```

> 💡 "for문에 넣을 수 있는 모든 것"을 통틀어 iterable이라고 부릅니다. 위 6가지만 우선 기억해두면 대부분의 코드를 읽고 쓰는 데 충분합니다.

---

## 12. 모듈과 패키지

지금까지는 한 파일 안에서만 코드를 작성했지만, 실무 코드는 항상 여러 파일과 외부 라이브러리로 쪼개져 있습니다.

### 12.1 import 기본

```python
import math
math.sqrt(16)        # 4.0

from math import sqrt
sqrt(16)              # 4.0

import math as m
m.sqrt(16)            # 4.0
```

> 💡 `from ... import *`(전체 가져오기)는 편해 보이지만, 어떤 함수가 어디서 왔는지 추적하기 어려워져서 실무에서는 거의 사용하지 않습니다.

### 12.2 자주 쓰는 표준 라이브러리

| 모듈 | 용도 |
|---|---|
| `math` | 수학 함수 (sqrt, pi, floor 등) |
| `random` | 난수 생성 |
| `datetime` | 날짜·시간 처리 |
| `os` | 파일/경로/환경변수 다루기 |
| `time` | 시간 측정, `sleep()` |

```python
import random
random.randint(1, 10)   # 1~10 사이 정수 (10 포함)

import time
time.sleep(1)            # 1초 대기
```

### 12.3 내가 만든 파일을 모듈로 가져오기

```python
# robot_utils.py
def greet(name):
    return f"{name}, 안녕!"
```

```python
# main.py
from robot_utils import greet
print(greet("turtlebot"))
```

> 💡 코드가 길어지면 기능별로 파일을 나누고 `import`로 가져다 쓰는 것이 한 파일에 모든 걸 다 적는 것보다 관리하기 훨씬 쉽습니다. 클래스(8장)와 함수(6장)가 "코드 내부의 정리 단위"라면, 모듈은 "파일 단위의 정리 단위"라고 생각하면 됩니다.

---