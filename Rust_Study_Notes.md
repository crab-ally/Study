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

> `&str`: **읽기만 가능**. 변경 불가능. 소유권 없음.  
> `String`: **읽기+쓰기 가능**. 변경 가능. 소유권 있음. (mut 키워드 필요)

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
    let name = String::from("robot");
    let name2 = name; // 소유권이 name → name2 로 이동

    println!("{}", name);  // ❌ 컴파일 에러 — 소유권 없음
    println!("{}", name2); // ✅ 정상
}
```

---

## 함수

```rust
// 함수 기본형
fn print_robot_status() {
    println!("로봇 상태를 출력합니다.");
}

// 매개변수가 있는 함수 — 매개변수 타입 필수
fn print_battery(level: u32) {
    println!("배터리 잔량: {}%", level);
}

// 반환값이 있는 함수 — 세미콜론 X
fn add(a: i32, b: i32) -> i32 {
    a + b
}

// 반환값이 있는 함수 — 명시적 반환
fn sub(a: i32, b: i32) -> i32 {
    return a - b;
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