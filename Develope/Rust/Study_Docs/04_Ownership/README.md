소유권은 러스트가 가비지 컬렉터 없이 메모리 안정성을 보장한다

# 4.1. 소유권
러스트는 스택, 힙으로 저장하여 메모리를 할당하고 해제한다

소유권 규칙:
* 러스트에서, 각각의 값은 소유자 (owner) 가 정해져 있습니다.
* 한 값의 소유자는 동시에 여럿 존재할 수 없습니다.
* 소유자가 스코프 밖으로 벗어날 때, 값은 버려집니다 (dropped).

```rust
    {                      // s는 아직 선언되지 않아서 여기서는 유효하지 않습니다
        let s = "hello";   // 이 지점부터 s가 유효합니다

        // s로 어떤 작업을 합니다
    }                      // 이 스코프가 종료되었고, s가 더 이상 유효하지 않습니다
``
스코프

```rust
    let mut s = String::from("hello");  // 스트링 타입 생성

    s.push_str(", world!"); // push_str()이 문자열에 리터럴을 추가합니다

    println!("{}", s); // 이 줄이 `hello, world!`를 출력합니다
```
스트링 타입

```rust
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {}, s2 = {}", s1, s2);
```
가변적 길이의 타입은 클론을 사용하고, 스칼라 타입은 복사한다

```rust
fn main() {
    let s = String::from("hello");  // s가 스코프 안으로 들어옵니다

    takes_ownership(s);             // s의 값이 함수로 이동됩니다...
                                    // ... 따라서 여기서는 더 이상 유효하지 않습니다

    let x = 5;                      // x가 스코프 안으로 들어옵니다

    makes_copy(x);                  // x가 함수로 이동될 것입니다만,
                                    // i32는 Copy이므로 앞으로 계속 x를
                                    // 사용해도 좋습니다

} // 여기서 x가 스코프 밖으로 벗어나고 s도 그렇게 됩니다. 그러나 s의 값이 이동되었으므로
  // 별다른 일이 발생하지 않습니다.

fn takes_ownership(some_string: String) { // some_string이 스코프 안으로 들어옵니다
    println!("{}", some_string);
} // 여기서 some_string이 스코프 밖으로 벗어나고 `drop`이 호출됩니다.
  // 메모리가 해제됩니다.

fn makes_copy(some_integer: i32) { // some_integer가 스코프 안으로 들어옵니다
    println!("{}", some_integer);
} // 여기서 some_integer가 스코프 밖으로 벗어납니다. 별다른 일이 발생하지 않습니다.
```
어떤 값을 다른 변수에 대입하면 값이 이동하고, 힙에 데이터를 갖는 변수가 스코프를 벗어나면, 사전에 해당 데이터가 이동하여 소유권이 다른 변수에 이동되지 않은 이상 drop 에 의해 데이터가 제거됩니다.

# 4.2. 참조와 대여
```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize { // s는 String의 참조자입니다
    s.len()
} // 여기서 s가 스코프 밖으로 벗어납니다. 하지만 참조하는 것을 소유하고 있진 않으므로,
  // 버려지지는 않습니다.
```
![image](https://doc.rust-kr.org/img/trpl04-05.svg)

참조자 (reference) 는 해당 주소에 저장된 데이터에 접근할 수 있도록 해주는 주솟값에 해당하는, 포인터와 같다

```rust
    let mut s = String::from("hello");

    let r1 = &s; // 문제없음
    let r2 = &s; // 문제없음
    let r3 = &mut s; // 큰 문제

    println!("{}, {}, and {}", r1, r2, r3);
```
가변 참조자의 경우 데이터 경합에 주의

데이터 경합이란 다음 세 가지 상황이 겹칠 때 일어나는 특정한 경합 조건 (race condition) 입니다:
* 둘 이상의 포인터가 동시에 같은 데이터에 접근
* 포인터 중 하나 이상이 데이터에 쓰기 작업을 시행
* 데이터 접근 동기화 메커니즘이 없음

```rust
fn dangle() -> &String { // dangle은 String의 참조자를 반환합니다

    let s = String::from("hello"); // s는 새로운 String입니다

    &s // String s의 참조자를 반환합니다
} // 여기서 s는 스코프 밖으로 벗어나고 버려집니다. 해당 메모리는 해제됩니다.
  // 위험합니다!
```
참조자 규칙:
* 여러분은 단 하나의 가변 참조자만 갖거나, 여러 개의 불변 참조자를 가질 수 있습니다.
* 참조자는 항상 유효해야 합니다.

# 4.3. 슬라이스
슬라이스 (slice) 는 컬렉션 (collection) 을 통째로 참조하는 것이 아닌, 컬렉션의 연속된 일련의 요소를 참조하도록 해줍니다. 슬라이스는 참조자의 일종으로서 소유권을 갖지 않습니다.

```rust
    let s = String::from("hello world");

    let hello = &s[0..5];   // 슬라이스 표현
    let world = &s[6..11];
```
![image](https://doc.rust-kr.org/img/trpl04-06.svg)

```rust
let s = "Hello, world!";    // s는 &str타입의 불변 참조자
```
String은 소유권이 있는 문자열; &str은 빌려오는 문자열 슬라이스

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}

fn main() {
    let my_string = String::from("hello world");

    // `first_word`는 `String`의 일부 혹은 전체 슬라이스에 대해 작동합니다
    let word = first_word(&my_string[0..6]);
    let word = first_word(&my_string[..]);
    // 또한 `first_word`는 `String`의 전체 슬라이스와 동일한 `String`의
    // 참조자에 대해서도 작동합니다
    let word = first_word(&my_string);

    let my_string_literal = "hello world";

    // `first_word`는 문자열 리터럴의 일부 혹은 전체 슬라이스에 대해 작동합니다
    let word = first_word(&my_string_literal[0..6]);
    let word = first_word(&my_string_literal[..]);

    // 문자열 리터럴은 *곧* 문자열 슬라이스이므로,
    // 아래의 코드도 슬라이스 문법 없이 작동합니다!
    let word = first_word(my_string_literal);
}
```