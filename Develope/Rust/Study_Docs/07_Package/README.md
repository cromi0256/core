# 7.1. 패키지와 크레이트
크레이트 (crate) 는 러스트가 컴파일 한 차례에 고려하는 가장 작은 코드 단위

크레이트는 바이너리일 수도 있고, 라이브러리일 수도 있음

* 바이너리 크레이트 : 커맨드 라인 프로그램이나 서버처럼 실행 가능한 실행파일로 컴파일할 수 있는 프로그램
* 라이브러리 크레이트 : main 함수를 가지고 있지 않고 실행파일 형태로 컴파일되지 않음

패키지 (package) 는 일련의 기능을 제공하는 하나 이상의 크레이트로 구성된 번들

# 7.2. 모듈
모듈 치트 시트:
* 크레이트 루트부터 시작: 크레이트를 컴파일할 때 컴파일러는 먼저 크레이트 루트 파일을 봅니다 (보통은 라이브러리 크레이트의 경우 src/lib.rs 혹은 바이너리 크레이트의 경우 src/main.rs입니다).
* 모듈 선언: 크레이트 루트 파일에는 새로운 모듈을 선언할 수 있습니다; mod garden;이라는 코드로 ‘garden’ 모듈을 선언할 수 있습니다. 컴파일러는 아래의 장소에서 이 모듈의 코드가 있는지 살펴볼 것입니다:
    * mod garden 뒤에 세미콜론 대신 중괄호를 써서 안쪽에 코드를 적은 인라인
    * src/garden.rs 파일 안
    * src/garden/mod.rs 파일 안
* 서브모듈 선언: 크레이트 루트가 아닌 다른 파일에서는 서브모듈 (submodule) 을 선언할 수 있습니다. 예를 들면 src/garden.rs 안에 mod vegetables;를 선언할 수도 있습니다. 컴파일러는 부모 모듈 이름의 디렉터리 안쪽에 위치한 아래의 장소들에서 이 서브모듈의 코드가 있는지 살펴볼 것입니다:
    * mod vegetables 뒤에 세미콜론 대신 중괄호를 써서 안쪽에 코드를 적은 인라인
    * src/garden/vegetables.rs 파일 안
    * src/garden/vegetables/mod.rs 파일 안
* 모듈 내 코드로의 경로: 일단 모듈이 크레이트의 일부로서 구성되면, 공개 규칙이 허용하는 한도 내에서라면 해당 코드의 경로를 사용하여 동일한 크레이트의 어디에서든 이 모듈의 코드를 참조할 수 있게 됩니다. 예를 들면, garden vegetables 모듈 안에 있는 Asparagus 타입은 crate::garden::vegetables::Asparagus로 찾아 쓸 수 있습니다.
* 비공개 vs 공개: 모듈 내의 코드는 기본적으로 부모 모듈에게 비공개 (private) 입니다. 모듈을 공개 (public) 로 만들려면, mod 대신 pub mod를 써서 선언하세요. 공개 모듈의 아이템들을 공개하려면 마찬가지로 그 선언 앞에 pub을 붙이세요.
* use 키워드: 어떤 스코프 내에서 use 키워드는 긴 경로의 반복을 줄이기 위한 어떤 아이템으로의 단축경로를 만들어 줍니다. crate::garden::vegetables::Asparagus를 참조할 수 있는 모든 스코프에서 use crate::garden::vegetables::Asparagus;로 단축경로를 만들 수 있으며, 그 이후부터는 스코프에서 이 타입을 사용하려면 Asparagus만 작성해주면 됩니다.

```rust
// 디렉토리
// backyard
// ├── Cargo.lock
// ├── Cargo.toml
// └── src
//     ├── garden
//     │   └── vegetables.rs
//     ├── garden.rs
//     └── main.rs

use crate::garden::vegetables::Asparagus;

pub mod garden;

fn main() {
    let plant = Asparagus {};
    println!("I'm growing {:?}!", plant);
}
```
`pub mod garden;`을 사용하려면 하위 파일에 `pub`을 선언해야함

모듈 내의 코드는 기본적으로 비공개

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}

// 모듈 트리
// crate
//  └── front_of_house
//      ├── hosting
//      │   ├── add_to_waitlist
//      │   └── seat_at_table
//      └── serving
//          ├── take_order
//          ├── serve_order
//          └── take_payment
```

# 7.3. 경로를 사용한 참조
```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 절대 경로
    crate::front_of_house::hosting::add_to_waitlist();

    // 상대 경로
    front_of_house::hosting::add_to_waitlist();
}
```
위의 모듈은 모듈이 비공개 되어 정상적으로 컴파일 되지 않음

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 절대 경로
    crate::front_of_house::hosting::add_to_waitlist();

    // 상대 경로
    front_of_house::hosting::add_to_waitlist();
}
```
위와 같이 하위 모듈 모두 `pub`을 붙여야 실행 가능

```rust
fn deliver_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::deliver_order();
    }

    fn cook_order() {}
}
```
상대 경로의 경우 `super`을 사용하여 상위 모듈로 이동

```rust
mod back_of_house {
    pub enum Appetizer {
        Soup,
        Salad,
    }
}

pub fn eat_at_restaurant() {
    let order1 = back_of_house::Appetizer::Soup;
    let order2 = back_of_house::Appetizer::Salad;
}
```
열거형의 경우 배리언트는 기본적으로 공개형

# 7.4. use 키워드
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```
단축키처럼 짧게 모듈을 가져옴

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

// 문제
mod customer {
    pub fn eat_at_restaurant() {
        hosting::add_to_waitlist(); // 오류가 발생하는 곳
    }
}

// 해결방법 1
mod customer {
    // customer 모듈 내부로 use 문 이동
    use crate::front_of_house::hosting;

    pub fn eat_at_restaurant() {
        hosting::add_to_waitlist(); // 정상 컴파일됨
    }
}

// 해결방법 2
mod customer {
    pub fn eat_at_restaurant() {
        // 부모 모듈로 거슬러 올라가서 hosting 참조
        super::hosting::add_to_waitlist(); 
        
    }
}
```
스코프 내부에서만 유효하므로 코드 작성에 유의

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --생략--
}

fn function2() -> IoResult<()> {
    // --생략--
}
```
`as`를 사용하여 새 이름으로 불러오기

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist(); // restaurant::front_of_house::hosting::add_to_waitlist()처럼 길게 작성 하지 않아도 됨
}
```
`pub use`를 사용하면 직관적으로 활용가능

# 7.5. 파일로 모듈 분리
```rust
// 파일명: src/lib.rs
mod front_of_house;

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
// src/front_of_house.rs 파일을 만들기전까진 컴파일 되지 않음

// 파일명: src/front_of_house.rs
pub mod hosting {
    pub fn add_to_waitlist() {}
}
```
모듈의 부모-자식 관계가 실제 컴퓨터의 폴더-파일 구조와 1:1로 정확히 일치해야 컴파일이 된다