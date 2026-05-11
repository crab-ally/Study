# Javascript

## 1. 기초

### 1-1. 기본

| 연산자/명령어 | 내용 |
| --- | --- |
| `//`, `/* */` | 주석 |
| `;` | 문장의 끝을 알림 |
| `+`, `,` | 문자열 더하기 |
| `<br>` | 줄바꿈 |
| `&nbsp` | 공백 |

### 1-2. 화면출력

| | `document.write()` | `console.log()` |
|---|---|---|
| 사용이유 | 화면(웹브라우저)에 출력 | 개발자 도구(F12)에 출력 |
| 줄바꿈 | `<br>` | 자동 줄바꿈 |
| 공백 | `&nbsp` | 여러 개의 공백 유지 |

### 1-3. 변수 선언

```javascript
let a = 10;
let b;
b = "Hello";

const c = 10;   // 값을 다시 대입할 수 없는 변수 선언
```

> 자료형 : String, **Number(정수, 실수)**, Boolean ...  
> `typeof` - 자료형 확인

> [!Warning]  
> 변수를 중괄호(`{}`) 안에서 선언하면, 해당 중괄호가 끝나는 시점에서 변수가 소멸됨

### 1-4 변수명명 규칙

- 대소문자 구분
- 공백 불가
- 숫자로 시작 불가
- 특수문자는 `_`, `$` 만 가능
- 예약어(키워드) 사용불가

### 1-5. 모달창(modal window)

현재 작업을 멈추고 사용자 응답을 강제로 받는 창

| 함수 | 기능 | 반환값 |
| --- | --- | --- |
| `alert(문자열)` | 경고창을 띄움 | undefined |
| `confirm(문자열)` | 확인/취소 선택창을 띄움 | true/false |
| `prompt(문자열)` | 텍스트 입력창을 띄움 | 입력값 (문자열) |


---

## 2. 산술연산자

| 연산자/명령어 | 내용 | 특이사항 |
| --- | --- | ---|
| `+` | 덧셈 | |
| `-` | 뺄셈 | |
| `*` | 곱셈 | |
| `/` | 나누기 | 0으로 나누면 `Infinity` 반환 | | 
| `parseInt(a/b)` | 나눗셈 몫 구하기 | 반환값 타입 - Number |
| `%` | 나머지 | 0으로 나누면 `NaN` 반환 |
| `Math.sqrt(제곱)` | 제곱근 | |

### 2-1. 할당 연산자

| 연산자 | 내용 |
| --- | --- |
| `+=` | 증가 할당 |
| `-=` | 감소 할당 |
| `*=` | 곱셈 할당 |
| `/=` | 나눗셈 할당 |
| `%=` | 나머지 할당 |

### 2-2. 증감 연산자

| 연산자 | 내용 | 특이사항 |
| --- | --- | ---|
| `++a` | 전위 증가 | |
| `a++` | 후위 증가 | 우선순위 마지막(출력 이후 증가 처리) |
| `--a` | 전위 감소 | |
| `a--` | 후위 감소 | 우선순위 마지막(출력 이후 감소 처리) |

### 2-3. 소수점 처리

소수 부분이 0이면 생략

| 명령어 | 내용 | 특이사항 |
| --- | --- | ---|
| `실수.toFixed(자릿수)` | 소수점 자릿수 지정 | 반올림 처리, 남은 자릿수를 0으로 채움, 반환값 타입 - String |

### 2-4. 랜덤

| 연산자 | 내용 |
| --- | --- |
| `Math.random()` | 0.0 이상 1.0 미만의 실수를 반환 |
| `Math.floor()` | 소수점 버림 |

---

## 3. 비교연산자 (반환값: true/false)

| 연산자 | 내용 |
| --- | --- |
| `>` | 크다 |
| `<` | 작다 |
| `>=` | 크거나 같다 |
| `<=` | 작거나 같다 |
| `==` | 같다 |
| `!=` | 같지 않다 |
| `===` | 같다 (타입까지) |
| `!==` | 같지 않다 (타입까지) |

### 3-1. 논리연산자

| 연산자 | 내용 | 특이사항 |
| --- | --- | ---|
| `&&` | 그리고 | 앞의 조건이 false이면 뒤의 조건은 실행되지 않음 |
| `\|\|` | 또는 | 앞의 조건이 true이면 뒤의 조건은 실행되지 않음 |
| `!` | 부정 | |

---

## 4. 조건문

### 4-1. If ~ Else 문

```javascript
if(조건식1) {
    조건식1이 참(true)일 때 실행할 문장;
} else if(조건식2) {
    조건식1이 거짓(false)이고, 조건식2가 참(true)일 때 실행할 문장;  
} else if(조건식3) {
    조건식1,2가 거짓(false)이고, 조건식3이 참(true)일 때 실행할 문장;    
} else {
    모든 조건식이 거짓(false)일 때 실행할 문장;
}
```

### 4-2. Switch ~ Case 문

```javascript
switch(변수) {
    case 값1:
        // 변수의 값이 값1과 같을 때 실행할 문장;
        break;
    case 값2:
        // 변수의 값이 값2와 같을 때 실행할 문장;
        break;
    default:
        // 변수의 값이 어떤 값과도 같지 않을 때 실행할 문장;
        break;
}
```

### 4-3. 삼항연산자

```javascript
조건식 ? 참일 때 실행할 문장 : 거짓일 때 실행할 문장;

/* 중첩 */
let c = r > 10 ? "r는 10보다 크다" : (r < 10 ?  "r은 10보다 작다" : "r은 10이다");
```

---

## 5. 반복문

### 5-1. While 문

```javascript
while (조건식) {
    // 조건식이 참(true)인 동안 실행될 문장
}
```

### 5-2. do ~ while 문

```javascript
do {
    // 일단 한 번은 무조건 실행됨
    // 조건식이 참(true)인 동안 실행될 문장
} while(조건식);
```

> [!Warning]
> do ~ while문 뒤에는 반드시 세미콜론(;)을 붙여야 합니다.

### 5-3. for 문

```javascript
for (초기식; 조건식; 증감식) {
    // 조건식이 참(true)인 동안 반복해서 실행할 문장
}
```

> [!Warning]
> `for (let i = 0; i < 10; i++)` - for문이 종료되면 i 변수는 소멸됨

### 5-4. break, continue

- `break` - 반복문을 즉시 탈출(종료)
- `continue` - 현재 반복문의 나머지 코드를 건너뛰고, 다음 반복을 시작

---

## 6. 함수

### 6-1. 함수 선언

```javascript
function 함수명(매개변수1, 매개변수2, ...) {
    // 함수 본문
    return 반환값;
}
```

> [!Warning]
> 함수는 값을 복사해서 전달받기 때문에 원본 데이터는 변경되지 않으며,  
> 반환된 값(return값)을 새로운 변수에 할당해야 변경된 값을 사용할 수 있습니다.

### 6-2. 함수 표현식

Javascript는 함수를 값처럼 다룸

```javascript
const add = function(a, b) {    // 익명 함수
    return a + b;
};

const hello = function () {     // 익명 함수
    console.log("hi");
};

hello(); // hi 출력
```

> [!Warning]
> 함수 표현식은 선언 전에 호출 불가 (함수 호이스팅이 발생하지 않음)

### 6-3. 화살표 함수

함수 func는 arg1, arg2, ...argN 매개변수를 받아
화살표(=>) 우측의 expression을 평가하고
평가 결과를 반환합니다.

```javascript
/* 한 줄 표현식 */
let func = (arg1, arg2, ...argN) => expression

/* 여러 줄 표현식 */
let func = (arg1, arg2, ...argN) => {
    // 코드 작성
    return 반환값;
}
```

> 매개변수가 하나일 때는 소괄호 생략 가능  
> 매개변수가 없으면 빈 소괄호 사용

### 6-4. 매개변수

매개변수에 값을 전달하지 않으면 `undefined`가 할당됨

```javascript
/* 매개변수에 기본값 설정 */
function greet(name = "Guest") {
    console.log(name);
}
const greet = function(name = "Guest") {
    console.log(name);
};
const greet = (name = "Guest") => {
    console.log(name);
};

greet(); // Guest
```

---

## 7. 객체

```javascript
/* 객체 선언 */
let user = {
  name: "John",  // 프로퍼티(property) - name: "John", age: 30, "likes birds": true
  age: 30,       // 키: "name", "age", "likes birds"   값: "John", 30, true
  "likes birds": true // 프로퍼티 이름에 공백이 있을 경우 따옴표로 묶어줘야 한다.
};

/* 단축 프로퍼티: 기존 변수 값을 가져와 넣는 문법 */
const name = "John";
const age = 30;

const user = {
  name,     // name: name 과 같음 -> name: "John"
  age       // age: age 와 같음 -> age: 30
};
```

> 프로퍼티 이름에 공백이 있을 경우 대괄호 표기법을 사용해야 한다.

### 7-1. 객체 값 접근

```javascript
/* 점 표기법 */
console.log(user.name); // John

/* 대괄호 표기법 */
console.log(user["age"]); // 30

/* 변수를 이용한 접근: 대괄호 표기법만 가능 */
let prop = "name";
console.log(user[prop]); // John
```

### 7-2. 프로퍼티 추가 및 삭제

```javascript
// 프로퍼티 추가
user.isAdmin = true;
user["isAdmin"] = true;

// 프로퍼티 삭제
delete user.age;
delete user["age"];
```

### 7-3. 프로퍼티 존재 여부 확인

존재하지 않는 프로퍼티에 접근하려 해도 에러가 발생하지 않고 undefined를 반환

```javascript
"key" in object     // true/false 반환

/* 프로퍼티 전체를 반복할 때 */
for (let key in object) {
    console.log(key);    // 프로퍼티 이름 출력
    console.log(object[key]); // 프로퍼티 값 출력
}
```

### 7-4. 객체 참조

객체가 할당된 변수를 복사하면 동일한 객체에 대한 참조 값이 하나 더 만들어진다

```javascript
let user = { name: 'John' };

let admin = user;

admin.name = 'Pete'; // 'admin' 참조 값에 의해 변경됨

alert(user.name); // 'Pete'가 출력됨. 'user' 참조 값을 이용해 변경사항을 확인함
```

### 7-5. 메서드와 this

```javascript
/* 메서드: 객체 프로퍼티에 할당된 함수 */
user = {
    name: "John",
    age: 30,
    sayHi: function() {
        alert(this.name);   // this: 현재 메서드를 호출한 객체
    }
};

/* 메서드 단축 문법 */
user = {
    name: "John",
    age: 30,

    sayHi() {
        alert(this.name);
    }
};

/* 메서드 할당 */
user.sayHi = function() {
  alert(this.name);
};

user.sayHi(); // John
```

> [!Warning]
> 화살표 함수에서 this는 바깥쪽 스코프의 this 값을 그대로 가져옴 (고유한 의미 없음)

---

