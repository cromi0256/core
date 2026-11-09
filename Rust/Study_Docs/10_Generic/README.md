# 10.1. 제네릭 데이터 타입
```rust
fn largest_i32(list: &[i32]) -> &i32 {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> &char {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest_i32(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest_char(&char_list);
    println!("The largest char is {}", result);
}
```
제네릭을 쓰지 않으면 코드가 중복되고 유연성이 떨어짐

```rust
fn largest<T>(list: &[T]) -> &T {   // 함수명과 매개변수 목록 사이 <>로 타입 매개변수를 넣는다
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```
아직 위 코드는 모든 타입에 동작하지 못 하므로 컴파일 불가

```rust
struct Point<T> {
    x: T,
    y: T,   // 에러발생: 식별자를 달리하면 해결가능
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```
`T`라는 동일한 제네릭 타입은 컴파일 시 같은 타입으로 예상하므로 에러가 발생

```rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
열겨형에서 제네릭 타입 사용의 대표젹 예시

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {  // 모든 제네릭 타입 T에 적용되는 메서드
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```
`impl` 바로 뒤에 `T`를 선언하여 `Point<T>` 타입에 메서드를 구현한다고 명시했음을 주의

이렇게 하면 러스트는 `Point`의 꺾쇠괄호 내 타입이 구체적인 타입이 아닌 제네릭 타입임을 인지

```rust
impl Point<f32> {   // impl블록 뒤에 제네릭 타입 미적용
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```
특정 구체적 타입인 경우에만 적용되는 impl 블록은 뒤에 <>를 붙이지 않음

# 10.2. 트레이트
트레이트(trait) 는 특정한 타입이 가지고 있으면서 다른 타입과 공유할 수 있는 기능이다

다양한 타입이 동일한 메서드를 호출할 수 있도록 함
```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```
`summarize` 메서드가 제공하는 동작으로 구성된 `Summary` 트레이트 정의

```rust
impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}
```
`trait`를 정의하고 `impl TraitName for TypeName` 문법으로 트레이트 구현

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```
제네릭 함수나 구조체가 특정 동작을 수행하는 타입만 받는 트레이트 바운드 문법

트레이트 바운드는 꺾쇠괄호 안의 제네릭 타입 매개변수 선언에 붙은 콜론(:) 뒤에 위치

```rust
pub fn notify(item: &(impl Summary + Display)) {    // +로 트레이트를 여러개 지정

pub fn notify<T: Summary + Display>(item: &T) { // 제네릭 타입 트레이트 바운드도 사용가능
```
`+` 기호: 여러 트레이트 바운드를 한 번에 지정

```rust
// before
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {

// after
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
```
`where` 절을 사용하여 트레이트 바운드가 여러 개일 경우 함수 시그니처를 깔끔하게 정리

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

// PartialOrd 트레이트와 출력을 가능하게 만드는 Display 트레이트를 모두 구현한 타입인 경우에 대해서만 cmp_display 메서드를 구현
impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```
`impl<T> Type<T>`와 같이 제네릭 타입에 트레이트 바운드를 걸어, 특정 트레이트를 구현한 타입일 때만 메서드를 사용

# 10.3. 라이프타임
라이프타임의 주목적은 댕글링 참조 (dangling reference) 방지

댕글링 참조 : 이미 사라진 데이터를 참조하는 현상
```rust
fn main() {
    let r;

    {
        let x = 5;
        r = &x; // r은 x를 참조
    }   // x는 여기서 버려짐

    println!("r: {}", r);   // r은 사라진 x를 참조함
}
```
스코프 밖으로 벗어난 값을 참조하는 코드로 컴파일 되지 않음

```rust
fn main() {
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {}", r); //   |       |
                          // --+       |
}                         // ----------+
```
러스트 컴파일러는 대여 검사기 (borrow checker) 로 스코프를 비교하여 대여의 유효성을 판단

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}
```
`longest` 함수가 매개변수의 소유권을 얻지 않도록, 문자열 대신 참조자인 문자열 슬라이스를 전달함

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }   // 반환할 참조자가 x인지, y인지 러스트가 알 수 없다
}
```
더 긴 문자열을 출력하는 `longest` 함수는 컴파일 되지 않음

이 함수를 정의하는 시점에서는 함수가 전달받을 구체적인 값을 알 수 없으니, if의 경우가 실행될지 else의 경우가 실행될지 알 수 없다

```rust
&i32        // 참조자
&'a i32     // 명시적인 라이프타임이 있는 참조자
&'a mut i32 // 명시적인 라이프타임이 있는 가변 참조자
```
라이프타임 매개변수의 이름은 어퍼스트로피(')로 시작해야 하며, 보통은 제네릭 타입처럼 매우 짧은 소문자로 정함

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str { // 'a라는 공통의 수명 범위 지정
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
시그니처 내 모든 참조자가 동일한 라이프타임 `'a`를 가져야 함을 나타낸 `longest` 함수 정의

`x` 라이프타임과 `y` 라이프타임 중 더 작은 쪽이 제네릭 라이프타임 `'a`의 구체적인 라이프타임이 됨

```rust
// ImportantExcerpt 구조체는 &'a str 타입의 참조를 담고 있다.
// 즉, 이 구조체는 참조하는 데이터보다 오래 살아남을 수 없다.
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("옛날 옛적에... 어떤 마을에...");
    let first_sentence = novel.split('.').next().expect("문장을 찾을 수 없습니다.");
    
    // 구조체 생성
    let i = ImportantExcerpt { part: first_sentence };
}
```
참조자를 품고 있는 구조체를 만들 때는 라이프타임을 명시할 것

```rust
let s: &'static str = "I have a static lifetime.";
```
정적 라이프타임 `'static`은 프로그램의 전체 생애주기 동안 살아있음을 의미

다만, 일반적으로 해결 가능한 문제에 이 라이프타임은 쓰지 말 것

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
제네릭 타입 매개변수, 트레이트 바운드, 라이프타임을 한 곳에 사용해 보기