# 동등성

C#에서 두 값의 동등성을 비교할 때는 _값 동등성_을 검사하는 경우도 있고,
두 변수가 메모리의 같은 객체를 가리키는지 확인하는 _참조 동등성_을
검사하는 경우도 있습니다. 모든 사용자 정의 타입은 `System.Object`를
상속하므로 동등성을 비교할 수 있습니다. 값 타입은 `System.Object`를
상속하는 `System.ValueType`을 통해 이 기능을 얻습니다. 비교할 때는
타입에 적용된 동등성 의미에 따릅니다.

다음 C# 예제는 값 동등성과 참조 동등성을 비교합니다.

```csharp
var a = new Point(1, 2);
var b = new Point(1, 2);
var c = a;
Console.WriteLine(a == b); // (1) True
Console.WriteLine(a.Equals(b)); // (1) True
Console.WriteLine(a.Equals(new Point(2, 2))); // (1) False
Console.WriteLine(ReferenceEquals(a, b)); // (2) False
Console.WriteLine(ReferenceEquals(a, c)); // (2) True

record Point(int X, int Y);
```

1. `record Point`의 `==` 연산자와 `Equals` 메서드는 값 동등성을
   비교합니다. 레코드는 기본적으로 값 기반 동등성을 지원합니다.
2. 참조 동등성 비교는 두 변수가 메모리의 같은 객체를 가리키는지 검사합니다.

Rust에서는 같은 예제를 다음과 같이 작성할 수 있습니다.

```rust
#[derive(Copy, Clone)]
struct Point(i32, i32);

fn main() {
    let a = Point(1, 2);
    let b = Point(1, 2);
    let c = a;
    println!("{}", a == b); // 오류: "an implementation of `PartialEq<_>` might be missing for `Point`"
    println!("{}", a.eq(&b));
    println!("{}", a.eq(&Point(2, 2)));
}
```

위 컴파일 오류는 Rust의 동등성 비교가 언제나 트레이트 구현에 달려
있음을 보여 줍니다. `==` 비교를 지원하려면 타입이
[`PartialEq`][partialeq.rs]를 구현해야 합니다.

예제를 고치려면 `Point`에 `PartialEq`를 파생 구현합니다. 기본 파생
구현은 모든 필드를 비교하므로 각 필드도 `PartialEq`를 구현해야
합니다. C# 레코드의 동등성 비교와 비슷합니다.

```rust
#[derive(Copy, Clone, PartialEq)]
struct Point(i32, i32);

fn main() {
    let a = Point(1, 2);
    let b = Point(1, 2);
    let c = a;
    println!("{}", a == b); // true
    println!("{}", a.eq(&b)); // true
    println!("{}", a.eq(&Point(2, 2))); // false
    println!("{}", a.eq(&c)); // true
}
```

Rust의 모든 값이 참조인 것은 아니므로 참조 동등성과 일대일로 대응하는
개념은 없습니다. 다만 참조가 둘 있다면 `std::ptr::eq()`로 같은 대상을
가리키는지 확인할 수 있습니다.

```rust
fn main() {
    let a = 1;
    let b = 1;
    println!("{}", a == b); // true
    println!("{}", std::ptr::eq(&a, &b)); // false
    println!("{}", std::ptr::eq(&a, &a)); // true
}
```

참조를 원시 포인터로 변환하고 `==`로 비교하는 방법도 있습니다.

관련 자료:

- `PartialEq`보다 엄격한 동등성 조건을 다루는 [`Eq`][eq.rs]

[partialeq.rs]: https://doc.rust-lang.org/std/cmp/trait.PartialEq.html
[eq.rs]: https://doc.rust-lang.org/std/cmp/trait.Eq.html
