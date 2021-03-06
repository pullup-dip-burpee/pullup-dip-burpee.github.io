---
layout: post
title:  "Rust Tutorials 1"
date:   2021-02-18 14:19:35 +0900
categories: Rust
---

'rust book'(공식튜토리얼)을 보면서 메모했습니다. 튜토리얼 1-3장에 대응합니다. 

# references
공식튜토리얼: https://doc.rust-lang.org/book  
번역판: https://rinthel.github.io/rust-lang-book-ko/ch01-00-getting-started.html  
자료구조 관련? https://rust-unofficial.github.io/too-many-lists/index.html  
cheatsheet? https://happygrammer.github.io/rust/cheat-seat/


# rustc
rust compiler. `rustc ./main.rs` 등으로 바이너리 생성

---

# Cargo
## cargo commands
cargo 는 rust 의 build system / package manager. 
- `cargo new [project name]`: project name이라는 이름으로 새 프로젝트 시작(`--lib`, `--bin` 등으로 library / binary 선택 가능)
- `cargo build`: 컴파일. `target/debug` 위치에 바이너리 생성.
- `cargo build --release`: 배포 위해 최적화된 형태로 컴파일. `target/release` 위치에 바이너리 생성. 
- `cargo run`: 컴파일 후 실행.
- `cargo check`: 컴파일 가능한지만 확인.

## Cargo.toml
Dependencies 등 설정. 

## Cargo.lock

라이브러리 버전 충돌 등? 문제 발생을 방지합니다. 버전을 고정시킵니다. project를 `cargo new [project name]`으로 만들 때는 없지만, 이후 `cargo build`를 하면 생깁니다.   

```
# This file is automatically @generated by Cargo.
# It is not intended for manual editing.
[[package]]
name = "hello_cargo"
version = "0.1.0"
```

import 할 것들을 넣어주고 다시 `cargo build`를 하면 아래와 같이 변합니다. 이후 build시에는 Cargo가 Cargo.lock을 보고 해당 버전을 사용합니다. 또한 코드에 rand라는 crate를 import 했다면 해당 crate 의 버전을 기록합니다. 


```
# This file is automatically @generated by Cargo.
# It is not intended for manual editing.
[[package]]
name = "bitflags"
version = "1.2.1"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "cf1de2fe8c75bc145a2f577add951f8134889b4795d47466a54a5c846d691693"

...(중략)

[[package]]
name = "rand"
version = "0.5.6"
source = "registry+https://github.com/rust-lang/crates.io-index"
checksum = "c618c47cd3ebd209790115ab837de41425723956ad3ce2e6a7f09890947cacb9"
dependencies = [
 "cloudabi",
 "fuchsia-cprng",
 "libc",
 "rand_core 0.3.1",
 "winapi",
]

...(이하생략)
```

# macro
C/C++에서의 macro와는 좀 달라보이는데? `println!("Hello world")` 같이 `!`로 끝납니다. 
- 함수와의 차이점: 

# prelude
`std::prelude`란 rust가 자동으로 모든 rust 프로그램에 import하는 것들, implicit하게 import 되는 것들입니다. std::prelude 말고 다른 prelude도 있는데 `std::io::prelude` 처럼 import할 수 있나봅니다.

# 튜토리얼 2장에서 나온 함수들
- `{String Object}.trim()`: String에서 `\n`을 지웁니다. 
- `{String Object}.parse()`

# 변수선언
- 변수선언: `let`, `let mut` 등의 키워드로 immutable/mutable 변수등을 선언합니다. `let`으로 선언했으면 이후 다시 선언하지 않는 이상 값을 다시 할당할 수 없음. `let foo = 5`, `let mut foo = String::new()` 처럼 사용합니다.

- 상수선언: `const`. `mut` 불가능, 어느 scope에서든 사용가능, runtime에 할당되게 할 수 없음. 
- shadowing: 가령 `let mut guess = ...` 하는 식으로 앞에 선언했어도 뒤에 `let guess: u32 = ...` 하는 식으로 덮어쓸 수 있습니다. 

# Data types  
- integer: unsigned(`u8`, `u16`, `u32`, `u64`.. ), signed(`i8`, `i16`, `i32`, `i64`..), default: i32. signed: two's complement. 
- `isize`, `usize`: machine architecture에 따라 32bit / 64bit. 
- 정수 표현 시 `_`를 visual separator로 사용 가능! C/C++은 찾아보니 digit separator가 `'` 입니다. 
- integer overflow시, `--debug`로 컴파일 했으면 `panic!`을 띄우고, `--release`로 컴파일 했으면 two's complement 따름
- `char`는 4 byte - unicord 표현. 
- `arrray`는 아래와 같이 선언.

    ```
    fn main() {
    let a: [i32; 5] = [1, 2, 3, 4, 5];
    }
    ```
- index out of bounds를 컴파일 타임에 잡아줍니다. panic 낼 수 있는 코드를 잡습니다. 가령 위의 array에서 `let elem = a[10];` 이라고 했으면, 런타임이 아닌 컴파일 타임에 out of bound를 잡습니다. 

-----

# Functions

```rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();

    println!("The value of x is: {}", x);
}
```

위 코드와 같이 리턴 값의 데이터 타입을 `-> i32`같은 식으로 작성합니다. `fn five()`에는 세미콜론 없이 5 하나만 있는데, 이것도 `return 5`와 동일하게 취급됩니다. 세미콜론을 쓰면 에러를 냅니다. 

