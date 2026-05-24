# Rust 완벽 가이드

## 1. 기본 개념

### 변수와 상수

```rust
fn main() {
    // 불변 변수 (기본값)
    let x = 10;
    let y: i32 = 20;  // 타입 명시
    
    // 가변 변수
    let mut count = 0;
    count = 1;
    
    // 상수 (컴파일 타임에 결정됨)
    const PI: f64 = 3.14;
    const MAX_SPEED: u32 = 100;
}
```

### Shadowing (변수 재선언)

같은 이름으로 변수를 재선언하면 이전 값이 덮어씌워짐. **타입 변경도 가능**.

```rust
let x = 5;
let x = x + 1;      // x = 6
let x = "hello";    // 다른 타입으로 변경 가능
```

---

## 2. 데이터 타입

| 분류 | 타입 | 예 |
|------|------|-----|
| 정수형 (부호 있음) | i8, i16, i32, i64, isize | -5 |
| 정수형 (부호 없음) | u8, u16, u32, u64, usize | 5 |
| 실수형 | f32, f64 | 3.14 |
| 불린형 | bool | true, false |
| 문자형 | char | 'A' (4바이트) |
| 문자열 (참조) | &str | "hello" (읽기 전용) |
| 문자열 (소유) | String | String::from("hello") (읽기+쓰기) |

### 타입 변환

```rust
// as — 숫자 타입 변환
let a: i32 = 10;
let b = a as f64;

// parse() — 문자열 → 숫자
let n: i32 = "42".parse().expect("숫자 아님");

// to_string() — 숫자 → 문자열
let s = 42.to_string();
```

---

## 3. 소유권 (Ownership) & 참조 (Borrowing)

Rust의 가장 핵심 개념. **메모리 안전성**을 컴파일 타임에 보장한다.

### 소유권 규칙

1. 각 값은 **단 하나의 소유자**만 가짐
2. 소유자가 스코프를 벗어나면 메모리 자동 해제
3. 소유권은 **이동(move)** 또는 **복사(copy)**로 전달됨

```rust
// String은 heap 데이터 → move 발생
let s1 = String::from("hello");
let s2 = s1;           // s1의 소유권이 s2로 이동
// println!("{}", s1);  // ❌ 컴파일 에러

// 정수형, 실수형, 불린형은 stack 데이터 → copy 발생
let n1 = 10;
let n2 = n1;           // n1 복사됨
println!("{}", n1);    // ✅ 정상
```

> `&str` 타입은 참조라서 소유권 이동(move)이 발생하지 않는다.

### 참조 (Borrowing)

소유권을 넘기지 않고 **빌려서** 사용. `&` 기호 사용.

```rust
fn greet(name: &String) {  // 소유권 없이 참조만 받음
    println!("Hello, {}!", name);
}

let s = String::from("Alice");
greet(&s);             // 참조 전달
println!("{}", s);     // ✅ s는 여전히 사용 가능
```

### 가변 참조 (Mutable Reference)

```rust
fn change(s: &mut String) {
    s.push_str(" world");
}

let mut s = String::from("hello");
change(&mut s);
println!("{}", s);     // hello world
```

**중요**: 한 번에 하나의 가변 참조만 가능. 가변과 불변 참조는 동시에 불가.

```rust
let mut s = String::from("hello");
let r1 = &s;
let r2 = &s;           // ✅ 여러 개 불변 참조는 가능
let r3 = &mut s;       // ❌ 에러 (r1, r2를 아직 사용 중)
```

### Dangling Reference (댕글링 참조)

```rust
fn dangle() -> &String {
    let s = String::from("hello");
    &s  // ❌ s가 함수 종료 시 사라짐
}

// 올바른 방식 — 소유권 반환
fn no_dangle() -> String {
    String::from("hello")  // ✅
}
```

---

## 4. 함수

main 함수보다 아래에 함수 정의를 하여도 에러가 발생하지 않는다.

```rust
fn main() {
    add(5, 3);
    let result = multiply(4, 5);
}

// 매개변수 타입 명시 필수
fn add(a: i32, b: i32) {
    println!("{}", a + b);
}

// 반환값 — 마지막 표현식에 세미콜론 없음
fn multiply(a: i32, b: i32) -> i32 {
    a * b  // 세미콜론 X
}

// 명시적 return
fn divide(a: i32, b: i32) -> i32 {
    if b == 0 {
        return 0;
    }
    a / b
}
```

---

## 5. 조건문 & 반복문

### 조건문

```rust
let age = 20;

if age >= 18 {
    println!("성인");
} else if age >= 13 {
    println!("청소년");
} else {
    println!("어린이");
}

// 삼항 연산자처럼 사용
let msg = if age >= 18 { "성인" } else { "미성년" };
```

### 반복문

```rust
// for — 범위 반복
for i in 0..5 {
    println!("{}", i);  // 0~4
}

for i in 0..=5 {
    println!("{}", i);  // 0~5 (포함)
}

// 컬렉션 순회
let nums = vec![1, 2, 3];
for n in nums.iter() {  // 요소를 참조로 빌림 
// for n in nums 는 소유권을 가져옴
    println!("{}", n);
}

// while — 조건 반복
let mut count = 0;
while count < 5 {
    println!("{}", count);
    count += 1;
}

// loop — 무한 반복
let mut x = 0;
loop {
    x += 1;
    if x == 5 {
        break;
    }
}
```

---

## 6. 구조체 (Struct)

```rust
#[derive(Debug)]
struct User {
    name: String,
    age: u32,
    email: String,
}

impl User {
    // 생성자
    fn new(name: String, age: u32, email: String) -> Self {
        User { name, age, email }
    }
    
    // 읽기 메서드 (&self)
    fn display(&self) {
        println!("이름: {}", self.name);
    }
    
    // 수정 메서드 (&mut self)
    fn update_age(&mut self, new_age: u32) {
        self.age = new_age;
    }
    
    // 소유권 가져오는 메서드 (self)
    fn consume(self) -> String {
        self.name  // self가 움직임
    }
}

fn main() {
    let mut user = User::new(
        String::from("Alice"),
        25,
        String::from("alice@example.com")
    );
    
    user.display();
    user.update_age(26);
    let name = user.consume();  // user는 더 이상 사용 불가
}
```

---

## 7. Enum & Match

```rust
#[derive(Debug)]
enum Direction {
    Up,
    Down,
    Left,
    Right,
}

enum Message {
    Text(String),
    Number(i32),
    User { name: String, age: u32 },
}

fn main() {
    let dir = Direction::Up;
    
    match dir {
        Direction::Up => println!("위로"),
        Direction::Down => println!("아래로"),
        _ => println!("다른 방향"),  // 나머지 모든 경우
    }
    
    let msg = Message::Text(String::from("hello"));
    match msg {
        Message::Text(s) => println!("텍스트: {}", s),
        Message::Number(n) => println!("숫자: {}", n),
        Message::User { name, age } => println!("{}, {}", name, age),
    }
}
```

---

## 8. Option & Result

### Option — 값의 존재/부재를 표현

```rust
enum Option<T> {
    Some(T),  // 값 있음
    None,     // 값 없음
}

fn find_max(nums: &[i32]) -> Option<i32> {
    if nums.is_empty() {
        None
    } else {
        Some(*nums.iter().max().unwrap())   // *(&i32)
    }
}

fn main() {
    let nums = vec![1, 5, 3, 9, 2];
    
    match find_max(&nums) {
        Some(max) => println!("최댓값: {}", max),
        None => println!("배열이 비어있음"),
    }
    
    // if let — match보다 간단할 때
    if let Some(max) = find_max(&nums) {
        println!("최댓값: {}", max);
    }
}
```

### Result — 성공/실패를 표현

```rust
enum Result<T, E> {
    Ok(T),   // 성공
    Err(E),  // 실패
}

fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err(String::from("0으로 나눌 수 없음"))
    } else {
        Ok(a / b)
    }
}

fn main() {
    match divide(10, 2) {
        Ok(result) => println!("결과: {}", result),
        Err(e) => println!("오류: {}", e),
    }
    
    // unwrap() — Ok면 값 반환, Err면 panic
    let result = divide(10, 2).unwrap();
    
    // expect() — unwrap() + 사용자 정의 에러 메시지
    let result = divide(10, 0).expect("0으로 나눠서는 안 됨");
}
```

### ? 연산자 (에러 전파)

성공하면 값을 반환하고 실패하면 즉시 에러값 반환하고 함수 종료.

```rust
fn read_config() -> Result<String, String> {
    let content = std::fs::read_to_string("config.txt")
        .map_err(|e| format!("파일 읽기 실패: {}", e))?;
    
    if content.is_empty() {
        return Err(String::from("파일이 비어있음"));
    }
    
    Ok(content)
}
```

---

## 9. 컬렉션

### Vector (동적 배열)

```rust
let mut v = Vec::new();
v.push(1);
v.push(2);
v.push(3);

let v = vec![1, 2, 3, 4, 5];

// 접근
println!("{}", v[0]);  // 인덱스 범위 벗어나면 panic

// 안전한 접근 - .get(index) 사용
if let Some(val) = v.get(2) {
    println!("{}", val);
}

// 순회
for (i, val) in v.iter().enumerate() {  // 인덱스도 같이 반환
    println!("{}: {}", i, val);
}

// 값 수정
for val in v.iter_mut() {   // 가변 참조로 순회
    *val *= 2;
}
```

### HashMap (키-값 저장소)

```rust
use std::collections::HashMap;

let mut map = HashMap::new();
map.insert("name", "Alice");
map.insert("age", "25");

// 조회 - .get(key) 사용
if let Some(value) = map.get("name") {
    println!("{}", value);
}

// 키 있으면 수정, 없으면 삽입
map.insert("name", "Bob");

// entry API — 키 존재 여부 확인 후 작업
// or_insert() — 키가 없으면 삽입, 있으면 값을 반환 
*map.entry("level").or_insert(1) += 1;

// 순회
for (k, v) in &map {
    println!("{}: {}", k, v);
}
```

---

## 10. Trait (인터페이스)

여러 타입이 공통 메서드를 구현하도록 강제하는 것.

```rust
trait Animal {
    fn speak(&self);
    fn move_forward(&self);
    
    // 기본 구현
    fn sleep(&self) {
        println!("zzzz...");
    }
}

struct Dog;
struct Cat;

impl Animal for Dog {
    fn speak(&self) { println!("멍멍!"); }
    fn move_forward(&self) { println!("네 발로 뜀"); }
}

impl Animal for Cat {
    fn speak(&self) { println!("야옹!"); }
    fn move_forward(&self) { println!("살금살금"); }
}

fn main() {
    let dog = Dog;
    let cat = Cat;
    
    dog.speak();
    cat.speak();
    dog.sleep();
}

// Trait bound — 제네릭 타입 제약
fn make_sound<T: Animal>(animal: &T) {
    animal.speak();
}

// impl Trait — 함수 반환값 추상화
fn get_animal() -> impl Animal {
    Dog
}
```

---

## 11. Generic (제네릭)

타입을 고정하지 않고 코드 재사용.

```rust
// 함수 제네릭
fn first<T>(list: &[T]) -> Option<&T> {
    list.first()
}

// 구조체 제네릭
struct Container<T> {
    value: T,
}

impl<T> Container<T> {
    fn get(&self) -> &T {
        &self.value
    }
}

// Trait bound 여러 개
fn compare<T: std::fmt::Display + std::cmp::PartialOrd>(a: T, b: T) {
    if a > b {
        println!("{} > {}", a, b);
    }
}

// where 절 (복잡한 제약조건)
fn process<T, U>() 
where
    T: std::fmt::Debug,
    U: std::clone::Clone,
{
    // ...
}
```

---

## 12. 파일 I/O

### 읽기

```rust
use std::fs;

fn main() {
    let content = fs::read_to_string("file.txt")
        .expect("파일을 읽을 수 없음");
    
    for line in content.lines() {
        println!("{}", line);
    }
}
```

### 쓰기

```rust
use std::fs;
use std::io::Write;

fn main() {
    // 전체 덮어쓰기
    fs::write("output.txt", "Hello, Rust!")
        .expect("파일을 쓸 수 없음");
    
    /* 파일 끝에 추가
     * .create(true) — 파일이 없으면 생성
     * .append(true) — 파일 끝에 추가
     */
    let mut file = fs::OpenOptions::new()
        .create(true)
        .append(true)
        .open("log.txt")
        .expect("파일을 열 수 없음");
    
    file.write_all(b"로그 메시지\n")    // b"..."는 바이트 문자열
        .expect("쓰기 실패");
}
```

### JSON 처리

```rust
use serde::{Serialize, Deserialize};

#[derive(Debug, Serialize, Deserialize)]
struct Person {
    name: String,
    age: u32,
}

fn main() {
    // JSON → Struct
    let json = r#"{"name": "Alice", "age": 30}"#;
    let person: Person = serde_json::from_str(json).expect("파싱 실패");
    
    // Struct → JSON
    let p = Person {
        name: String::from("Bob"),
        age: 25,
    };
    let json = serde_json::to_string_pretty(&p).expect("변환 실패");
    println!("{}", json);
}
```

---

## 13. 비동기 (Async/Await)

하나의 스레드에서 여러 작업을 효율적으로 처리. **I/O 대기 시간** 활용.

```rust
use tokio::time::{sleep, Duration};

async fn fetch_data(id: u32) -> String {
    sleep(Duration::from_secs(1)).await;
    format!("Data {}", id)
}

async fn process() {
    // 순차 실행
    let a = fetch_data(1).await;
    let b = fetch_data(2).await;
    
    // 동시 실행 — join! (순서 보장 안 됨)
    let (a, b) = tokio::join!(
        fetch_data(1),
        fetch_data(2)
    );
    
    println!("{}, {}", a, b);
}

#[tokio::main]
async fn main() {
    process().await;    // 처리가 모두 완료될 때까지 대기
}
```

### spawn — 백그라운드 태스크

```rust
#[tokio::main]
async fn main() {
    let handle = tokio::spawn(async {
        println!("백그라운드 작업");
    });
    
    handle.await.expect("태스크 실패");
}
```

---

## 14. 멀티스레드 (병렬 처리)

실제 병렬 실행. **CPU 작업**에 유리.

```rust
use std::thread;

// 참조
fn main() {
    let handle = thread::spawn(|| {
        for i in 0..5 {
            println!("스레드: {}", i);
        }
    });
    
    for i in 0..5 {
        println!("메인: {}", i);
    }
    
    handle.join().unwrap();  // 스레드 종료 대기
}

// 소유권 이동 — move 클로저
fn main() {
    let data = vec![1, 2, 3];
    
    let handle = thread::spawn(move || {
        println!("{:?}", data);
    });
    
    handle.join().unwrap();
    // println!("{:?}", data);  // ❌ data는 이미 이동됨
}
```

---

## 15. 모듈 (Module)

코드를 논리적으로 분할.

### 동일 디렉토리

```
src/
 ├── main.rs
 ├── robot.rs
 ├── sensor.rs
 └── control.rs
```

```rust
/* ========== robot.rs ========== */
pub struct Robot {
    pub name: String,
    pub battery: i32,
}

/* ========== sensor.rs ========== */
pub fn read_sensor() -> i32 {
    10
}

/* ========== control.rs ========== */
pub fn decide(distance: i32) {
    if distance < 5 {
        println!("정지");
    } else {
        println!("이동");
    }
}

/* ========== main.rs ========== */
mod robot;
mod sensor;
mod control;

use sensor::read_sensor;
use control::decide;

fn main() {
    let distance = read_sensor();
    decide(distance);
}
```

### 폴더 모듈

```
src/
 ├── main.rs
 └── robot/
     ├── mod.rs
     ├── sensor.rs
     └── control.rs
```

```rust
/* ========== robot/mod.rs ========== */
pub mod sensor;
pub mod control;

/* ========== robot/sensor.rs ========== */
pub fn read_sensor() -> i32 {
    10
}

/* ========== robot/control.rs ========== */
pub fn decide(distance: i32) {
    if distance < 5 {
        println!("정지");
    } else {
        println!("이동");
    }
}

/* ========== main.rs ========== */
mod robot;

use robot::sensor::read_sensor;
use robot::control::decide;

fn main() {
    let distance = read_sensor();
    decide(distance);
}
```

---

## 16. Rust 프로젝트 관리 (Cargo)

```bash
# 새 프로젝트 생성
cargo new my_app

# 컴파일 (디버그)
cargo build

# 컴파일 + 실행
cargo run

# 최적화 빌드
cargo build --release

# 빌드 없이 문법만 검사
cargo check

# 정리
cargo clean
```
