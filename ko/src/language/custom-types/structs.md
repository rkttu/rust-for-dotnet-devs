# 구조체(`struct`)

Rust와 C#의 구조체에는 몇 가지 공통점이 있습니다.

- 두 언어 모두 `struct` 키워드로 정의합니다. Rust의 `struct`는 데이터와
  필드만 정의하며 함수와 메서드로 표현하는 동작은 별도의 _구현 블록_
  (`impl`)에 작성합니다.
- C# 구조체가 여러 인터페이스를 구현할 수 있듯 Rust 구조체도 여러
  트레이트를 구현할 수 있습니다.
- 구조체를 상속해 하위 클래스를 만들 수 없습니다.
- 기본적으로 스택에 할당되지만 다음 경우에는 달라집니다.
  - .NET에서 박싱하거나 인터페이스로 변환한 경우
  - Rust에서 `Box`, `Rc`, `Arc` 같은 스마트 포인터로 감싼 경우

C#의 `struct`는 .NET의 _값 타입_을 모델링합니다. 도메인에 특화된
기본 값이나 값 동등성 의미를 가진 복합 값을 표현할 때 자주 사용합니다.
Rust의 `struct`는 데이터 구조를 모델링하는 주요 구문이며 다른 하나는
`enum`입니다.

C#의 `struct`와 `record struct`에는 기본적으로 값 복사와 값
동등성 의미가 있습니다. Rust에서 같은 기능을 얻으려면
[`#[derive]` 특성][derive]으로 구현할 트레이트를 지정합니다.

[derive]: https://doc.rust-lang.org/stable/reference/attributes/derive.html

```rust
#[derive(Clone, Copy, PartialEq, Eq, Hash)]
struct Point {
    x: i32,
    y: i32,
}
```

C#/.NET에서는 대개 값 타입을 불변으로 설계합니다. 의미상 좋은
설계로 여겨지지만 언어 자체가 `struct`의 제자리 수정을 막지는
않습니다. Rust에서도 타입을 불변으로 설계하려면 개발자가 의도적으로
구현해야 합니다.

Rust에는 클래스와 하위 클래스에 기반한 타입 계층이 없습니다. 여러
타입이 동작을 공유할 때는 트레이트와 제네릭을 사용하며,
[트레이트 객체]의 가상 디스패치로 다형성을 구현합니다.

[트레이트 객체]: https://doc.rust-lang.org/book/ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types

다음 C# `struct`는 직사각형을 나타냅니다.

```c#
struct Rectangle
{
    public Rectangle(int x1, int y1, int x2, int y2) =>
        (X1, Y1, X2, Y2) = (x1, y1, x2, y2);

    public int X1 { get; }
    public int Y1 { get; }
    public int X2 { get; }
    public int Y2 { get; }

    public int Length => Y2 - Y1;
    public int Width => X2 - X1;

    public (int, int) TopLeft => (X1, Y1);
    public (int, int) BottomRight => (X2, Y2);

    public int Area => Length * Width;
    public bool IsSquare => Width == Length;

    public override string ToString() => $"({X1}, {Y1}), ({X2}, {Y2})";
}
```

Rust에서 대응하는 코드는 다음과 같습니다.

```rust
#![allow(dead_code)]

struct Rectangle {
    x1: i32, y1: i32,
    x2: i32, y2: i32,
}

impl Rectangle {
    pub fn new(x1: i32, y1: i32, x2: i32, y2: i32) -> Self {
        Self { x1, y1, x2, y2 }
    }

    pub fn x1(&self) -> i32 { self.x1 }
    pub fn y1(&self) -> i32 { self.y1 }
    pub fn x2(&self) -> i32 { self.x2 }
    pub fn y2(&self) -> i32 { self.y2 }

    pub fn length(&self) -> i32 {
        self.y2 - self.y1
    }

    pub fn width(&self)  -> i32 {
        self.x2 - self.x1
    }

    pub fn top_left(&self) -> (i32, i32) {
        (self.x1, self.y1)
    }

    pub fn bottom_right(&self) -> (i32, i32) {
        (self.x2, self.y2)
    }

    pub fn area(&self)  -> i32 {
        self.length() * self.width()
    }

    pub fn is_square(&self)  -> bool {
        self.width() == self.length()
    }
}

use std::fmt::*;

impl Display for Rectangle {
    fn fmt(&self, f: &mut Formatter<'_>) -> Result {
        write!(f, "({}, {}), ({}, {})", self.x1, self.y1, self.x2, self.y2)
    }
}
```

C#의 `struct`는 `object`에서 `ToString` 메서드를 상속하므로
사용자 정의 문자열 표현이 필요하면 기본 구현을 _재정의_합니다.
Rust에는 상속이 없습니다. 타입이 `Display` 트레이트를 구현하면
포맷된 표현을 지원한다는 사실을 나타낼 수 있습니다. 그러면 다음
`println!` 호출처럼 구조체 인스턴스를 포맷할 수 있습니다.

```rust
fn main() {
    let rect = Rectangle::new(12, 34, 56, 78);
    println!("Rectangle = {rect}");
}
```
