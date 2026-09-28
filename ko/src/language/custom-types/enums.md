# 열거형(`enum`)

C#의 `enum`은 기호 이름을 정수 값에 대응시키는 값 타입입니다.

```c#
enum DayOfWeek
{
    Sunday = 0,
    Monday = 1,
    Tuesday = 2,
    Wednesday = 3,
    Thursday = 4,
    Friday = 5,
    Saturday = 6,
}
```

Rust에서도 거의 같은 구문으로 열거형을 정의합니다.

```rust
enum DayOfWeek
{
    Sunday = 0,
    Monday = 1,
    Tuesday = 2,
    Wednesday = 3,
    Thursday = 4,
    Friday = 5,
    Saturday = 6,
}
```

.NET과 달리 Rust의 `enum` 인스턴스에는 상속받는 기본 동작이
없습니다. 처음에는 `dow == DayOfWeek::Friday` 같은 동등성 비교도
할 수 없습니다. C#의 `enum`에서 흔히 쓰는 기능을 갖추려면
[`#[derive]` 특성][derive]으로 필요한 구현을 자동 생성합니다.

```rust,does_not_compile
#[derive(Debug,     // enables formatting in "{:?}"
         Clone,     // required by Copy
         Copy,      // enables copy-by-value semantics
         Hash,      // enables hash-ability for use in map types
         PartialEq  // enables value equality (==)
)]
enum DayOfWeek
{
    Sunday = 0,
    Monday = 1,
    Tuesday = 2,
    Wednesday = 3,
    Thursday = 4,
    Friday = 5,
    Saturday = 6,
}

fn main() {
    let dow = DayOfWeek::Wednesday;
    println!("Day of week = {dow:?}");

    if dow == DayOfWeek::Friday {
        println!("Yay! It's the weekend!");
    }

    // coerce to integer
    let dow = dow as i32;
    println!("Day of week = {dow:?}");

    let dow = dow as DayOfWeek;
    println!("Day of week = {dow:?}");
}
```

위 예제처럼 열거형을 지정된 정수 값으로 변환할 수 있습니다. 반대
방향의 변환은 C#처럼 바로 할 수 없습니다. C#/.NET에서는 표현하지
않은 정수 값이 `enum` 인스턴스에 들어갈 수 있다는 단점도 있습니다.
Rust에서는 변환을 위한 보조 함수를 직접 만들 수 있습니다.

```rust
impl DayOfWeek {
    fn try_from_i32(n: i32) -> Result<DayOfWeek, i32> {
        use DayOfWeek::*;
        match n {
            0 => Ok(Sunday),
            1 => Ok(Monday),
            2 => Ok(Tuesday),
            3 => Ok(Wednesday),
            4 => Ok(Thursday),
            5 => Ok(Friday),
            6 => Ok(Saturday),
            _ => Err(n)
        }
    }
}
```

`try_from_i32` 함수는 `n`이 올바르면 성공을 나타내는 `Ok`와
`DayOfWeek`를 `Result`로 반환합니다. 그렇지 않으면 실패를
나타내는 `Err`에 원래 `n`을 담아 반환합니다.

```rust
let dow = DayOfWeek::try_from_i32(5);
println!("{dow:?}"); // prints: Ok(Friday)

let dow = DayOfWeek::try_from_i32(50);
println!("{dow:?}"); // prints: Err(50)
```

정수 타입과 열거형 사이의 변환을 직접 구현하지 않아도 되도록 돕는
Rust crate도 있습니다.

Rust의 `enum`으로는 각 변형이 서로 다른 데이터를 담는 _구분된
유니언_ 타입도 설계할 수 있습니다. 다음 예제를 살펴보겠습니다.

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
```

C#에는 같은 형태의 `enum` 선언이 없지만 클래스 레코드로 비슷하게
표현할 수 있습니다.

```c#
var home = new IpAddr.V4(127, 0, 0, 1);
var loopback = new IpAddr.V6("::1");

abstract record IpAddr
{
    public sealed record V4(byte A, byte B, byte C, byte D): IpAddr;
    public sealed record V6(string Address): IpAddr;
}
```

Rust의 정의는 변형의 집합이 고정된 _닫힌 타입_을 만듭니다.
컴파일러는 `IpAddr`에 `IpAddr::V4`와 `IpAddr::V6` 이외의 변형이
없다는 사실을 압니다. 따라서 C#의 `switch` 식과 비슷한 Rust의
`match` 식에서 모든 변형을 다루지 않으면 컴파일 오류를 냅니다.
반면 C# 레코드로 흉내 낸 형태는 간결해 보이더라도 클래스 계층을
만듭니다. `IpAddr`가 _추상 기본 클래스_이므로 컴파일러는 이 타입이
표현할 수 있는 모든 하위 타입을 알지 못합니다.

[derive]: https://doc.rust-lang.org/stable/reference/attributes/derive.html
