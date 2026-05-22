# Rust

## 기본 구조

```rust
fn main() {
    // 불변 변수 — 선언 후 재할당 불가
    let x = 10;
    let y = "hello";                    // &str 타입 (문자열 참조)
    let z = String::from("hello");      // String 타입 (문자열 소유)
    let s1 = format!("{} world", z);    // hello world 문자열 만들어 변수에 저장 (String 타입)

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

## 타입 변환

```rust
// as — 숫자 변환 (정수/실수/문자)
let a: i32 = 10;
let b = a as f64;

// parse() — 문자열 -> 숫자 [반환값 : Result<i32, ParseIntError>]
let s = "10";
let n = s.parse::<i32>().unwrap();

// to_string() — 숫자 -> 문자열
let n = 10;
let s = n.to_string();
```

> 문자 <-> 숫자 변환은 ASCII 코드 기준

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
fn print_string(s: String) {
    println!("{}", s);
}

fn main() {
    // string 타입 — heap 데이터 (Copy 불가)
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

    print_string(name3);    // 함수로 소유권 이동
    print_string(&name2);   // 함수로 참조 전달
    println!("{}", name3);  // ❌ 에러
    println!("{}", name2);  // ✅ 정상
}
```

> Stack: 작고 크기가 정해진 값 저장(정수, 실수, 불린형 등)  
> Heap: 실행 중에 크기가 달라질 수 있는 데이터 저장(문자열 등)

---

## Borrowing (참조)

소유권을 넘기지 않고 값을 **빌려서** 사용하는 방식.

```rust
fn change(s: &mut String) { // 변경 가능한 참조
    s.push_str(" world");
}
fn print_distances(distances: &[f64]) { // 배열 참조
    for (index, distance) in distances.iter().enumerate() {
        println!("{}번 거리값: {} m", index, distance);
    }
}
fn sanitize_distances(distances: &mut [f64]) {  // 배열 변경 가능한 참조
    // iter_mut() — 변경 가능한 참조를 반환
    for distance in distances.iter_mut() {  
        if *distance < 0.0 {
            *distance = 0.0;
        }
    }
}

fn main() {
    let mut s = String::from("hello");
    let lidar_ranges = [1.2, 0.9, 0.45, 2.0, 0.6];
    let mut lidar_ranges = [1.2, -1.0, 0.45, 2.0, -0.5];

    print_distances(&lidar_ranges);
    sanitize_distances(&mut lidar_ranges);

    // 한 번에 하나의 mutable reference만 가능
    let r1 = &mut s;
    let r2 = &mut s; // ❌ 오류

    /* mutable과 immutable reference 동시에 불가
     * (immutable 참조 사용이 끝나면 mutable 참조 가능)
     */
    let r1 = &s;
    let r2 = &mut s;  // ❌ 오류

    change(&mut s);
    println!("{}", s);  // hello world
}

// dangling reference: 이미 사라진 데이터를 가리키는 참조
fn dangle() -> &String {
    let s = String::from("hello");
    &s  // ❌ s가 함수 종료 시 사라짐
}

// 올바른 방식 — 소유권 반환
fn no_dangle() -> String {
    let s = String::from("hello");
    s // ✅ 받은 소유권 다시 반환 
}

// &str 반환
fn get_default_status() -> &'static str {
    "READY"
}

```

> 함수 매개변수 `&str` — String도 받을 수 있고, 문자열 리터럴(&str)도 받을 수 있다.

---

## 조건문

```rust
if 조건 {
    실행
} else if 조건 {
    실행
} else {
    실행
}
```

---

## 반복문

### loop — 무한 반복

```rust
fn main() {
    let mut count = 0;

    loop {
        count += 1;
        println!("{}", count);

        if count == 5 {
            break;
        }
    }
}
```

### while

```rust
fn main() {
    let mut battery = 100;

    while battery > 0 {
        println!("배터리: {}", battery);
        battery -= 20;
    }
}
```

### for

```rust
fn main() {

    let distances = [1.2, 0.9, 0.45, 2.0, 0.6];

    /* iter() — 벡터를 참조로 순회
     * enumerate() — 인덱스와 값을 함께 반환
     */
    for (index, distance) in distances.iter().enumerate() {
        println!("{}번 센서값: {} m", index, distance); // println!이 참조를 자동 처리
    }

    for (index, distance) in distances.iter().enumerate() {
        if is_dangerous_distance(*distance) {   // 함수에는 역참조 사용
            println!("{}번 값 위험: {} m", index, distance);
        } else {
            println!("{}번 값 안전: {} m", index, distance);
        }
    }

    for i in 0..5 { // 0부터 4까지 반복
        println!("i = {}", i);
    }

    for i in 0..=5 { // 0부터 5까지 반복
        println!("i = {}", i);
    }
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
    // 생성자
    fn new(name: String, age: u32) -> User {
        User {
            name,
            age,
        }
    }
    // 읽기 전용
    fn introduce(&self) {   
        println!("안녕하세요, 저는 {}입니다", self.name);
    }
    // 변경 가능
    fn change_name(&mut self, new_name: String) {
        self.name = new_name;
    }
}

fn main() {
    let user = User {
        name: String::from("Lee"),
        age: 25,
    };
    let user1 = User::new(String::from("Kim"), 30);

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

### Option

값의 **존재 또는 부재**를 표현하는 enum.  
Rust에는 `null`이 없으며, 대신 `Option`을 사용한다.

```rust
enum Option<T> {
    Some(T), // 값이 있음
    None,    // 값이 없음
}
```

```rust
fn find_first_dangerous_distance(ranges: &[f64]) -> Option<f64> {
    for distance in ranges {
        if *distance < 0.7 {
            return Some(*distance);
        }
    }
    None
}

fn main() {
    let lidar_ranges = [1.2, 0.9, 0.45, 2.0];

    let result = find_first_dangerous_distance(&lidar_ranges);

    // match — 모든 경우를 다루어야 함
    match result {
        Some(distance) => println!("첫 번째 위험 거리: {} m", distance),
        None => println!("위험 거리 없음"),
    }
    // if let ~ else
    if let Some(distance) = result {
        println!("감지된 위험 거리: {} m", distance);
    } else {
        println!("위험 거리 없음");
    }
}
```

---

### 에러처리 (Result)

```rust
enum Result<T, E> {
    Ok(T),      // 성공, 결과값 있음
    Err(E),     // 실패, 오류정보 있음
}
```

```rust
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

동적 배열
- 같은 타입만 저장 가능

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

// 안전한 접근 — .get(index)
match v.get(2) {
    Some(value) => println!("{}", value),
    None => println!("없음"),
}

// 참조 반복문 — 값 수정
for i in &mut v {
    *i += 10;
}
```

```rust
#[derive(Debug)]
struct MotorStatus {
    id: u32,
    speed: f64,
    temperature: f64,
}

fn main() {
    // 구조체 목록 저장
    let mut motors: Vec<MotorStatus> = Vec::new();

    motors.push(MotorStatus {
        id: 1,
        speed: 1.2,
        temperature: 45.0,
    });

    motors.push(MotorStatus {
        id: 2,
        speed: 0.8,
        temperature: 52.0,
    });

    motors.push(MotorStatus {
        id: 3,
        speed: 0.0,
        temperature: 38.0,
    });

    for motor in motors.iter() {
        println!("{:?}", motor);
    }
}
```

```rust
fn main() {
    // 문자열 저장
    let mut logs: Vec<String> = Vec::new();

    logs.push(String::from("[INFO] Robot initialized"));
    logs.push(String::from("[INFO] Sensor connected"));
    logs.push(String::from("[WARNING] Obstacle detected"));

    for log in logs.iter() {
        println!("{}", log);
    }
}
```

### Hash Map

키-값 저장소
- HashMap은 순서를 보장하지 않는다

```rust
use std::collections::HashMap;

// 생성
let mut map = HashMap::new();
map.insert("name", "Kim");
map.insert("age", "30");

// 값 수정
map.insert("name", "Lee");

// ownership 주의
let key = String::from("name");
map.insert(key, String::from("Kim"));
// key ❌ 사용 불가 (move 발생)

// 값 가져오기 — .get(key) [반환값 : Option<&String>]
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

    /* {:?} : 디버그 출력 — Vec, 구조체(struct), enum 같은 타입들 출력 가능
     * vec는 #[derive(Debug)] 필요 없음
     * struct와 enum은 #[derive(Debug)] 필요
     */
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

## 모듈 (mod)

코드를 나누는 단위

```
src/
 ├── main.rs
 └── robot/
     ├── robot.rs
     ├── sensor.rs
     └── control.rs
```

> rs 파일이 같은 폴더에 있을 경우 각각 mod  
> 폴더 계층 구조를 가지고 있으면 하나의 rs 파일에 pub mod

```rust
/* ============= robot.rs ================= */
// pub mod — main.rs에서 접근 가능하게
pub mod sensor;
pub mod control;

// pub — 외부에서 접근 가능
pub fn say_hello() {
    println!("로봇 시작");
}

// pub 필드 — pub struct 안의 필드와 메서드도 pub으로 선언해야 외부에서 접근 가능
#[derive(Debug)]
pub struct RobotConfig {
    pub robot_name: String,
    pub max_speed: f64,
    pub safety_distance: f64,
    pub battery_warning_level: u32,
}

/* ============= main.rs ================= */
mod robot;
use robot::say_hello;
use robot::sensor::read_sensor;
use robot::control::decide;

fn main() {
    robot::say_hello(); // 모듈 이름으로 접근
    say_hello();        // use로 가져와서 바로 접근
}
```

---

## Trait

공통 기능 이름을 약속하는 것

```rust
trait DeviceInfo {
    fn device_name(&self) -> String;
    fn is_healthy(&self) -> bool;
}

trait StatusReport {
    fn device_name(&self) -> String;

    // Default Implementation
    fn status_report(&self) {
        println!("장치 이름: {}", self.device_name());
        println!("기본 상태 보고서입니다.");
    }
}

struct LidarSensor {
    name: String,
    distance: f64,
}

struct Motor {
    id: u32,
    speed: f64,
    temperature: f64,
}

impl StatusReport for LidarSensor {
    fn device_name(&self) -> String {
        self.name.clone()
    }

    // Default Implementation 재정의
    fn status_report(&self) {
        println!("--- LiDAR Status Report ---");
        println!("센서 이름: {}", self.name);
        println!("측정 거리: {} m", self.distance);
    }
}

impl DeviceInfo for LidarSensor {
    fn device_name(&self) -> String {
        self.name.clone()
    }

    fn is_healthy(&self) -> bool {
        self.distance >= 0.0
    }
}

impl StatusReport for Motor {
    fn status_report(&self) {
        println!("--- Motor Status Report ---");
        println!("모터 ID: {}", self.id);
        println!("속도: {} m/s", self.speed);
        println!("온도: {} C", self.temperature);
    }
}

impl DeviceInfo for Motor {
    fn device_name(&self) -> String {
        format!("motor_{}", self.id)
    }

    fn is_healthy(&self) -> bool {
        self.temperature <= 80.0
    }
}

fn main() {
    let lidar = LidarSensor {
        name: String::from("front_lidar"),
        distance: 0.45,
    };

    let motor = Motor {
        id: 1,
        speed: 1.2,
        temperature: 65.0,
    };

    // trait을 사용하면 같은 이름의 메서드를 호출할 수 있다.
    lidar.status_report();
    motor.status_report();

    println!("장치 이름: {}", lidar.device_name());
    println!("정상 여부: {}", lidar.is_healthy());

    println!("장치 이름: {}", motor.device_name());
    println!("정상 여부: {}", motor.is_healthy());
}
```

---

## Generic

타입을 고정하지 않고 코드를 재사용하는 문법

```rust
use std::fmt::Display;

fn print_value<T: Display>(value: T) {  // T 타입은 출력 가능한 타입만 허용 (Trait bound)
    println!("{}", value);
}

fn main() {
    print_value(10);
    print_value(3.14);
    print_value("robot");
}
```

```rust
trait StatusReport {
    fn status_report(&self);
}

struct LidarSensor {
    name: String,
    distance: f64,
}

struct Motor {
    id: u32,
    speed: f64,
}

impl StatusReport for LidarSensor {
    fn status_report(&self) {
        println!("LiDAR {} 거리: {} m", self.name, self.distance);
    }
}

impl StatusReport for Motor {
    fn status_report(&self) {
        println!("Motor {} 속도: {} m/s", self.id, self.speed);
    }
}

fn print_report<T: StatusReport>(device: &T) {  // T 타입은 StatusReport trait을 구현한 타입만 허용 (Trait bound)
    device.status_report();
}

fn print_report(device: &impl StatusReport) {  // Trait bound를 더 간결하게 표현
    device.status_report();
}

fn main() {
    let lidar = LidarSensor {
        name: String::from("front_lidar"),
        distance: 0.45,
    };

    let motor = Motor {
        id: 1,
        speed: 1.2,
    };

    print_report(&lidar);
    print_report(&motor);
}
```

```rust
fn debug_and_report<T>(device: &T)
where
    T: StatusReport + Debug,    // 조건 여러개
{
    ...
}
```

---

## 파일

### 쓰기

#### fs::write

```rust
use std::fs;

fn main() {
    let log = "[INFO] Robot initialized\n[INFO] Sensor connected\n";

    fs::write("robot_log.txt", log)
        .expect("로그 파일을 저장할 수 없습니다."); // 기존 내용이 있다면 덮어쓰기

    println!("robot_log.txt 파일 저장 완료");
}
```

#### OpenOptions

```rust
use std::fs::OpenOptions;
use std::io::Write;

fn main() {
    let log_line = "[INFO] Robot heartbeat received\n";

    let mut file = OpenOptions::new()
        .create(true)               // 파일 없으면 새로 만든다
        .append(true)               // 기존 내용 뒤에 추가
        .open("robot_runtime.log")  // 해당 파일을 연다
        .expect("로그 파일을 열 수 없습니다.");

    file.write_all(log_line.as_bytes()) // as_bytes() : 문자열을 바이트로 변환
        .expect("로그를 파일에 쓸 수 없습니다.");

    println!("로그 추가 완료");
}
```

#### Json

```rust
use serde::Deserialize; // 외부 데이터를 Rust Struct로 바꿀 수 있게 해주는 기능
use serde::Serialize;   // Rust Struct를 외부 데이터 형식으로 바꿀 수 있게 해주는 기능

#[derive(Debug, Deserialize, Serialize)]
struct RobotConfig {
    robot_name: String,
    max_speed: f64,
    safety_distance: f64,
    battery_warning_level: u32,
}

fn load_robot_config(path: &str) -> Result<RobotConfig, String> {
    let content = fs::read_to_string(path)  // Json 파일 읽기
        .map_err(|error| format!("설정 파일 읽기 실패: {}", error))?;

    let config: RobotConfig = serde_json::from_str(&content)  // Json 파싱
        .map_err(|error| format!("JSON 파싱 실패: {}", error))?;

    Ok(config)
}

fn save_analysis_result(path: &str, result: &SensorAnalysisResult) -> Result<(), String> {
    let json_text = serde_json::to_string_pretty(result)  // Struct -> JSON
        .map_err(|error| format!("JSON 변환 실패: {}", error))?;

    fs::write(path, json_text)  // Json 파일 저장
        .map_err(|error| format!("분석 결과 파일 저장 실패: {}", error))?;

    Ok(())
}

fn main() {
    let json_text = r#"
    {
        "robot_name": "mobile_robot_01",
        "max_speed": 1.5,
        "safety_distance": 0.7,
        "battery_warning_level": 20
    }
    "#;
    let result = SensorAnalysisResult {
        robot_name: String::from("mobile_robot_01"),
        danger_count: 2,
        min_distance: 0.45,
        need_stop: true,
    };
    // JSON -> Struct
    let config: RobotConfig = serde_json::from_str(json_text)
        .expect("JSON을 RobotConfig로 변환할 수 없습니다.");

    // Struct -> JSON
    let json_text = serde_json::to_string_pretty(&result)
        .expect("Struct를 JSON으로 변환할 수 없습니다.");

    println!("{:#?}", config);  // {:#?} : 예쁘게 줄바꿈 출력
    println!("로봇 이름: {}", config.robot_name);
    println!("최대 속도: {} m/s", config.max_speed);
}
```

### 읽기

```rust
use std::fs;

fn find_config_value(content: &str, key: &str) -> Option<String> {
    for line in content.lines() {           // .lines() : 문자열을 줄 단위로 분리
        let mut parts = line.split('=');    // .split() : 문자열을 특정 문자 기준으로 분리

        let config_key = parts.next()?;     // .next() : 다음값을 반환하는 함수, 반환값 Option<&str>
        let config_value = parts.next()?;

        if config_key == key {
            return Some(config_value.to_string());
        }
    }

    None
}

fn read_f64_config(content: &str, key: &str) -> Result<f64, String> {
    let value_text = find_config_value(content, key)
        .ok_or(format!("{} 설정이 없습니다.", key))?;   // .ok_or(error값) : Option을 Result로 변환

    let value = value_text
        .parse::<f64>()
        .map_err(|_| format!("{} 설정값을 숫자로 변환할 수 없습니다.", key))?; // .map_err(새로운 error값) : 에러값 변환 [|_| : 매개변수는 받되 사용하지 않겠다]

    Ok(value)
}

fn main() {
    let content = fs::read_to_string("robot_config.txt")
        .expect("robot_config.txt 파일을 읽을 수 없습니다.");

    match read_f64_config(&content, "max_speed") {
        Ok(max_speed) => println!("최대 속도 설정: {} m/s", max_speed),
        Err(error) => println!("설정 오류: {}", error),
    }
}
```

---

## 비동기

어떤 작업이 기다리는 동안 다른 작업을 진행할 수 있는 방식
- **async** : 비동기 함수를 정의
- **await** : 비동기 함수의 결과를 기다림
- **tokio** : 비동기 런타임
    - 런타임 : 프로그램이 실행되는 동안 뒤에서 일을 처리해주는 시스템

### join!

독립적인 비동기 작업 동시 실행

```rust
/* async fn을 호출하면 Future가 만들어진다.
 * Future는 비동기 작업의 결과를 나타내는 객체.
 * Future는 await 하거나 spawn 해야 실행된다.
 */
use tokio::time::{sleep, Duration};

async fn read_lidar_data() {
    println!("LiDAR 데이터 읽기 시작");
    sleep(Duration::from_secs(2)).await;
    println!("LiDAR 데이터 읽기 완료");
}

async fn read_motor_status() {
    println!("Motor 상태 읽기 시작");
    sleep(Duration::from_secs(2)).await;
    println!("Motor 상태 읽기 완료");
}

async fn read_battery_status() {
    println!("Battery 상태 읽기 시작");
    sleep(Duration::from_secs(2)).await;
    println!("Battery 상태 읽기 완료");
}

async fn read_lidar_min_distance() -> f64 {
    println!("LiDAR 최소 거리 계산 중...");
    sleep(Duration::from_secs(1)).await;
    0.45
}

#[tokio::main]
async fn main() {
    println!("동시 실행 시작");

    let (
        min_distance,
        _,
        _,
        _,
    ) = tokio::join!(   // 출력 순서 보장 안됨
        read_lidar_min_distance(),
        read_lidar_data(),
        read_motor_status(),
        read_battery_status()
    );

    println!("동시 실행 종료");
}
```

### spawn

비동기 작업을 별도 태스크로 실행

```rust
use tokio::time::{sleep, Duration, interval};

async fn sensor_task() {
    for i in 1..=5 {
        println!("[Sensor Task] LiDAR 데이터 수집 {}", i);
        sleep(Duration::from_millis(500)).await;    // 주기
    }
}

async fn diagnostic_task() {
    for i in 1..=5 {
        println!("[Diagnostic Task] 시스템 진단 {}", i);
        sleep(Duration::from_millis(700)).await;
    }
}

async fn sensor_loop() {
    let mut ticker = interval(Duration::from_millis(500)); // 500ms 마다 tick 발생

    for count in 1..=5 {
        ticker.tick().await;
        println!("[Sensor Loop] tick {}", count);
    }
}

async fn read_sensor_value(sensor_name: &str) -> Result<f64, String> {
    sleep(Duration::from_millis(500)).await;

    if sensor_name == "lidar" {
        Ok(0.45)
    } else {
        Err(format!("알 수 없는 센서: {}", sensor_name))
    }
}

#[tokio::main]
async fn main() {

    sensor_loop().await; // spawn X

    println!("로봇 비동기 태스크 시작");

    // 출력 순서 보장 안됨
    let sensor_handle = tokio::spawn(sensor_task());
    let diagnostic_handle = tokio::spawn(diagnostic_task());

    sensor_handle.await.expect("sensor_task 실패");
    diagnostic_handle.await.expect("diagnostic_task 실패");

    println!("로봇 비동기 태스크 종료");

    match read_sensor_value("lidar").await {
        Ok(value) => println!("센서값: {}", value),
        Err(error) => println!("센서 오류: {}", error),
    }
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

- **Package**: Cargo가 관리하는 프로젝트 단위 (Cargo.toml이 있는 Rust 프로젝트)
- **Binary Crate**: 실행 파일을 만드는 Crate (src/main.rs)
- **Library Crate**: 라이브러리를 만드는 Crate (src/lib.rs)
- **module**: 코드 안에서 기능별로 코드를 나누는 단위

### `cargo new` 직후

```
my_project/     # Package
├── Cargo.toml  # 프로젝트 설정 파일
├── src/
│   └── main.rs
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
└── target/                 # 빌드 결과물 저장
    └── release/
        └── my_project      # 최적화된 실행 파일
```