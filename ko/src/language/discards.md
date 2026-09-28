# 버림

C#의 [버림][net-discards]은 식의 결과 또는 결과의 일부를 사용하지
않겠다는 의도를 컴파일러와 다른 개발자에게 나타냅니다.

적용할 수 있는 위치는 여러 곳입니다. 가장 간단한 예로 식의 결과를
무시하는 C# 코드를 살펴보겠습니다.

```csharp
_ = city.GetCityInformation(cityName);
```

Rust에서 [식의 결과를 무시할 때][rust-ignoring-values]도 같은
형태로 작성합니다.

```rust
_ = city.get_city_information(city_name);
```

C#에서 튜플을 분해할 때도 버림을 사용할 수 있습니다.

```csharp
var (_, second) = ("first", "second");
```

Rust에서도 같은 방식으로 작성합니다.

```rust
let (_, second) = ("first", "second");
```

Rust는 튜플뿐 아니라 구조체와 열거형의 [분해][rust-destructuring]도
지원합니다. 이때 `..`는 타입에서 나머지 부분을 뜻합니다.

```rust
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

let origin = Point { x: 0, y: 0, z: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x), // x is 0
}
```

패턴을 일치시킬 때 결과의 일부를 버리거나 무시하면 편리한 경우가
있습니다. 다음은 C# 예제입니다.

```csharp
_ = ("first", "second") switch
{
    ("first", _) => "first element matched",
    (_, _) => "first element did not match"
};
```

Rust에서도 거의 같은 형태로 작성합니다.

```rust
_ = match ("first", "second")
{
    ("first", _) => "first element matched",
    (_, _) => "first element did not match"
};
```

[net-discards]: https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/discards
[rust-ignoring-values]: https://doc.rust-lang.org/stable/book/ch18-03-pattern-syntax.html#ignoring-values-in-a-pattern
[rust-destructuring]: https://doc.rust-lang.org/reference/patterns.html#destructuring
