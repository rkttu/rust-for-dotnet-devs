# 문자열

Rust에는 `String`과 `&str`이라는 두 가지 주요 문자열 타입이 있습니다.
`String`은 힙에 할당되며, `&str`은 문자열의 슬라이스입니다.

다음 표는 .NET의 대응 타입을 보여 줍니다.

| Rust               | .NET                 | 비고      |
| ------------------ | -------------------- | --------- |
| `&mut str`         | `Span<char>`         |           |
| `&str`             | `ReadOnlySpan<char>` |           |
| `Box<str>`         | `String`             | 주 1 참조 |
| `String`           | `String`             |           |
| 가변 `String`      | `StringBuilder`      | 주 1 참조 |

Rust와 .NET에서 문자열을 다루는 방식에는 차이가 있지만 위 대응 관계는
출발점으로 활용할 수 있습니다. Rust 문자열은 UTF-8로 인코딩하고 .NET
문자열은 UTF-16으로 인코딩합니다. .NET 문자열은 불변입니다. Rust의
`String`은 `let mut s = String::from("hello");`처럼 선언하면
변경할 수 있습니다.

소유권 개념도 문자열 사용 방식에 영향을 줍니다. `String`의 소유권에
관해서는 [Rust Book][ownership-string-type-example]에서 확인할 수
있습니다.

[ownership-string-type-example]: https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#the-string-type

주석:

1. Rust의 `Box<str>`은 .NET의 `String`에 대응합니다. Rust에서
   `Box<str>`은 포인터와 크기를 저장하지만 `String`은 포인터, 크기,
   용량을 저장합니다. 따라서 `String`의 크기를 늘릴 수 있습니다.
   Rust의 `String`을 가변으로 선언했을 때는 .NET의 `StringBuilder`와
   비슷한 면이 있습니다.

먼저 C# 예제입니다.

```csharp
ReadOnlySpan<char> span = "Hello, World!";
string str = "Hello, World!";
StringBuilder sb = new StringBuilder("Hello, World!");
```

Rust에서는 다음과 같이 작성합니다.

```rust
let span: &str = "Hello, World!";
let str: Box<str> = Box::from("Hello World!");
let mut sb = String::from("Hello World!");
```

## 문자열 리터럴

.NET의 문자열 리터럴은 불변 `String`이며 힙에 할당됩니다. Rust의
문자열 리터럴은 `&'static str` 타입입니다. 불변이고 전역 수명을
가지며, 힙에 할당되지 않고 컴파일된 바이너리에 포함됩니다.

먼저 C# 예제입니다.

```csharp
string str = "Hello, World!";
```

Rust 예제는 다음과 같습니다.

```rust
let str: &'static str = "Hello, World!";
```

C#의 축어 문자열 리터럴은 Rust의 원시 문자열 리터럴에 대응합니다.
먼저 C# 코드입니다.

```csharp
string str = @"Hello, \World/!";
```

Rust 코드는 다음과 같습니다.

```rust
let str = r#"Hello, \World/!"#;
```

C#의 UTF-8 문자열 리터럴은 Rust의 바이트 문자열 리터럴에 대응합니다.
먼저 C# 코드입니다.

```csharp
ReadOnlySpan<byte> str = "hello"u8;
```

Rust 코드는 다음과 같습니다.

```rust
let str = b"hello";
```

## 문자열 보간

C#에는 문자열 리터럴 안에 식을 넣을 수 있는 문자열 보간 기능이
있습니다. 다음 예제에서 사용법을 확인할 수 있습니다.

```csharp
string name = "John";
int age = 42;
string str = $"Person {{ Name: {name}, Age: {age} }}";
```

Rust에는 언어 차원의 문자열 보간 기능이 없습니다. 대신 `format!`
매크로로 문자열을 포맷합니다. 다음 예제를 살펴보겠습니다.

```rust
let name = "John";
let age = 42;
let str = format!("Person {{ name: {name}, age: {age} }}");
```

`format!`은 문자열 안에 변수 이름을 바로 넣는 방식만 지원합니다.
더 복잡한 식은 `format!("1 + 1 = {}", 1 + 1)`처럼 별도 인수로
전달합니다.

C#의 사용자 정의 클래스와 구조체는 모두 `object`에서 `ToString()`을
상속하므로 문자열 보간에 사용할 수 있습니다.

```csharp
class Person
{
    public string Name { get; set; }
    public int Age { get; set; }

    public override string ToString() =>
        $"Person {{ Name: {Name}, Age: {Age} }}";
}

var person = new Person { Name = "John", Age = 42 };
Console.Writeline(person);
```

Rust에서는 모든 타입이 기본 포맷 구현을 물려받지 않습니다. 문자열로
표현할 타입마다 `std::fmt::Display` 트레이트를 구현해야 합니다.

```rust
use std::fmt::*;

struct Person {
    name: String,
    age: i32,
}

impl Display for Person {
    fn fmt(&self, f: &mut Formatter<'_>) -> Result {
        write!(f, "Person {{ name: {}, age: {} }}", self.name, self.age)
    }
}

let person = Person {
    name: "John".to_owned(),
    age: 42,
};

println!("{person}");
```

포맷 문자열 없이 `Display`로 값을 문자열로 바꿀 때는
`std::string::ToString` 트레이트를 사용할 수 있습니다. 이 트레이트의
`to_string()` 메서드는 .NET의 `ToString()`과 비슷하며 `Display`를
구현하면 자동으로 제공됩니다. 다음 두 표현은 같은 역할을 합니다.

```rust
// Display를 구현했으므로 to_string()을 자동으로 사용할 수 있습니다.
let s = person.to_string();
// s == "Person { name: John, age: 42 }"
```

`std::fmt::Debug` 트레이트를 사용하는 방법도 있습니다. 표준 타입은
모두 `Debug`를 구현하며 타입의 내부 표현을 출력할 때 사용할 수
있습니다. 다음 예제는 `derive` 특성으로 `Person` 구조체에 `Debug`를
자동 구현하고 내부 표현을 출력합니다.

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: i32,
}

let person = Person {
    name: "John".to_owned(),
    age: 42,
};

println!("{person:?}");
```

> `:?` 포맷 지정자는 `Debug` 트레이트로 구조체를 출력합니다. 이를
> 생략하면 `Display` 트레이트를 사용합니다.
>
> `:#?` 지정자를 사용하면 디버그 출력을 보기 좋게 정렬할 수 있습니다.

관련 자료:

- [Rust By Example의 Debug 설명](https://doc.rust-lang.org/stable/rust-by-example/hello/print/print_debug.html?highlight=derive#debug)
