# 멤버

## 생성자

Rust에는 생성자라는 구문이 없습니다. 대신 타입 인스턴스를 반환하는
팩터리 함수를 작성합니다. 팩터리 함수는 독립 함수일 수도 있고 타입의
_연관 함수_일 수도 있습니다. C#으로 비유하면 연관 함수는 타입의 정적
메서드와 비슷합니다. 구조체에 팩터리 함수가 하나만 있다면 관례상
`new`라고 이름을 붙입니다.

```rust
struct Rectangle {
    x1: i32, y1: i32,
    x2: i32, y2: i32,
}

impl Rectangle {
    pub fn new(x1: i32, y1: i32, x2: i32, y2: i32) -> Self {
        Self { x1, y1, x2, y2 }
    }
}
```

Rust 함수는 연관 함수 여부와 관계없이 오버로딩을 지원하지 않습니다.
따라서 여러 팩터리 함수에는 서로 다른 이름을 붙입니다. `String`에서
제공하는 생성 함수의 예는 다음과 같습니다.

- `String::new`: 빈 문자열 생성
- `String::with_capacity`: 초기 버퍼 용량을 지정한 문자열 생성
- `String::from_utf8`: UTF-8 텍스트 바이트에서 문자열 생성
- `String::from_utf16`: UTF-16 텍스트 바이트에서 문자열 생성

Rust의 `enum`에서는 변형이 생성자 역할을 합니다. 자세한 내용은
[열거형 절][enums]에서 확인할 수 있습니다.

관련 자료:

- [생성자는 정적 고유 메서드(C-CTOR)][rs-api-C-CTOR]

[enums]: enums.md
[rs-api-C-CTOR]: https://rust-lang.github.io/api-guidelines/predictability.html?highlight=new#constructors-are-static-inherent-methods-c-ctor

## 정적 메서드와 인스턴스 메서드

C#처럼 Rust의 `enum`과 `struct`에도 정적 메서드와 인스턴스
메서드에 대응하는 함수를 둘 수 있습니다. Rust에서 _메서드_는 항상
인스턴스에 속하며 첫 매개변수의 이름이 `self`입니다. `self`는
메서드가 속한 타입을 뜻하므로 별도의 타입 표기를 하지 않습니다.
정적 메서드에 해당하는 함수는 _연관 함수_라고 부릅니다. 다음
예제에서 `new`는 연관 함수이고 `length`, `width`, `area`는
메서드입니다.

```rust
struct Rectangle {
    x1: i32, y1: i32,
    x2: i32, y2: i32,
}

impl Rectangle {
    pub fn new(x1: i32, y1: i32, x2: i32, y2: i32) -> Self {
        Self { x1, y1, x2, y2 }
    }

    pub fn length(&self) -> i32 {
        self.y2 - self.y1
    }

    pub fn width(&self)  -> i32 {
        self.x2 - self.x1
    }

    pub fn area(&self)  -> i32 {
        self.length() * self.width()
    }
}
```

## 상수

C#과 마찬가지로 Rust의 타입에도 상수를 둘 수 있습니다. Rust에서는
타입 인스턴스 자체를 상수로 정의할 수도 있습니다.

```rust
struct Point {
    x: i32,
    y: i32,
}

impl Point {
    const ZERO: Point = Point { x: 0, y: 0 };
}
```

C#에서 같은 역할을 하려면 정적 읽기 전용 필드가 필요합니다.

```c#
readonly record struct Point(int X, int Y)
{
    public static readonly Point Zero = new(0, 0);
}
```

## 이벤트

C#의 `event` 키워드처럼 타입 멤버가 이벤트를 알리고 발생시키는
기능은 Rust 언어에 내장되어 있지 않습니다.

## 속성

C#에서는 대체로 타입의 필드를 비공개로 두고 `get`, `set` 접근자를
가진 속성으로 읽기와 쓰기를 캡슐화합니다. 접근자에는 값을 설정할
때 검증하거나 읽을 때 계산하는 로직을 넣을 수 있습니다. Rust에는
메서드가 있으며 [getter는 필드와 같은 이름을 쓰고 setter에는
`set_` 접두사를 붙이는 관례][get-set-name.rs]가 있습니다. Rust에서는
메서드와 필드가 같은 이름을 가질 수 있습니다.

[get-set-name.rs]: https://github.com/rust-lang/rfcs/blob/master/text/0344-conventions-galore.md#gettersetter-apis

다음 예제는 Rust 타입에서 속성과 비슷한 접근자 메서드를 작성하는
방식을 보여 줍니다.

```rust
struct Rectangle {
    x1: i32, y1: i32,
    x2: i32, y2: i32,
}

impl Rectangle {
    pub fn new(x1: i32, y1: i32, x2: i32, y2: i32) -> Self {
        Self { x1, y1, x2, y2 }
    }

    // 속성 getter와 비슷하며 각각 필드와 이름이 같습니다.

    pub fn x1(&self) -> i32 { self.x1 }
    pub fn y1(&self) -> i32 { self.y1 }
    pub fn x2(&self) -> i32 { self.x2 }
    pub fn y2(&self) -> i32 { self.y2 }

    // 속성 setter와 비슷합니다.

    pub fn set_x1(&mut self, val: i32) { self.x1 = val }
    pub fn set_y1(&mut self, val: i32) { self.y1 = val }
    pub fn set_x2(&mut self, val: i32) { self.x2 = val }
    pub fn set_y2(&mut self, val: i32) { self.y2 = val }

    // 계산 속성과 비슷합니다.

    pub fn length(&self) -> i32 {
        self.y2 - self.y1
    }

    pub fn width(&self)  -> i32 {
        self.x2 - self.x1
    }

    pub fn area(&self)  -> i32 {
        self.length() * self.width()
    }
}
```

> C#에서는 필드마다 속성을 공개하고 필드는 비공개로 유지하는 방식이
> 일반적입니다. Rust에서는 가능하면 필드를 직접 공개하는 경우가 더
> 많습니다. 접근자 메서드를 위한 전용 구문이 없고 빌림 검사기와
> 관련한 복잡성이 생길 수 있기 때문입니다.

## 확장 메서드

C#의 확장 메서드는 기존 타입의 정의를 수정하지 않고 정적으로
바인딩되는 새 메서드를 붙입니다. 다음 C# 예제에서는 `StringBuilder`
클래스에 `Wrap` 메서드를 확장 방식으로 추가합니다.

```csharp
using System;
using System.Text;
using Extensions; // (1)

var sb = new StringBuilder("Hello, World!");
sb.Wrap(">>> ", " <<<"); // (2)
Console.WriteLine(sb.ToString()); // 출력: >>> Hello, World! <<<

namespace Extensions
{
    static class StringBuilderExtensions
    {
        public static void Wrap(this StringBuilder sb,
                                string left, string right) =>
            sb.Insert(0, left).Append(right);
    }
}
```

확장 메서드를 사용하려면 (1) 해당 메서드가 들어 있는 타입의
네임스페이스를 가져와야 (2) 메서드를 호출할 수 있습니다. Rust에서는
_확장 트레이트_로 비슷한 기능을 구현합니다. 다음 Rust 예제는
`String`에 `wrap` 메서드를 추가합니다.

```rust
#![allow(dead_code)]

mod exts {
    pub trait StrWrapExt {
        fn wrap(&mut self, left: &str, right: &str);
    }

    impl StrWrapExt for String {
        fn wrap(&mut self, left: &str, right: &str) {
            self.insert_str(0, left);
            self.push_str(right);
        }
    }
}

fn main() {
    use exts::StrWrapExt as _; // (1)

    let mut s = String::from("Hello, World!");
    s.wrap(">>> ", " <<<"); // (2)
    println!("{s}"); // 출력: >>> Hello, World! <<<
}
```

C#과 마찬가지로 확장 트레이트의 메서드를 사용하려면 (1) 트레이트를
가져와야 (2) 메서드를 호출할 수 있습니다. 가져올 때 트레이트 이름
`StrWrapExt`를 `_`로 버려도 `String`의 `wrap` 메서드는 계속 사용할
수 있습니다.

## 가시성과 접근 한정자

C#에는 다음과 같은 접근 한정자가 있습니다.

- `private`
- `protected`
- `internal`
- `protected internal`(패밀리)
- `public`

Rust 프로그램은 모듈로 이루어진 트리로 구성됩니다. 모듈에는 타입,
트레이트, 열거형, 상수, 함수 같은 [_항목_][items]을 정의합니다.
거의 모든 항목은 기본적으로 비공개입니다. 예외적으로 공개 트레이트의
연관 항목은 기본적으로 공개됩니다. C# 인터페이스에서 `public`을
명시하지 않은 멤버가 기본적으로 공개되는 것과 비슷합니다. Rust에서는
`pub` 한정자로 모듈 트리에 대한 가시성을 변경합니다. `pub`의 변형을
사용하면 공개 범위를 더 좁게 지정할 수 있습니다.

- `pub(self)`
- `pub(super)`
- `pub(crate)`
- `pub(in PATH)`

자세한 내용은 The Rust Reference의 [가시성과 비공개 범위][privis] 절에서
확인할 수 있습니다.

[privis]: https://doc.rust-lang.org/reference/visibility-and-privacy.html
[items]: https://doc.rust-lang.org/reference/items.html

다음 표는 C#과 Rust 한정자의 대략적인 대응 관계를 보여 줍니다.

| C#                            | Rust         | 비고      |
| ----------------------------- | ------------ | --------- |
| `private`                     | 기본값       | 주 1 참조 |
| `protected`                   | 해당 없음    | 주 2 참조 |
| `internal`                    | `pub(crate)` |           |
| `protected internal`(패밀리)  | 해당 없음    | 주 2 참조 |
| `public`                      | `pub`        |           |

1. Rust에서는 비공개 가시성을 나타내는 별도 키워드가 없습니다. 기본값이
   비공개입니다.
2. Rust에는 클래스 기반 타입 계층이 없으므로 `protected`에 대응하는
   한정자가 없습니다.

## 가변성

C#에서 타입이 변경 가능한지와 파괴적 또는 비파괴적 변경을 지원하는지는
개발자가 설계합니다. C#의 위치 기반 레코드 선언(`record class` 또는
`readonly record struct`)은 불변 설계를 지원합니다. Rust에서는
다음 예제처럼 `self` 매개변수의 타입으로 메서드의 가변성을 표현합니다.

```rust
struct Point { x: i32, y: i32 }

impl Point {
    pub fn new(x: i32, y: i32) -> Self {
        Self { x, y }
    }

    // self는 가변이 아닙니다.

    pub fn x(&self) -> i32 { self.x }
    pub fn y(&self) -> i32 { self.y }

    // self는 가변입니다.

    pub fn set_x(&mut self, val: i32) { self.x = val }
    pub fn set_y(&mut self, val: i32) { self.y = val }
}
```

C#에서는 `with`로 비파괴적 변경을 할 수 있습니다.

```c#
var pt = new Point(123, 456);
pt = pt with { X = 789 };
Console.WriteLine(pt.ToString()); // 출력: Point { X = 789, Y = 456 }

readonly record struct Point(int X, int Y);
```

Rust에는 `with`가 없습니다. 비슷한 동작이 필요하면 타입을 설계할
때 이를 구현해야 합니다.

```rust
struct Point { x: i32, y: i32 }

impl Point {
    pub fn new(x: i32, y: i32) -> Self {
        Self { x, y }
    }

    pub fn x(&self) -> i32 { self.x }
    pub fn y(&self) -> i32 { self.y }

    // 다음 메서드는 self를 소비하고 새 인스턴스를 반환합니다.

    pub fn set_x(self, val: i32) -> Self { Self::new(val, self.y) }
    pub fn set_y(self, val: i32) -> Self { Self::new(self.x, val) }
}
```

C#의 `with`는 읽고 쓸 수 있는 필드를 공개한 일반 `struct`에도
사용할 수 있습니다. 레코드가 아니어도 됩니다.

```c#
struct Point
{
    public int X;
    public int Y;

    public override string ToString() => $"({X}, {Y})";
}

var pt = new Point { X = 123, Y = 456 };
Console.WriteLine(pt.ToString()); // 출력: (123, 456)
pt = pt with { X = 789 };
Console.WriteLine(pt.ToString()); // 출력: (789, 456)
```

Rust의 _[구조체 갱신 구문]_은 이와 비슷해 보일 수 있습니다.

```rust
mod points {
    #[derive(Debug)]
    pub struct Point { pub x: i32, pub y: i32 }
}

fn main() {
    use points::Point;
    let pt = Point { x: 123, y: 456 };
    println!("{pt:?}"); // 출력: Point { x: 123, y: 456 }
    let pt = Point { x: 789, ..pt };
    println!("{pt:?}"); // 출력: Point { x: 789, y: 456 }
}
```

C#의 `with`는 복사한 뒤 값을 바꾸는 비파괴적 변경을 수행합니다.
반면 [구조체 갱신 구문]은 필드에 대해서만 작동하며 값의 일부를
_이동_합니다. 이 구문을 사용하려면 타입의 필드에 접근할 수 있어야
하므로 비공개 세부 사항에 접근할 수 있는 Rust 모듈 안에서 더 자주
사용합니다.

[구조체 갱신 구문]: https://doc.rust-lang.org/stable/book/ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax
