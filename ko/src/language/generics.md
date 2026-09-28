# 제네릭

C#의 제네릭을 사용하면 다른 타입을 매개변수로 받는 타입과 메서드를
정의할 수 있습니다. 코드 재사용과 타입 안전성을 높이고 실행 시간의
캐스팅을 줄여 성능에도 도움이 됩니다. 다음 예제는 임의의 값에 시간
정보를 덧붙이는 제네릭 타입입니다.

```csharp
using System;

sealed record Timestamped<T>(DateTime Timestamp, T Value)
{
    public Timestamped(T value) : this(DateTime.UtcNow, value) { }
}
```

Rust에도 제네릭이 있습니다. 위 타입에 대응하는 코드는 다음과 같습니다.

```rust
use std::time::*;

struct Timestamped<T> { value: T, timestamp: SystemTime }

impl<T> Timestamped<T> {
    fn new(value: T) -> Self {
        Self { value, timestamp: SystemTime::now() }
    }
}
```

관련 자료:

- [제네릭 데이터 타입]

[제네릭 데이터 타입]: https://doc.rust-lang.org/book/ch10-01-syntax.html

## 제네릭 타입 제약 조건

C#에서는 `where` 절로 [제네릭 타입에 제약 조건][type-constraints.cs]을
지정할 수 있습니다. 다음 예제에서 제약 조건을 확인할 수 있습니다.

```csharp
using System;

// 참고: 레코드는 `IEquatable`을 자동으로 구현합니다. 다음
// 구현은 Rust와 비교하기 위해 이를 명시적으로 보여 줍니다.
sealed record Timestamped<T>(DateTime Timestamp, T Value) :
    IEquatable<Timestamped<T>>
    where T : IEquatable<T>
{
    public Timestamped(T value) : this(DateTime.UtcNow, value) { }

    public bool Equals(Timestamped<T>? other) =>
        other is { } someOther
        && Timestamp == someOther.Timestamp
        && Value.Equals(someOther.Value);

    public override int GetHashCode() => HashCode.Combine(Timestamp, Value);
}
```

Rust에서도 같은 목적의 제약 조건을 작성할 수 있습니다.

```rust
use std::time::*;

struct Timestamped<T> { value: T, timestamp: SystemTime }

impl<T> Timestamped<T> {
    fn new(value: T) -> Self {
        Self { value, timestamp: SystemTime::now() }
    }
}

impl<T> PartialEq for Timestamped<T>
where
    T: PartialEq,
{
    fn eq(&self, other: &Self) -> bool {
        self.value == other.value && self.timestamp == other.timestamp
    }
}
```

Rust에서는 매개변수를 선언할 때 바로 제약 조건을 붙이는 짧은 형식도
지원합니다.

```rust
impl<T: PartialEq> PartialEq for Timestamped<T> {
    fn eq(&self, other: &Self) -> bool {
        self.value == other.value && self.timestamp == other.timestamp
    }
}
```

다만 `where` 절은 `i32: PartialEq<T>`처럼 임의의 타입에도 제약
조건을 지정할 수 있어 표현 범위가 더 넓습니다.

Rust에서는 제네릭 타입 제약 조건을 [바운드][bounds.rs]라고 부릅니다.

C# 버전은 `T` 자체가 `IEquatable<T>`를 구현할 때만 `Timestamped<T>`
인스턴스를 만들 수 있습니다. Rust 버전에서는 `Timestamped<T>`가
조건부로 `PartialEq`를 구현하므로 적용 범위가 더 넓습니다.
동등성 비교를 지원하지 않는 `T`로도 `Timestamped<T>`를 만들 수
있지만, 그 인스턴스에는 `PartialEq`를 통한 동등성 비교가 없습니다.

관련 자료:

- [매개변수로 사용하는 트레이트]
- [트레이트를 구현하는 타입 반환]

[type-constraints.cs]: https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters
[bounds.rs]: https://doc.rust-lang.org/rust-by-example/generics/bounds.html
[매개변수로 사용하는 트레이트]: https://doc.rust-lang.org/book/ch10-02-traits.html#traits-as-parameters
[트레이트를 구현하는 타입 반환]: https://doc.rust-lang.org/book/ch10-02-traits.html#returning-types-that-implement-traits
