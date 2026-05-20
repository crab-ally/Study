# Rust

## 기본 구조

```rust
fn main() {
    // 불변 변수 — 선언 후 재할당 불가
    let x = 10;
    let y = "hello";                // &str 타입 (문자열 참조)
    let z = String::from("hello");  // String 타입 (문자열 소유)

    // 가변 변수 — mut 키워드로 재할당 허용
    let mut h = 20;
    let mut s = String::from("hello");
    
    // 자료형 명시
    let a: i32 = 100;
    let b: f64 = 3.14;
    let c: &str = "Hello";  // 객체

    // 상수 — 절대 바뀌지 않는 값
    const PI: f64 = 3.141592;

    // 출력 — {} 자리에 값이 순서대로 삽입됨
    println!("x: {}, y: {}, c: {}", x, y, c);
}
```

---

## 데이터 타입

| 분류 | 타입 |
|------|------|
| 부호 있는 정수형 | `i32`, `i64` |
| 부호 없는 정수형 | `u32`, `u64` |
| 실수형 | `f32`,`f64` |
| 불리언 | `bool` |
| 문자형 | `char` |
| 문자열 | `&str`, `String` |

> `&str`: 읽기 전용. 변경 불가. 소유권 없음.  
> `String`: 읽기+쓰기 가능. 변경 가능. 소유권 있음. (`mut` 키워드 필요)

---

## Shadowing

같은 이름으로 변수를 **재선언**해 이전 값을 덮어쓰는 기능.  
`mut`와 달리 **타입 변경**도 가능하다.

```rust
fn main() {
    // 같은 타입으로 shadowing
    let x = 10;
    let x = x + 5;
    println!("{}", x); // 15

    // 다른 타입으로 shadowing
    let y = "Hi";
    let y = y.len();
    println!("{}", y); // 2
}
```

---

## 소유권 (Ownership)

값은 **단 하나의 소유자**만 가질 수 있으며,
소유권이 이동(move)하면 이전 변수는 더 이상 사용할 수 없다.

```rust
fn main() {
    // string 타입 — heap 데이터
    let name = String::from("robot");
    let name2 = name;           // move — name의 소유권이 name2로 이동
    let name3 = name2.clone();  // clone — 값을 복사해 새 소유권 생성

    // 정수형, 불린형 — stack 데이터 (Copy 가능)
    let s1 = 10;
    let s2 = s1; // s1도 계속 사용 가능

    // scope 벗어나면 메모리 자동 해제
    {
        let s = String::from("hello");
    } // 여기서 자동으로 free

    println!("{}", name);  // ❌ 컴파일 에러 — 소유권 없음
    println!("{}", name2); // ✅ 정상
    println!("{}", name3); // ✅ 정상
}
```

---

## Borrowing (참조)

소유권을 넘기지 않고 값을 **빌려서** 사용하는 방식.

```rust
fn main() {
    let mut s = String::from("hello");

    // 한 번에 하나의 mutable reference만 가능
    let r1 = &mut s;
    let r2 = &mut s; // ❌ 오류

    // mutable과 immutable reference 동시에 불가
    let r1 = &s;      // immutable
    let r2 = &mut s;  // ❌ 오류

    change(&mut s);
    println!("{}", s);  // hello world
}

fn change(s: &mut String) {
    s.push_str(" world");
}

// dangling reference — 컴파일 에러
fn dangle() -> &String {
    let s = String::from("hello");
    &s  // ❌ s가 함수 종료 시 사라짐
}

// 올바른 방식 — 소유권 반환
fn no_dangle() -> String {
    let s = String::from("hello");
    s // ✅ 소유권 이동
}

```

---

## 함수

```rust
// 기본형
fn print_robot_status() {
    println!("로봇 상태를 출력합니다.");
}

// 매개변수 — 타입 명시 필수
fn print_battery(level: u32) {
    println!("배터리 잔량: {}%", level);
}

// 반환값 — 마지막 표현식에 세미콜론 없음
fn add(a: i32, b: i32) -> i32 {
    a + b
}

// 명시적 반환
fn sub(a: i32, b: i32) -> i32 {
    return a - b;
}
```

---

## 구조체 (struct)

```rust
struct User {
    name: String,
    age: u32,
}

// impl — 구조체에 메서드를 추가하는 문법
impl User {
    fn introduce(&self) {
        println!("안녕하세요, 저는 {}입니다", self.name);
    }
}

fn main() {
    let user = User {
        name: String::from("Lee"),
        age: 25,
    };

    user.introduce();
}
```

## enum & match

`enum` — 여러 종류의 값 중 하나를 표현하는 타입.  
`match` — 열거형의 **모든** 경우를 다루어야 하며, 그렇지 않으면 컴파일 에러 발생.

```rust
enum Message {
    Text(String),
    Number(i32),
}

fn main() {
    let msg = Message::Text(String::from("hello"));

    match msg {
        Message::Text(s) => println!("문자: {}", s),
        _ => println!("다른 타입"), // _는 나머지 모든 경우를 의미
    }
}
```

---

## Option

값의 **존재 또는 부재**를 표현하는 enum.  
Rust에는 `null`이 없으며, 대신 `Option`을 사용한다.

```rust
enum Option<T> {
    Some(T), // 값이 있음
    None,    // 값이 없음
}

fn main() {
    let value = Some(10);
    let nothing: Option<i32> = None;

    match value {
        Some(v) => println!("값이 있습니다: {}", v),
        None => println!("값이 없습니다"),
    }
}
```

---

## 에러처리 (Result)

```rust
enum Result<T, E> {
    Ok(T),      // 성공
    Err(E),     // 실패
}

fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err(String::from("0으로 나눌 수 없음"))
    } else {
        Ok(a / b)
    }
}

fn main() {
    let value = Some(10);
    let nothing: Option<i32> = None;

    // ? — 실패하면 즉시 함수 종료, 성공하면 값 반환
    let v = value?; // 10
    let v2 = nothing?; // return None

    // unwrap — 값이 있으면 반환, 없으면 에러(panic)
    println!("{}", value.unwrap()); // 10
    println!("{}", nothing.unwrap()); // ❌ 에러

    // expect — unwrap + 커스텀 에러 메시지
    println!("{}", value.expect("값이 없습니다"));
    println!("{}", nothing.expect("값이 없습니다")); // ❌ 에러 메시지 출력

    match divide(10, 2) {
        Ok(result) => println!("결과: {}", result),
        Err(error) => println!("에러: {}", error),
    }
}
```

---

## 컬렉션 (Vec / HashMap)

### Vector (Vec)

리스트(배열처럼 사용)

```rust
// 생성
let v = vec![1, 2, 3];		// 불변
let mut v1 = Vec::new();	// 가변

// 값 추가
v1.push(10);
v1.push(20);

// 값 접근
println!("{}", v[0]);	// 1
println!("{}", v[10]);	// 💥 panic

// 안전한 접근 — .get()
match v.get(10) {
    Some(value) => println!("{}", value),
    None => println!("없음"),
}

// 참조 반복문 — 값 수정
let mut v = vec![1, 2, 3];

for i in &mut v {
    *i += 10;
}
```

### Hash Map

키-값 저장소

```rust
use std::collections::HashMap;

// 생성
let mut map = HashMap::new();
map.insert("name", "Kim");
map.insert("age", "30");

// ownership 주의
let key = String::from("name");
map.insert(key, String::from("Kim"));
// key ❌ 사용 불가 (move 발생)

// 값 가져오기 — .get()
match map.get("name") {
    Some(v) => println!("{}", v),
    None => println!("없음"),
}

// 참조 반복문
for (k, v) in &map {
    println!("{}: {}", k, v);
}
```

---

#### entry API

키가 있으면 수정, 없으면 생성 — 반환값은 mutable reference

```rust
use std::collections::HashMap;

fn main() {
    let words = vec!["apple", "banana", "apple", "orange", "banana"];

    let mut count = HashMap::new();

    for word in words {
        *count.entry(word).or_insert(0) += 1;
    }

    println!("{:?}", count); // {"apple": 2, "banana": 2, "orange": 1}
}
```

---

## 비동기(async) & 병렬 처리(thread)

### 비동기(async)

하나의 스레드에서 여러 작업 처리. (I/O 작업)  
기다리는 동안 다른 일 먼저 하기.

```rust
use tokio;

#[tokio::main]      // #: 컴파일러에게 추가 지시를 주는 문법
async fn main() {
    hello().await;  // .await — 비동기 작업이 끝날 때까지 대기
}

async fn hello() {
    println!("Hello async!");
}
```

### 병렬 처리(thread)

실제 병렬 실행 (CPU 작업)

```rust
use std::thread;

fn main() {
    let handle = thread::spawn(|| {     // 스레드 시작
        for i in 1..5 {
            println!("spawned thread: {}", i);
        }
    });

    for i in 1..3 {     // 1 이상 3 미만
        println!("main thread: {}", i); // 스레드와 main의 출력 순서는 랜덤
    }

    handle.join().unwrap();     // 스레드 끝날 때까지 대기

    println!("main end");       // join() 때문에 스레드가 끝난 후 실행
}

fn main() {
    let s = String::from("hello");

    // move — s의 소유권을 클로저로 이동 (없으면 main이 먼저 끝날 시 s가 사라짐)
    let handle = thread::spawn(move || {
        println!("{}", s);
    });

    handle.join().unwrap();
}
```

---

## 실행 — Cargo

Cargo는 Rust의 공식 빌드 시스템 겸 패키지 매니저다.

| 명령어 | 설명 |
|--------|------|
| `cargo new <name>` | 새 프로젝트 생성 |
| `cargo check` | 컴파일 가능 여부만 검사 (빠름) |
| `cargo build` | 컴파일 → 디버그용 실행 파일 생성 |
| `cargo run` | 컴파일 + 실행을 한 번에 |
| `cargo build --release` | 최적화된 릴리즈 빌드 생성 |
| `cargo clean` | `target/` 폴더 전체 삭제 |

--

## 프로젝트 구조

### `cargo new` 직후

```
my_project/
├── Cargo.toml   # 프로젝트 설정 파일
├── src/
│   └── main.rs
├── .git/
└── .gitignore
```

### `cargo build` 후

```
my_project/
├── Cargo.lock   # 의존성 정보 파일
└── target/
    └── debug/
        ├── my_project      # 실행 파일
        └── ...
```

### `cargo build --release` 후

```
my_project/
└── target/
    └── release/
        └── my_project      # 최적화된 실행 파일
```