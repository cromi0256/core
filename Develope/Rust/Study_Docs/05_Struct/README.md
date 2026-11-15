# 5.1. 구조체 정의 및 인스턴트화
```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```
파이썬의 클래스와 비슷하게 struct로 인스턴스를 생성한다

```rust
fn main() {
    let mut user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com"); // 특정 필드의 값 변경
}
```

```rust
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username,
        email,
        sign_in_count: 1,
    }
}
```
함수로 매개변수를 받아 구조체를 생성할수도 있다

# 5.2. 예제
```rust
fn main() {
    let width1 = 30;
    let height1 = 50;

    println!(
        "The area of the rectangle is {} square pixels.",
        area(width1, height1)
    );
}

fn area(width: u32, height: u32) -> u32 {
    width * height
}
```
이 코드는 문법적으로 문제가 없지만 width와 height가 지나치게 반복되기에 고쳐본다

```rust
fn main() {
    let rect1 = (30, 50);

    println!(
        "The area of the rectangle is {} square pixels.",
        area(rect1)
    );
}

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}
```
튜플로 묶어 인덱스로 넓이를 계산하는 식으로 고친 것

여전히 계산식은 깔끔하지않고 유지보수가 쉽지 않은 방식이다

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```
구조체를 사용하면 코드가 한층 깔끔해진다

다만, 구조체의 소유권을 가져와 버리면 main 함수에서 area 함수 호출 이후에 rect1을 더 사용할 수 없으므로, rectangle 매개변수의 타입을 불변 참조자 타입으로 지정한다

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale),
        height: 50,
    };

    dbg!(&rect1);
}
// 출력화면
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.61s
     Running `target/debug/rectangles`
[src/main.rs:10] 30 * scale = 60
[src/main.rs:14] &rect1 = Rectangle {
    width: 60,
    height: 50,
}
```
디버그를 사용하면 코드가 어떤 일을 하고 있는지 알아볼 때 매우 유용하다

# 5.3. 메서드 문법
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```
파이썬의 클래스 메서드처럼 `impl` 블록으로 구현한다

```rust
impl Rectangle {
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
}
```
impl 블록 내에 구현된 모든 함수를 연관 함수 (associated function) 라고 부르는데, 이는 impl 뒤에 나오는 타입과 모두 연관된 함수이기 때문이다.

동작하는 데 해당 타입의 인스턴스가 필요하지 않다면 self를 첫 매개변수로 갖지 않는 (따라서 메서드가 아닌) 연관 함수를 정의할 수도 있다.(주로 생성자로 활용됨)

연관 함수를 호출할 땐 `let sq = Rectangle::square(3);`처럼 구조체 명에 :: 구문을 붙여서 호출