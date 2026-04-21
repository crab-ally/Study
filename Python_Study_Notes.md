# Python

## 1. Python 코드의 뼈대 익히기

### 1.1 출력문
- **기본형**: print()
    - **구분자 변환**: print(value1, value2, sep="원하는 문자")
    - **끝 문자 변환**: print(value, end="원하는 문자")
- **문자열 포맷팅**: print(f"...{변수명}...")
    - 파이썬 3.6 버전부터 사용 가능
    - **정렬**
        - **왼쪽 정렬**: {변수명:<자릿수}
        - **가운데 정렬**: {변수명:^자릿수}
        - **오른쪽 정렬**: {변수명:>자릿수}

### 1.2 주석
- **한 줄**: # statement
- **여러 줄**: """ statement """

### 1.3 자료형 확인
- **기본형**: type()

---

## 2. 조건문과 반복문

### 2.1 조건문
- **if 문**
```
if 조건:
    statement
elif 조건:
    statement
else:
    statement
```

### 2.2 반복문
- **for 문**
```
for 변수 in 반복대상:
    statement
```
- **while 문**
```
while 조건:
    statement
```

### 2.3 break와 continue
- **break**: 반복문 탈출
- **continue**: 반복문 건너뛰기

### 2.4 range() 함수
- **기본형**: range(start, stop, step)
    - **start**: 시작값 (기본값: 0)
    - **stop**: 끝값 (기본값: 0)
        - 결과값에 stop값은 포함되지 않음
    - **step**: 증가값 (기본값: 1)

### 2.5 논리 연산자
| 연산자 | 설명 |
|:---:|:---:|
| x and y | x와 y 모두 참이어야 참이다. |
| x or y | x와 y 둘 중 하나만 참이어도 참이다. |
| not x | x가 거짓이면 참이다. |

### 2.6 pass
- **pass**: 아무것도 하지 않음
    - 주로 코드 뼈대를 만들 때 사용
    - **활용**: if문, 함수, 클래스 등
    ```
    if True:
        pass
    ```

---

## 3. 리스트 / 튜플 / 딕셔너리

### 3.1 리스트
- **기본형**: sensors = [10, 20, 30]
- **길이**: len(sensors)
- **접근**: sensors[index]
- **추가**: sensors.append(value)
- **변경**: sensors[index] = value
- **삭제**: sensors.remove(value)

### 3.2 튜플
- **기본형**: position = (10, 20)
    - **특징**: 리스트와 달리 값 변경 불가능
    - **요소가 1개인 튜플**: (value,)
- **길이**: len(position)
- **접근**: position[index]
- **이어붙이기**: position + (30, 40)

### 3.3 딕셔너리
- **기본형**
```
robot = {
    "name": "turtlebot",
    "battery": 90,
    "speed": 1.2
}
```
- **길이**: len(robot)
- **접근** 
    - **기본형**: robot[key]
        - **key가 없을 때**: KeyError 발생
    - **안전 접근**: robot.get(key)
        - **key가 없을 때**: None 반환
            - **get(key, default_value)**: key 없을 시 default_value 반환
    - **key 접근**: robot.keys()
        - **반환값**: dict_keys 객체
    - **value 접근**: robot.values()
        - **반환값**: dict_values 객체
    - **key, value 접근**: robot.items()
        - **반환값**: dict_items 객체
        - **각 원소는 튜플**
- **변경**: robot[key] = value
- **추가**: robot[key] = value
- **삭제**:
    - **지정 삭제**
        - del robot[key]: 반환값 없음
        - robot.pop(key): 반환값(value) 있음
    - **마지막 삭제**
        - robot.popitem(): 반환값(key, value) 있음

### 3.4 set
- **기본형**: set_data = {1, 2, 3, 4, 5}
- **특징**
    - 중복된 값 제거
    - 순서 없음 -> 인덱스 접근 불가능

---

## 4. 함수

### 4.1 함수 정의
- **기본형**
```
def 함수명(매개변수1, 매개변수2, ...):
    statement
    return 반환값
```
- **특징**: 함수 안 변수는 밖에서 접근 불가능

### 4.2 매개변수
- **기본형**: def 함수명(매개변수1, 매개변수2, ...)
- **매개변수 기본값**: def 함수명(매개변수1=값1, 매개변수2=값2, ...)
- **가변인자**: def 함수명(*args, **kwargs)
    - **args**: 튜플 형태로 전달
    - **kwargs**: key=value(딕셔너리) 형태로 전달

### 4.3 반환값
- **기본형**: return 반환값
- **여러 반환값**: return 값1, 값2, ...
    - **결과값은 튜플로 반환**

### 4.4 타입 힌트
- "리턴값을 특정 자료형으로 반환하겠다"는 의미
- **장점**
    - 코드 가독성 향상
    - 협업 시 명확함
```
def main() -> int:
    return 0
```

---

## 5. 문자열 처리와 파일 기초

### 5.1 문자열
- **소문자 변환**: string.lower()
- **대문자 변환**: string.upper()
- **문자열 대체**: string.replace(old, new)
- **공백 제거**: string.strip()
- **문자열 찾기**: string.find(substring)

### 5.2 문자열 전송
- 네트워크/파일 전송은 문자열(byte) 기반이라서 리스트 같은 구조를 그대로 못 보냄
- **문자열 합치기**: separator.join(string_list)
    - 리스트를 문자열로 변환
- **문자열 분리**: string.split(separator)
    - 문자열을 리스트로 변환

### 5.3 파일
- **파일 열기**: open(filename, mode)
    - **mode**
        - **r**: 읽기
        - **w**: 쓰기
        - **a**: 추가
- **파일 쓰기**: file.write(string)
- **파일 닫기**: file.close()
- **파일 읽기**: file.read()

```
with open("log.txt", "w") as file:
    file.write("로봇 시작\n")
> 자동으로 close됨

with open("log.txt", "r") as file:
    data = file.read()
    print(data)

with open("log.txt", "a") as file:
    file.write("새 로그 추가\n")
```
 
---

## 6. 예외 처리와 디버깅 기초

### 6.1 에러
- **ValueError**: 형변환이 안 될 때
- **NameError**: 변수가 없을 때
- **TypeError**: 자료형이 맞지 않을 때
- **IndexError**: 리스트 범위를 벗어났을 때
- **KeyError**: 딕셔너리에 key가 없을 때

### 6.2 try-except
- **기본형**
```
try:
    위험할 수 있는 코드
except:
    에러가 났을 때 실행할 코드
```
- **특정 예외 처리**
```
try:
    number = int("abc")
except ValueError:
    print("정수를 입력해야 합니다.")
```
- **여러 예외 처리**
```
try:
    x = int(input())
    y = 10 / x
except ValueError:
    print("숫자 입력해야 함")
except ZeroDivisionError:
    print("0으로 나눌 수 없음")
```
- **예외 객체 보기**
```
try:
    number = int("abc")
except ValueError as e:
    print(e)
```
- **else**: 에러가 없을 때만 실행
```
try:
    number = int("abc")
except ValueError as e:
    print(e)
else:
    print("예외가 발생하지 않았습니다.")
```
- **finally**: 에러 여부와 상관없이 항상 실행
    - 주로 파일 닫기, 연결 해제 등 마무리 작업에 사용
```
try:
    number = int("abc")
except ValueError as e:
    print(e)
finally:
    print("예외 발생 여부와 상관없이 항상 실행")
```

### 6.3 파일 처리와 예외 처리
```
try:
    with open("sensor.txt", "r") as file:
        data = file.read()
        print(data)
except FileNotFoundError:
    print("파일이 존재하지 않습니다.")
```

---

## 7. 클래스

### 7.1 클래스 정의
- **기본형**
```
class 클래스이름:
    def __init__(self, 매개변수1, 매개변수2, ...):
        self.변수1 = 매개변수1
        self.변수2 = 매개변수2
        ...
    > 초기화 함수: 객체가 생성될 때 자동 실행
    > self: 자기 자신을 가리키는 변수
    
    def 메서드명(self, 매개변수1, 매개변수2, ...):
        statement
        return 반환값
```

### 7.2 객체
- **객체와 인스턴스**: a = Cookie()
    - **객체**: 그냥 만들어진 결과물 / a는 객체
    - **인스턴스**: “어떤 클래스로부터 만들어졌는지”를 강조한 말 / a는 Cookie의 인스턴스
- **생성**: 객체이름 = 클래스이름(인수1, 인수2, ...)
- **접근**: 객체이름.변수명, 객체이름.메서드명()

### 7.3 상속
- 부모 클래스의 속성과 메서드를 자식 클래스가 물려받는 것
- **활용**: 기존 클래스를 변경하지 않고 기능을 추가하거나 기존 기능을 변경하려고 할 때 사용
    - 기존 클래스가 라이브러리 형태로 제공되거나 수정이 허용되지 않는 상황이라면 상속을 사용
- **기본형**
```
class 자식클래스_이름(부모클래스_이름):
    def __init__(self, 매개변수1, 매개변수2, ...):
        super().__init__(매개변수1, 매개변수2, ...)
        self.변수1 = 매개변수1
        self.변수2 = 매개변수2
        ...
    > super().__init__(): 부모 클래스의 초기화 함수를 호출
    
    def 메서드명(self, 매개변수1, 매개변수2, ...):
        statement
        return 반환값
```

---

## 8. ROS2 연결 맛보기

### 8.1 데이터 통신 방식
- **토픽(Topic)**: 1:N, N:N 통신 / 비동기
- **서비스(Service)**: 1:1 통신 / 동기
- **액션(Action)**: 토픽 + 서비스 같이 사용하는 장시간 작업 통신

---

## 9. 추가 공부

### 9.1 언패킹 연산자
- *: 리스트, 튜플 언패킹
- **: 딕셔너리 언패킹
```
list_data = [1, 2, 3]
dict_data = {"a": 1, "b": 2, "c": 3}
    
def func(a, b, c):
    print(a, b, c)
    
func(*list_data)  # func(1,2,3)
func(**dict_data)  # func(a=1, b=2, c=3)
```

### 9.2 가변인자
- *args: 개수 제한 없는 매개변수를 받을 때
    - 자동으로 튜플로 묶임
- **kwargs: key=value 형태로 개수 제한 없는 매개변수를 받을 때
    - 자동으로 딕셔너리로 묶임
```
def func(*args, **kwargs):
    print("args:", args)
    print("kwargs:", kwargs)

func(1, 2, 3, a=1, b=2, c=3)  # args: (1, 2, 3), kwargs: {'a': 1, 'b': 2, 'c': 3}
```

### 9.3 zip 함수
- **zip()**: 여러 개의 리스트를 묶어서 튜플로 반환
```
list1 = [1, 2, 3]                      
list2 = [4, 5, 6]                       
                                        # 1 4
for item1, item2 in zip(list1, list2):  # 2 5
    print(item1, item2)                 # 3 6
```

### 9.4 iterable
- 반복해서 하나씩 꺼낼 수 있는 객체 or for문으로 순회할 수 있는 객체
- **예시**: list, tuple, str, dict, set, range

---

## 파이썬 라이브러리
- **faker**: 가짜 데이터 생성
- **pandas**: 데이터 읽기, 가공, 분석 도구(엑셀처럼 데이터를 다루게 해주는 도구)