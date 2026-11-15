# 6.1. 열거형
```rust
enum RPC {
    Rock,
    Scissors,
    Paper,
}
```
열거형은 어떤 값이 여러 개의 가능한 값의 집합 중 하나라는 것을 나타내는 방법

`주먹 = RPC::Rock`처럼 인스턴스를 생성

`fn ready(rpc : RPC) {}`함수로도 생성가능

```rust
    enum IpAddr {
        V4(u8, u8, u8, u8),
        V6(String),
    }

    let home = IpAddr::V4(127, 0, 0, 1);

    let loopback = IpAddr::V6(String::from("::1"));
```
열거형의 각 배리언트에 직접 데이터를 붙임으로써, 구조체를 생략가능

또한 각 배리언트는 다른 타입과 다른 양의 연관된 데이터를 가질 수 있다

```rust
fn main() {
    enum Message {
        Quit,
        Move { x: i32, y: i32 },
        Write(String),
        ChangeColor(i32, i32, i32),
    }

    impl Message {
        fn call(&self) {
            // 메서드 본문이 여기 정의될 것입니다
        }
    }

    let m = Message::Write(String::from("hello"));
    m.call();
}
```
열거형도 `impl`블록 사용가능

```rust
enum Option<T> {
    None,
    Some(T),
}
```
`Option<T>`로 null을 구현

```rust
    let some_number = Some(5);
    let some_char = Some('e');

    let absent_number: Option<i32> = None;
```
Option<T>와 T(T는 어떤 타입이던 될 수 있음)이 다른 타입이기 때문에, 컴파일러는 Option<T> 값을 명백하게 유효한 값처럼 사용하지 못하도록 함

# 6.2 매치 제어 흐름
```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```
매칭이 된다면 => 다음의 코드를 실행

```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // --생략--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}

fn main() {
    value_in_cents(Coin::Quarter(UsState::Alaska));
}
```
복잡한 패턴의 매칭의 예

```rust
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            None => None,
            Some(i) => Some(i + 1),
        }
    }

    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
```
러스트의 매치는 철저하기에(exhaustive) 모든 가능성을 샅샅이 다루어야 한다

```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => reroll(),  // 포괄 패턴
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn reroll() {}
```
`_`는 어떠한 값이라도 매칭되지만, 그 값을 바인딩하지는 않는 특별한 패턴

```rust
// before
    let config_max = Some(3u8);
    match config_max {
        Some(max) => println!("The maximum is configured to be {}", max),
        _ => (),
    }

// after
    let config_max = Some(3u8);
    if let Some(max) = config_max {
        println!("The maximum is configured to be {}", max);
    }
```
if와 let을 조합하여 하나의 패턴만 매칭시키고 나머지 경우는 무시하도록 값을 처리하는 간결한 방법

즉, if let은 한 패턴에 매칭될 때만 코드를 실행하고 다른 경우는 무시하는 match 문을 작성할 때 사용하는 문법 설탕
```rust
//before
    let mut count = 0;
    match coin {
        Coin::Quarter(state) => println!("State quarter from {:?}!", state),
        _ => count += 1,
    }

// after
    let mut count = 0;
    match coin {
        Coin::Quarter(state) => println!("State quarter from {:?}!", state),
        _ => count += 1,
    }
```
또 다른 예