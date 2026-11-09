# 11.1. 테스트 작성법
테스트란, 테스트할 코드가 의도대로 기능하는지 검증하는 함수

* 필요한 데이터나 상태 설정
* 테스트할 코드 실행
* 의도한 결과가 나오는지 확인

```rust
// #[cfg(test)]
// mod tests {
    #[test] // 테스트 함수를 명시하는 어노테이션
    fn it_works() {
        let result = 2 + 2;
        assert_eq!(result, 4);  // 두 값이 동일해야 통과하는 테스트
    }
}
```
`cargo test`로 `it_works` 함수를 테스팅함

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.57s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```
테스팅 실행 결과

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }

    #[test]
    fn another() {
        panic!("Make this test fail");  // panic! 매크로로 패닉을 강제로 발생
    }
}
```
결과는 2개 중 `another` 함수는 실패함

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(larger.can_hold(&smaller)); // 조건이 참이어야 통과하는 테스트
    }
}
```
`assert!`매크로는 부울린 타입을 받아 true면 통과시킴

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        assert_eq!(4, add_two(2));  // 2개의 인수를 받음
    }
}
```
`assert_eq!` 두 값이 동일해야 통과하는 테스트

반대로 `assert_ne!`는 두 값이 불일치해야 통과

```rust
    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(
            result.contains("Carol"),
            "Greeting did not contain name, value was `{}`",    // 포맷 문자열
            result
        );
    }
```
테스트 실패 시 출력 내용을 커스터마이즈

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
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic] // 테스트 어노테이션과 함수 사이에 작성
    fn greater_than_100() {
        Guess::new(200);
    }
}
```
`#[should_panic]`: 패닉이 발생해야 통과하는 테스트

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```
Result 타입을 사용해 실패 시 다른 값 반환

# 11.2. 