# C++

## 1. 기본

### 1-1. 자료형

| 자료형 | 설명 | 헤더파일 |
| --- | --- | --- |
| int | 정수형 | |
| float, double | 실수형 | |
| char | 문자형 | |
| bool | 불리언형 | |
| std::string | 문자열형 | `<string>` |

> bool 형식은 1이 참, 0이 거짓으로 표현된다.  
> true/false 형식으로 출력하려면 `std::boolalpha`를 사용해야 한다.  
> 이를 해제하려면 `std::noboolalpha`를 사용해야 한다.

### 1-2. 화면에 출력

`std::cout << "출력 문자열" << 변수명 << std::endl;`

| 함수 | 기능 | 비고 |
|---|---|---|
| std::cout | 화면에 출력 | 버퍼에 저장했다 출력할 수 있음 |
| std::endl | 줄바꿈 및 버퍼 비우기(flush) | 즉시 출력 보장 |

> `<iostream>` 헤더파일 필요  
> `std::` 생략 하려면 `using namespace std;` 사용

### 1-3. 키보드 입력 받기

| 함수 | 기능 |
|---|---|
| `std::cin >> 변수명` | 공백 전까지만 읽어옴 |
| `std::getline(std::cin, 변수명)` | 엔터 전까지 읽어옴 |
| `std::cin.ignore()` | 버퍼에 남아있는 엔터를 제거 |

```cpp
int age;
std::string name;

std::cin >> age;    // 버퍼에 엔터가 남아있음
std::cin.ignore();  // 버퍼에 남아있는 엔터를 제거
std::getline(std::cin,name);   // 버퍼에 엔터가 남아있다면 동작이 원활하지 않음
```

### 1-4. const

값을 변경하지 못하게 만드는 키워드

### 1-5. return 값

- 0: 정상 종료
- 1: 오류 종료

---

## 2. 조건문

### 2-1. if문

```cpp
if (조건) {
  // 조건이 참일 때 실행
} else if (조건) {
  // 조건이 참일 때 실행
} else {
  // 조건이 거짓일 때 실행
}
```

### 2-2. switch문

```cpp
switch (변수) {
  case 값1:  // 변수의 값이 값1일 때
    // 실행
    break;  // switch문 탈출
  case 값2:  // 변수의 값이 값2일 때
    // 실행
    break;
  default:  // 위의 모든 case에 해당하지 않을 때
    // 실행
    break;
}
```

---

## 3. 반복문

### 3-1. for문

```cpp
for (초기화; 조건; 증감) {
  // 실행
}
```

### 3-2. while문

```cpp
while (조건) {
  // 실행
}
```

### 3-3. do-while문

```cpp
do {
  // 실행
} while (조건);
```

### 3-4. break, continue

- break: 반복문 탈출
- continue: 현재 반복 건너뛰고 다음 반복 실행

---

## 4. 함수

```cpp
반환타입 함수명(매개변수) {
  // 실행
  return 반환값;
}
```

> 반환값이 없다면 반환타입으로 `void` 사용  
> `main` 함수 보다 아래에 있다면 함수 헤더를 먼저 선언

### 4-1. 콜백 함수

어떤 조건이나 이벤트가 발생했을 때 다른 코드가 대신 호출하는 함수

```cpp
void print_robot_status() {
    std::cout << "Robot status: OK" << std::endl;
}
void print_battery(int battery) {
    std::cout << "Battery: " << battery << "%" << std::endl;
}

/* 방법1: 함수 포인터 방식 */
void run_callback(void (*callback)()) {
    std::cout << "Before callback" << std::endl;
    callback();   // 전달받은 함수 실행
    std::cout << "After callback" << std::endl;
}
/* 방법2: std::function 방식 */
void run_callback(std::function<void()> callback) {
    std::cout << "Before callback" << std::endl;
    callback();
    std::cout << "After callback" << std::endl;
}
/* 매개변수가 있는 함수 */
void run_battery_callback(std::function<void(int)> callback) {
    int battery = 85;
    callback(battery);
}

int main() {
    run_callback(print_robot_status);   // 함수 자체 전달 (함수 주소 전달)
    run_battery_callback(print_battery);

    return 0;
}
```

### 4-2. 람다 함수


---

## 5. 배열

```cpp
#include <vector>

int battery[4] = {1,2,3,4}; // 크기 고정
std::vector<int> batterys = {1,2,3,4}; // 크기 가변

batterys.size(); // 벡터 크기 반환
batterys.push_back(5); // 벡터에 값 추가

for (std::size_t i : batterys) { // 벡터 순회
  std::cout << i << std::endl;
}
```

> `std::size_t`는 부호없는 정수형으로 벡터의 크기를 나타낼 때 사용한다.

---

## 6. 포인터와 참조

### 6-1. 포인터

주소를 저장하는 변수 (메모리 공간 차지)

```cpp
int a = 10;
int* p = &a; // 포인터 변수 p는 변수 a의 메모리 주소를 가리킴
int* ptr = nullptr; // 눌 포인터

std::cout << *p << std::endl; // 역참조(*), a의 값을 출력
```

#### 포인터 객체 접근

```cpp
struct Point {
    int x;
    int y;
};

int main() {
    Point p = {10, 20};
    Point* ptr = &p;

    // 1. 화살표 연산자 사용 (가장 일반적이고 권장되는 방법)
    std::cout << ptr->x << std::endl;  // 출력: 10
    std::cout << ptr->y << std::endl;  // 출력: 20

    // 2. 역참조와 점(.) 연산자 조합
    std::cout << (*ptr).x << std::endl;  // 출력: 10
    std::cout << (*ptr).y << std::endl;  // 출력: 20

    return 0;
}
```

#### 동적 할당

프로그램 실행 중(runtime)에 원하는 크기의 메모리를 만들기 위해서 사용

```cpp
int n;
std::cin >> n;

int* arr = new int[n];  // 메모리 주소 반환
delete[] arr;  // 메모리 해제
```

#### 스마트 포인터

메모리 해제를 자동으로 도와주는 포인터 객체
- `#include <memory>` 사용

1. `std::unique_ptr`: 하나의 객체를 딱 하나의 포인터만 소유할 수 있게 하는 스마트 포인터

```cpp
class Robot {
public:
    void hello() {
        std::cout << "Hello Robot" << std::endl;
    }
};

int main() {
    // 1. unique_ptr 생성 (자동 메모리 관리)
    std::unique_ptr<Robot> p = std::make_unique<Robot>();
    p->hello();

    // 2. 복사 시도 - 복사 불가 (❌ 컴파일 에러)
    std::unique_ptr<Robot> q = p;

    // 3. move로 소유권 이동 - p는 nullptr이 됨
    std::unique_ptr<Robot> q = std::move(p);

    if (!p) {
        std::cout << "p is empty (ownership moved)" << std::endl;
    }

    q->hello();

    // main 끝나면 q가 자동 delete
}
```

2. `std::shared_ptr`: 여러 포인터가 하나의 객체를 “공동 소유”할 수 있는 스마트 포인터

```cpp
class Robot {
public:
    Robot() {
        std::cout << "Robot created" << std::endl;
    }

    ~Robot() {
        std::cout << "Robot destroyed" << std::endl;
    }

    void hello() {
        std::cout << "Hello Robot" << std::endl;
    }
};

int main() {

    // 1. shared_ptr 생성
    std::shared_ptr<Robot> p1 = std::make_shared<Robot>();

    /* use_count()
     *: 스마트 포인터가 가리키는 객체를 소유한 스마트 포인터의 개수를 저장하는
     *  참조 카운터를 반환
     */
    std::cout << "p1 count: " << p1.use_count() << std::endl; // 1

    // 새로운 지역 범위
    {
        // 2. 복사 → 공동 소유
        std::shared_ptr<Robot> p2 = p1;

        p2->hello();

        std::cout << "p1 count: " << p1.use_count() << std::endl; // 2
        std::cout << "p2 count: " << p2.use_count() << std::endl; // 2

    } // p2 소멸

    std::cout << "After p2 destroyed" << std::endl;
    std::cout << "p1 count: " << p1.use_count() << std::endl; // 1

} // 마지막 p1 소멸 → 자동 delete
```

> 함수의 매개변수 - 값 전달: 복사되므로 참조 카운트 증가  
> 함수의 매개변수 - 참조 전달: 복사x, 참조 카운트 유지

### 6-2. 참조

기존 변수의 별명

```cpp
int a = 10;
int& r = a; // 참조 변수 r은 변수 a를 가리킴

std::cout << r << std::endl; // a의 값을 출력
```

```cpp
void print(int& a) {
  a = 10;
  std::cout << a << std::endl;
}

int main() {
  int b = 0;
  print(b); // 10
  std::cout << b << std::endl;  // 10
  return 0;
}
```

> `a`는 `b`와 같은 메모리를 가리키는 별명  
> 함수 호출 시 변수 복사가 일어나지 않음  
> `const`를 매개변수 앞에 붙여주면 함수 안에서 변수 값을 변경하지 못하게 할 수 있음

---

## 7. 구조체와 클래스

### 7-1. 구조체

```cpp
struct Point {   // 새로운 자료형 생성
    int x;
    int y;
};

int main() {
    Point p;
    p.x = 10;
    p.y = 20;
    std::cout << p.x << std::endl;
    std::cout << p.y << std::endl;
    return 0;
}
```

> 구조체는 기본값이 `public`

### 7-2. 클래스

- 멤버 변수: 클래스 내부 변수
- 멤버 함수: 클래스 내부 함수

```cpp
class 클래스이름 {
public:
    // 클래스 외부에서 접근 가능

private:
    // 클래스 외부에서 직접 접근 불가
};
```

> 클래스는 기본값이 `private`  
> **캡슐화**: 클래스 내부 데이터를 외부에서 마음대로 바꾸지 못하게 보호하고, 정해진 함수로만 접근하게 하는 설계 방식  

```cpp
class Robot {
public:
    /* 생성자: 객체가 만들어질 때 자동으로 호출되는 특별한 함수
     * 클래스 이름과 같음, 반환형 없음
     */
    Robot() { 
        name_ = "unknown";
        battery_ = 100;
        speed_ = 0.0;
    }
    
    /* 생성자 오버로딩(overloading): 매개변수가 다르면 다른 함수로 인식
     */
    Robot(const std::string& name, int battery) {
        name_ = name;
        battery_ = battery;
        speed_ = 0.0;
    }

    /* 생성자 초기화 리스트: 객체 생성과 동시에 초기화
     */
    Robot(const std::string& name, int battery, double speed)
    : name_(name), battery_(battery), speed_(speed)
    {
    }

    /* 소멸자: 객체가 소멸될 때 자동으로 호출되는 함수
       - 객체 소멸: 스코프 종료 / delete
     */
    ~Robot() {
        std::cout << name_ << " 소멸\n";
    }

    /* const 멤버 함수: 읽기 전용 함수
     */
    int get_battery() const {
        return battery_;
    }

private:
    std::string name_;
    int battery_;
    double speed_;
};
```

> 객체는 스택(LIFO)에 생성되므로 객체 생성 역순으로 소멸됨

#### 상속

기존 클래스(부모 클래스)의 기능을 **새로운 클래스(자식 클래스)**가 물려받는 문법

```cpp
class Robot {
public:
    Robot(const std::string& name)
        : name_(name),
          battery_(100)
    {
    }

    void print_basic_status() const {
        std::cout << "Robot name: " << name_ << std::endl;
        std::cout << "Battery: " << battery_ << "%" << std::endl;
    }

    void stop() const {
        std::cout << name_ << " stopped." << std::endl;
    }

protected:
    std::string name_;
    int battery_;
};

class MobileRobot : public Robot {
public:
    MobileRobot(const std::string& name)
        : Robot(name)   // 부모 생성자 호출
    {
    }

    void move_forward() const {
        std::cout << name_ << " moving forward." << std::endl;
    }
};

int main() {
    MobileRobot robot("turtlebot3");

    robot.print_basic_status();
    robot.move_forward();
    robot.stop();

    return 0;
}
```

> 자식 객체 생성 시: 부모 객체 생성 → 자식 객체 생성  
> 부모 클래스에 기본 생성자가 없으면 직접 호출 필요

| 접근 제어자 | 클래스 내부 | 자식 클래스 | 외부 | 
|---|---|---|---|
| `public` | O | O | O |
| `protected` | O | O | X |
| `private` | O | X | X |

| 상속 | 설명 |
|---|---|
| public 상속 | 부모의 public 기능을 자식에서도 public으로 유지하며 상속 |
| protected 상속 | 부모의 public 기능을 자식에서 private으로 상속 |
| private 상속 | 부모의 public 기능을 자식에서 protected으로 상속 |

#### 오버라이딩(override)

부모 클래스의 함수를 자식 클래스에서 다시 정의하는 것

```cpp
class Robot {
public:
    virtual void move() {
        std::cout << "Robot moves." << std::endl;
    }
};

class MobileRobot : public Robot {
public:
    void move() override {
        std::cout << "Mobile robot drives." << std::endl;
    }
};
```

| 키워드 | 의미 |
|---|---|
| `virtual` | 이 함수는 자식 클래스에서 재정의될 수 있음 |
| `override` | 부모 클래스의 virtual 함수를 정확히 재정의 |

#### 다형성

같은 함수 호출이라도 객체 종류에 따라 다르게 동작하는 것
- 장점: 코드 통합, 확장과 유지보수 쉬움

```cpp
class Robot {
public:
    virtual void move() {
        std::cout << "Robot moves." << std::endl;
    }

    /* 다형성 사용 시 부모 클래스의 소멸자 앞에 virtual을 붙여야 함
        - virtual이 없으면: 부모 소멸자만 호출 → 메모리 누수 발생 가능
    */
    virtual ~Robot() {
        std::cout << "Robot destroyed" << std::endl;
    }
};

class MobileRobot : public Robot {
public:
    void move() override {
        std::cout << "Mobile robot drives." << std::endl;
    }

    ~MobileRobot() {
        std::cout << "MobileRobot destroyed" << std::endl;
    }
};

class RobotArm : public Robot {
public:
    void move() override {
        std::cout << "Robot arm rotates." << std::endl;
    }

    ~RobotArm() {
        std::cout << "RobotArm destroyed" << std::endl;
    }
};

int main() {
    std::vector<Robot*> robots;

    robots.push_back(new MobileRobot());
    robots.push_back(new RobotArm());

    for (Robot* robot : robots) {
        robot->move();
    }

    for (Robot* robot : robots) {
        delete robot;   // MobileRobot destroyed → Robot destroyed, RobotArm destroyed → Robot destroyed
    }

    return 0;
}
```

---

## 8. 헤더파일(.hpp)

헤더파일은 보통 선언을 담는다

```cpp
/* include guard: 헤더파일이 중복include되는 것을 방지 */

/* 방법1 */
#ifndef ROBOT_STATUS_HPP   // ROBOT_STATUS_HPP가 정의되지 않았다면
#define ROBOT_STATUS_HPP   // ROBOT_STATUS_HPP를 정의

/* 방법2 */
#pragma once

/* =============================================== */

#include <string>

class RobotStatus {
public:
    RobotStatus(const std::string& name, int battery, double speed);

    void print() const;

private:
    std::string name_;
    int battery_;
    double speed_;
};

#endif  // #ifndef의 끝을 의미

/* ==================== 소스파일(.cpp) ==================== */

#include "robot_status.hpp"

#include <iostream>

// 헤더 파일에 선언된 클래스의 함수를 .cpp 파일에서 구현(정의)하는 것
RobotStatus::RobotStatus(const std::string& name, int battery, double speed)
    : name_(name),
      battery_(battery),
      speed_(speed)
{
}
void RobotStatus::print() const {
    std::cout << "===== Robot Status =====" << std::endl;
    std::cout << "Name: " << name_ << std::endl;
    std::cout << "Battery: " << battery_ << "%" << std::endl;
    std::cout << "Speed: " << speed_ << " m/s" << std::endl;
}
```

### 8-1. 오류

| 종류 | 설명 | 원인 |
|---|---|---|
| 컴파일 오류 | 컴파일 단계에서 발생하는 오류 | 문법 오류, 헤더 파일 누락 등 |
| 링커 오류 | 링크 단계에서 발생하는 오류 | 함수나 변수가 선언되었지만 구현되지 않은 경우 |
| 런타임 오류 | 프로그램 실행 중에 발생하는 오류 | 잘못된 메모리 접근, 0으로 나누기 등 |