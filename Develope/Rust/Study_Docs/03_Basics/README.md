# 3.1. 변수의 가변성
```rust
fn main() {
    let x = 5;
    println!("The value of x is: {x}");
    x = 6;  // 오류의 원인;
    println!("The value of x is: {x}");
}
```
이 파일은 컴파일 되지 않는다

그 이유는 `mut`(mutable)가 붙지 않은 불변 변수이기 때문이다

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```
상수는 const로 설정하며(mut 혼용 불가) 값의 타입이 명시되어야 한다

```rust
fn main() {
    let x = 5;

    let x = x + 1;  // x = 5는 가려지고 x = x(x=5) + 1 인 x = 6 이 남는다

    {
        let x = x * 2;
        println!("The value of x in the inner scope is: {x}");  // 이 스코프{}에서 x = 12 
    }

    println!("The value of x is: {x}"); // 스코프를 벗어나면 다시 x = 6
}
```
섀도잉은 기존 변수를 덮는 역할을 한다

```rust
    let spaces = "   "; // 문자열
    let spaces = spaces.len();  // 정수형
```
섀도잉은 또한 타입을 바꿀수도 있다(mut설정으로는 타입변경 불가)

# 3.2. 데이터 타입
정수형 : int/uint, 8*n bit(255, 65535, 4294967295, 2^64-1)

부동소수점 : 기본값 f64

수치연산 : 파이썬과 유사하지만 정수 나눗셈은 0에 가까운 방향으로 절삭

부울린(bool) : true false

문자(char) : 단 하나의 4바이트 문자(작은 따옴표 사용) vs 문자열(&str, 큰 따옴표)

복합타입:
* 튜플
```rust
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);  // 다른 타입 혼용 가능, 고정 크기, 소괄호

    let five_hundred = x.0; // 파이썬 x[0]과 같이 .0로 접근

    let six_point_four = x.1;

    let one = x.2;
}   // 빈 튜플은 유닛이라고 함 
```
* 배열
```rust
fn main() {
    let a = [1, 2, 3, 4, 5];    // 다른 타입 혼용 불가, 대괄호

    let first = a[0];
    let second = a[1];
}
```

# 3.3. 함수
파이썬과 달리 함수 위치에 제약 받지 않는다

```rust
fn main() {
    print_labeled_measurement(5, 'h');
}

fn print_labeled_measurement(value: i32, unit_label: char) {
    println!("The measurement is: {value}{unit_label}");
}
```
매개변수는 타입을 반드시 명시해야 한다

```rust
fn main() {
    let y = {
        let x = 3;
        x + 1   // 여기에 세미콜론이 없다; 즉 x+1을 반환
    };

    println!("The value of y is: {y}");
}
```
표현식은 계산을 통해 '값(Value)'을 만들어내는 코드(; 없음)

구문은 어떤 행동을 수행하지만, '값(Value)'을 반환하지 않는 코드(; 있음)

```rust
fn main() {
    let x = plus_one(5);

    println!("The value of x is: {x}");
}

fn plus_one(x: i32) -> i32 {    // 반환 값을 -> (타입) 으로 지정
    x + 1
}
```

# 3.4. 주석
//를 사용하여 주석처리(컴파일 시 무시됨)

# 3.5. 제어 흐름문
```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {    // 조건식은 반드시 bool타입일 것
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```
if와 else 갈래 값의 타입이 호환되지 않음(if > int32, else > char같이 혼용 안 됨)

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;  // break으로 무한루프를 탈출, 그리고 값을 반환
        }
    };

    println!("The result is {result}");
}
```

```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {count}");
        let mut remaining = 10;

        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {count}");
}
```
루프 속 루프가 있다면 루프 라벨로 지정하여 제어가능(루프 라벨은 반드시 작은 따옴표로 시작)

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{number}!");

        number -= 1;
    }

    println!("LIFTOFF!!!");
}
```
while로 true인 동안 루프 순회

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!("the value is: {element}");
    }
}
```
for문으로 배열의 요소를 순회
