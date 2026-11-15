# 9.1. panic!으로 복구 불가능한 에러 처리
```rust
fn main() {
    panic!("crash and burn");
}
```
직접 패닉 발생 매크로 실행

```rust
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```
패닉을 일으키는 범위 외 인덱스 호출

버퍼 초과 읽기 (buffer overread)라 하며, 접근이 허용되어서는 안 되는 데이터를 읽기 위해 어떤 공격자가 배열 뒤에 저장된 데이터를 읽어낼 요량으로 인덱스를 다루게 된다면, 이는 보안 취약점으로 이어질 수 있다

# 9.2. Result로 복구 가능한 에러 처리
```rust
enum Result<T, E> {
    Ok(T),  // 성공시
    Err(E), // 실패시
}
```
`T`와 `E`는 제네릭 타입 매개변수

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt"); // Result<T, E> 반환

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {:?}", error),
    };
}
```
`match` 표현식을 사용하여 반환 가능한 `Result` 배리언트들을 처리하기

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {  // kind 메서드를 호출하여 io::ErrorKind값을 얻음
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => {
                panic!("Problem opening the file: {:?}", other_error);
            }
        },
    };
}
```
조금 더 복잡한 코드의 예시

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt").unwrap();
}
```
`unwrap`은 `Ok`일때 내부값 반환, `Err`일때 `Panic!`호출

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")
        .expect("hello.txt should be included in this project");
}
```
`expect`는 `Err`시 지정한 에러 메시지를 출력

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```
함수 내에서 실패할 수 있는 연산(예: 파일을 여는 File::open) 뒤에 ?를 붙이는 에러 전파 숏컷

실패(Err)인 경우 발생한 에러 값을 반환

```rust
use std::fs::File;
use std::io::{self, Read};

// before
fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = String::new();

    File::open("hello.txt")?.read_to_string(&mut username)?;

    Ok(username)
}

// after
fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```
`read_to_string`은 위의 코드를 간단히 함

`?` 연산자가 `Result`, `Option` 혹은 `FromResidual`을 구현한 타입을 반환하는 함수에서만 사용될 수 있음

# 9.3. 언제 panic!을 쓰는가
실패할지도 모르는 함수를 정의할 때는 기본적으로 Result를 반환하는 것이 좋은 선택

코드가 어떤 계약, 보장, 불변성 등이 깨지는 '나쁜 상태'에 처했을 때는 panic!이 바람직

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess { value }
    }

    pub fn value(&self) -> i32 {
        self.value
    }
}
```
유효성을 위한 커스텀 타입 생성