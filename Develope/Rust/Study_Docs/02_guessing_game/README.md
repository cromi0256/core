# 2. 추리게임
이전에 배운대로 새 프로젝트를 만든다
```PowerShell
cargo new guessing_game
```

# 추릿값 처리
이제 src/main.rs파일을 다시 작성한다
```rust
use std::io;    // 표준 라이브러리 std에서 io를 가져옴

fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();  // mut는 가변성 변수 String타입의 연관함수; 비어있는 새로운 문자열 인스턴스

    io::stdin()
        .read_line(&mut guess)  // &은 참조자로 가변성 변수를 받음
        .expect("Failed to read line"); // 위의 작업이 실패 시 출력

    println!("You guessed: {guess}");   // {} : 자리표시자
}
```
사용자의 입력값을 받아 그대로 출력하는 함수

# 비밀번호 생성하기

크레이트는 러스트 코드 파일들의 모음

Cargo.toml [dependencies]에 rand = "0.8.5"를 추가 > cargo build로 의존성 추가

Cargo.lock은 재현 가능한 빌드를 보장

```rust
use std::io;
use rand::Rng;  // Rng는 난수 생성기를 구현한 메서드들을 정의한 트레이트

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);  // 1과 100사이의 랜덤값 생성

    println!("The secret number is: {secret_number}");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {guess}");
}
```

# 추릿값 비교

```rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    // --생략--
    let guess: u32 = guess.trim().parse().expect("Please type a number!");  // 입력시의 빈문자열을 trim하고 u32타입 정수형으로 바꿈

    println!("You guessed: {guess}");

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }   // std::cmp::Ordering이라는 타입을 가져옴
}
```

# 반복문 사용
```rust
    // --생략--

    println!("The secret number is: {secret_number}");

    loop {
        println!("Please input your guess.");

        // --생략--

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        // --생략--

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```
loop로 반복하며 break에 탈출

continue로 숫자가 아닌값은 무시하고 재루프

# 정리
이 프로젝트는 let, match, 메서드, 연관 함수, 외부 크레이트 사용과 같이 많은 새로운 러스트 개념들을 소개하기 위한 실습