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