# 8.1. 벡터
```rust
    let v: Vec<i32> = Vec::new();
```
새 벡터 생성

```rust
    let mut v = Vec::new();

    v.push(5);
    v.push(6);
    v.push(7);
    v.push(8);
```
벡터 업데이트

```rust
    let v = vec![1, 2, 3, 4, 5];

    let third: &i32 = &v[2];    // 인덱스 사용
    println!("The third element is {third}");

    let third: Option<&i32> = v.get(2); // get 사용
    match third {
        Some(third) => println!("The third element is {third}"),
        None => println!("There is no third element."), // 유효하지 않는 인덱스는 None 처리
    }
```
벡터 요소 읽기

```rust
    let mut v = vec![1, 2, 3, 4, 5];

    let first = &v[0];

    v.push(6);

    println!("The first element is: {first}");
```
불변 참조자(&)가 존재하는 동안에는 값을 변경할 수 있는 가변 참조자(&mut)를 만들거나 값을 수정할 수 없다

```rust
    let mut v = vec![100, 32, 57];
    for i in &mut v {
        *i += 50;   // i는 포인터, *는 역참조 > vec![150, 82, 107]
    }
```
`for` 루프로 벡터의 요소들에 대해 반복

```rust
    enum SpreadsheetCell {
        Int(i32),
        Float(f64),
        Text(String),
    }

    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Text(String::from("blue")),
        SpreadsheetCell::Float(10.12),
    ];
```
기본적으로 동일한 타입만 벡터로 묶으나, `enum`열거형을 정의하면 혼용가능

# 8.2. 문자열
```rust
    let mut s = String::new();
```
비어있는 문자열 생성

```rust
    let data = "initial contents";

    let s = data.to_string();

    // 이 메서드는 리터럴에서도 바로 작동합니다:
    let s = "initial contents".to_string();
```
`to_string` 메서드를 사용한 생성

```rust
    let s = String::from("initial contents");
```
`String::from` 함수를 사용한 생성

```rust
    let mut s = String::from("foo");
    s.push_str("bar");
```
`push_str` 메서드로 문자열 슬라이스 추가

```rust
    let mut s = String::from("lo");
    s.push('l');
```
`push` 메서드는 1개의 글자를 받아 추가

```rust
    let s1 = String::from("Hello, ");
    let s2 = String::from("world!");
    let s3 = s1 + &s2; // s1은 여기로 이동되어 더 이상 사용할 수 없음을 주의하세요
// fn add(self, s: &str) -> String {
```
내부 구조에 따라 합쳐짐

```rust
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    let s = format!("{s1}-{s2}-{s3}");
```
`format!`매크로로 문자열 접합

```rust
let hello = "Здравствуйте";
let answer = &hello[0]; // 오류발생
```
러스트는 UTF-8 인코딩을 사용함에 따라 영어, 숫자가 아니면 인덱싱이 직관적이지 않음

```rust
for c in "Зд".chars() {
    println!("{c}");
}

// 실행
// З
// д

for b in "Зд".bytes() {
    println!("{b}");
}

// 실행
// 208
// 151
// 208
// 180
```

# 8.3. 해시맵
`HashMap<K, V>`은 Key와 Value 해시함수로 이루어짐
```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    for (key, value) in &scores {
        println!("{key}: {value}");
    }
```
새 해시맵 생성과 값접근

```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Blue"), 25);

    println!("{:?}", scores);   // 출력: 25
```
값 덮어쓰기

```rust
    use std::collections::HashMap;

    let text = "hello world wonderful world";

    let mut map = HashMap::new();

    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }

    println!("{:?}", map);
// 출력
// {"world": 2, "hello": 1, "wonderful": 1}
```
예전 값에 기초하여 값을 업데이트하기